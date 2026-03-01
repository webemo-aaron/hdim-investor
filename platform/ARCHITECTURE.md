# HDIM Architecture Deep-Dive

**For technical due diligence teams, CTOs, and architects**

---

## Architecture Overview

HDIM (HealthData-in-Motion) is built on three foundational patterns: event sourcing with
CQRS, database-per-service isolation, and FHIR R4 native execution. This combination creates
a 2+ year technical moat -- any single pattern can be replicated in isolation, but integrating
all of them into a working, HIPAA-compliant healthcare platform requires significant coordinated
engineering effort across clinical domain knowledge, distributed systems, and regulatory
compliance.

The architecture serves three goals: (1) real-time quality measurement via event-driven
processing, where clinically significant changes propagate through the system in seconds rather
than overnight batch cycles; (2) HIPAA compliance via immutable audit trails and tenant
isolation baked into the architecture rather than bolted on as middleware; and (3) horizontal
scalability via independent service scaling, where read-heavy analytics and write-heavy data
ingestion operate on separate infrastructure.

59 microservices. 29 independent PostgreSQL databases. 4 modularized API gateways. Event
backbone via Apache Kafka. Each service owns its data, publishes domain events, and scales
independently. The frontend is an Angular 17+ clinical portal with HIPAA-compliant audit
logging, session management, and PHI filtering enforced at the framework level.

```
                        +---------------------------+
                        |     Load Balancer / CDN   |
                        +---------------------------+
                                    |
              +---------------------+---------------------+
              |                     |                     |
     +--------v-------+   +--------v-------+   +---------v------+
     |  API Gateway   |   | Clinical GW    |   |  Admin GW      |
     |  (external)    |   | (portal)       |   |  (ops)         |
     +--------+-------+   +--------+-------+   +---------+------+
              |                     |                     |
              +---------------------+---------------------+
                                    |
                         +----------v-----------+
                         |   Internal Gateway   |
                         +----------+-----------+
                                    |
         +-----------+-----------+--+--+-----------+-----------+
         |           |           |     |           |           |
     +---v---+  +----v----+ +---v---+ +---v---+ +---v----+ +--v---+
     |Patient|  |Care Gap | | FHIR  | |Quality| | Risk   | | CQL  |
     |Service|  |Service  | |Service| |Measure| |Stratif.| |Engine|
     +---+---+  +----+----+ +---+---+ +---+---+ +---+----+ +--+---+
         |           |           |         |         |          |
     +---v---+  +----v----+ +---v---+ +---v---+ +---v----+     |
     |  DB   |  |   DB    | |  DB   | |  DB   | |   DB   |     |
     +-------+  +---------+ +-------+ +-------+ +--------+     |
                                                                |
         +------------------------------------------------------+
         |              Apache Kafka (Event Backbone)
         +------------------------------------------------------+
         |           |              |              |
     +---v--------+ +v-----------+ +v-----------+ +v-----------+
     |Patient     | |Care Gap    | |Quality     | |Risk        |
     |Event Svc   | |Event Svc   | |Measure Evt | |Strat. Evt  |
     +------------+ +------------+ +------------+ +------------+
```

---

## 1. Event Sourcing + CQRS

**Replication time: 18-24 months**

**The Problem:** Traditional healthcare systems store only current state. When a patient's
risk score changes from 80 to 75, there is no record of why it changed, what data was
considered, or when the calculation ran. Database transaction logs capture
"UPDATE risk_score SET value = 75" but not the business intent -- was it a new lab result?
A medication change? A corrected diagnosis? For healthcare payers managing value-based
contracts worth millions, this lack of provenance is a compliance risk and an operational
blind spot.

**HDIM's Solution:** Every clinically significant event is captured in an immutable,
append-only log via Apache Kafka. Events are never modified or deleted -- only new events
are appended. The event store is the system of record; all other data representations are
derived projections that can be rebuilt from the event history at any time.

```
+-------------------------------------------------+
| IMMUTABLE EVENT STORE (Kafka)                   |
+-------------------------------------------------+
| Event 1: PatientCreatedEvent                    |
|   Timestamp, Patient ID, Demographics           |
| Event 2: RiskScoreCalculatedEvent               |
|   Timestamp, Patient ID, Score: 85, Factors: [] |
| Event 3: CareGapIdentifiedEvent                 |
|   Timestamp, Patient ID, Measure: HBA1C         |
| Event 4: CareGapClosedEvent                     |
|   Timestamp, Patient ID, Closure Method: Lab    |
+-------------------------------------------------+
         | Projections (Materialized Views)
         v
+----------------+ +----------------+ +----------------+
| Patient View   | | Risk View      | | Quality View   |
| (PostgreSQL)   | | (PostgreSQL)   | | (PostgreSQL)   |
+----------------+ +----------------+ +----------------+
```

**CQRS Separation:** Write operations (commands) flow through event handlers that validate
business rules and publish domain events. Read operations (queries) hit materialized
projections optimized for fast retrieval -- denormalized, indexed, and cached. This
separation allows independent scaling: read-heavy analytics dashboards do not compete with
write-heavy clinical data ingestion. During open enrollment periods, read replicas can be
scaled horizontally without touching the write path.

**Why it matters for HIPAA:** The immutable event log is a native audit trail. HIPAA Section
164.312(b) requires audit controls that record and examine activity in systems containing
PHI. Event sourcing provides this by design, not by adding logging as an afterthought. Every
state change is traceable to a specific event with a timestamp, actor, and payload. Auditors
can reconstruct the complete history of any patient record at any point in time by replaying
its event stream.

**Event replay capability:** If a projection becomes corrupted or a new view is needed, the
system can rebuild materialized views by replaying the event log from any point in time. This
is not a theoretical capability -- it is tested in CI and used operationally when new
projections are added.

**Production services:** 4 event services are deployed and operational:

| Event Service | Domain | Key Events |
|---------------|--------|------------|
| patient-event-service | Patient lifecycle | Created, Updated, Merged, Deceased |
| care-gap-event-service | Care gap tracking | Identified, Addressed, Closed, Reopened |
| quality-measure-event-service | Measure evaluation | Evaluated, Scored, Reported |
| risk-stratification-event-service | Risk scoring | Calculated, Recalculated, Threshold-crossed |

---

## 2. Database-per-Service

**Replication time: 12 months**

29 independent PostgreSQL 16 databases, each owned by exactly one service. No shared
database access. No cross-service joins. No shared schemas.

**Benefits:**

- **Tenant isolation at the database kernel level.** This is not application-level row
  filtering that can be bypassed by a SQL injection or a developer mistake. Each tenant's
  data is physically separated in query execution paths enforced by PostgreSQL.
- **Independent schema evolution.** Each service manages its own schema through Liquibase
  migrations -- 199 changesets across the platform, with 100% rollback coverage. A schema
  change in the care gap service cannot break the patient service.
- **Service-specific query optimization.** The patient service can add indexes optimized
  for demographic searches without affecting the quality measure service's aggregation
  performance. Each database has its own connection pool, memory allocation, and vacuum
  schedule.
- **Blast radius containment.** A runaway query or connection pool exhaustion in one
  service's database does not cascade to other services. In a shared-database architecture,
  a single bad query can take down the entire platform.

**Schema management discipline:**

```
Entity change --> Liquibase migration --> CI validation --> Docker build
                  (XML changeset)        (Hibernate       (only if
                                          validate mode)   validation
                                                           passes)
```

Entity-migration validation is enforced in CI: every service validates its JPA entities
against actual Liquibase-migrated schemas at test time. Hibernate is set to `validate` mode
(never `create` or `update`). Schema drift is caught in the test suite, not discovered at
2 AM in production.

A pre-build validation script (`validate-before-docker-build.sh`) runs three checks before
any Docker image is built: database configuration validation, entity-migration synchronization,
and Liquibase rollback coverage. This shift-left approach has eliminated an entire class of
production incidents.

---

## 3. Gateway Architecture

**4 modularized API gateways with shared core**

In January 2026, the monolithic API gateway was decomposed into 4 specialized gateways
sharing a `gateway-core` module. Each gateway handles a specific traffic pattern with
tailored security, rate limiting, and routing policies.

| Gateway | Purpose | Traffic Pattern |
|---------|---------|-----------------|
| API Gateway | External API consumers | High-volume, rate-limited |
| Clinical Gateway | Clinical portal (Angular) | Session-based, PHI-heavy |
| Admin Gateway | Administrative operations | Low-volume, high-privilege |
| Internal Gateway | Service-to-service | High-trust, low-latency |

**Trust Authentication Pattern:**

```
Client --> Gateway (validates JWT) --> Service (trusts X-Auth-* headers)

Headers injected by gateway:
  X-Auth-User-ID:  authenticated user identity
  X-Auth-Roles:    comma-separated role list
  X-Tenant-ID:     validated tenant context
  X-Request-ID:    distributed trace correlation
```

The gateway validates JWT tokens and injects trusted headers. Downstream services trust
these headers and enforce authorization via Spring Security's `@PreAuthorize` annotations.
This eliminates redundant token validation across 59 services while maintaining a single
enforcement point for authentication policy changes. Rotating JWT signing keys requires a
change in one place, not fifty-nine.

**Operations Orchestration (February 2026):** A 16-class gateway framework standardizes
security enforcement, rate limiting, and multi-tenancy across all gateways. New services
automatically inherit enterprise operations patterns -- rate limiting, header sanitization,
tenant validation, and audit logging -- without per-service configuration. This is the
difference between "we have security" and "security is impossible to bypass."

**Header security hardening:** All gateway traffic validates and sanitizes headers before
forwarding to downstream services. Injection attempts via header manipulation are blocked
at the gateway perimeter.

---

## 4. FHIR R4 Native Execution

**Replication time: 9-12 months**

HDIM processes FHIR R4 resources natively. No ETL translation layer. No proprietary data
model conversion. No attribute loss.

**Why this matters:** Competitors extract FHIR data from EHR systems and translate it into
proprietary internal models. This translation typically loses 50-70% of FHIR resource
attributes -- extensions, modifiers, contained resources, and provenance metadata are
discarded because the proprietary model has no place for them. When a quality measure depends
on a FHIR extension (common in specialty care), these systems produce incorrect results.
HDIM preserves 100% of FHIR attributes because it operates directly on FHIR resources.

**CQL Evaluation Pipeline:**

```
FHIR Resource --> CQL Engine --> Measure Result --> Event Publishing
   (native)       (direct)      (<2 seconds)       (Kafka)
```

The CQL (Clinical Quality Language) engine executes directly against FHIR R4 resources using
HAPI FHIR 7.x. There is no intermediate translation step. This is why a single measure
evaluation completes in under 2 seconds while competitors using batch ETL pipelines report
24-hour turnaround times. The difference is architectural, not hardware -- no amount of
compute makes up for a fundamentally slower data path.

**Measure library:**

- 80+ HEDIS measures pre-built as CQL libraries
- Coverage: diabetes (HBA1C), cardiovascular, preventive screening, behavioral health,
  medication adherence, women's health
- Custom measure creation via UI shipped February 2026 -- clinical quality teams can author
  and test new measures without engineering involvement
- Each measure is version-controlled, testable in isolation, and deployable independently

**FHIR resource coverage:** Patient, Observation, Condition, Procedure, Encounter,
MedicationRequest, Immunization, DiagnosticReport, Claim, Coverage, and ExplanationOfBenefit.
Full R4 conformance validated against HAPI FHIR reference implementation.

---

## 5. Multi-Tenant Isolation

Three independent layers of tenant isolation, each enforced at a different level of the stack:

**Database Level:** Every repository query includes `WHERE tenantId = :tenantId`. This is
enforced via JPA annotations and Spring Data repository patterns, not developer discipline.
A service physically cannot execute a query without a tenant context. Code review tooling
flags any repository method missing tenant filtering.

**Cache Level:** Redis cache keys are tenant-prefixed (`{tenantId}:patient:{patientId}`).
PHI cache TTL is enforced at a maximum of 5 minutes per HIPAA compliance requirements.
All PHI responses include `Cache-Control: no-store` headers to prevent intermediate proxy
caching.

**Application Level:** `TrustedTenantAccessFilter` validates tenant headers on every inbound
request. The gateway injects `X-Tenant-ID` after JWT validation. Downstream services reject
any request arriving without a valid tenant context -- there is no "default tenant" fallback.
This is a hard rejection, not a log-and-continue.

**Role hierarchy:**

```
SUPER_ADMIN  -->  Full system access (cross-tenant)
  ADMIN      -->  Tenant-level administration
    EVALUATOR  -->  Run evaluations, view results
      ANALYST  -->  View reports and dashboards
        VIEWER -->  Read-only access
```

Role enforcement uses Spring Security's `@PreAuthorize` on every API endpoint. The CI
pipeline includes RBAC permission tests that verify role boundaries are maintained.

---

## 6. Observability

Full-stack observability is not optional in healthcare -- when a measure evaluation returns
an unexpected result, the clinical team needs to trace exactly which data was considered
and which code path executed.

- **OpenTelemetry distributed tracing** across all 59 services with automatic trace
  propagation over HTTP (Feign, RestTemplate) and Kafka (producer and consumer headers).
  A single request generates a correlated trace spanning gateway, service, database, cache,
  and event bus.
- **Custom spans** for critical operations: CQL measure evaluation, care gap detection,
  risk score calculation, and FHIR resource resolution. Each span captures
  operation-specific attributes (measure ID, patient count, evaluation duration).
- **Prometheus metrics collection** with service-level indicators (SLIs) for latency,
  throughput, and error rates per service, per tenant, and per measure.
- **Grafana dashboards** for operational visibility, including per-tenant resource
  utilization, quality measure processing throughput, and event processing lag.
- **Performance budgets** enforced at the p95 level in CI -- a pull request that degrades
  p95 latency beyond the budget threshold is flagged before merge.

---

## 7. Testing and Quality Assurance

613+ automated tests across 6 execution modes, with CI/CD completing in 23-25 minutes
(down from 60-70 minutes after 7 phases of infrastructure optimization).

| Mode | Duration | Purpose |
|------|----------|---------|
| testUnit | 30-45s | Fast feedback during development |
| testFast | 1.5-2 min | Pre-commit validation |
| testIntegration | 1.5-2 min | API and service layer verification |
| testSlow | 3-5 min | Heavyweight validation (Kafka, DB) |
| testAll | 10-15 min | Final merge gate (100% stable) |
| testParallel | 5-8 min | Experimental parallel execution |

**CI/CD pipeline (7 phases of optimization):**

- Phase 5: Embedded Kafka migration -- eliminated Docker dependency for Kafka tests
- Phase 6: Gradle parallelization -- 6 parallel JVM forks, 33% faster test suite
- Phase 7: CI/CD parallelization -- 4 parallel test jobs, 3 parallel validation jobs,
  intelligent change detection (21 service-specific filters)

**Contract testing** via Pact (consumer-driven) and OpenAPI validation ensures API contracts
between services remain stable across independent deployments. The `contract-gate` CI job
blocks merges if any contract test fails.

**Entity-migration validation** tests run in every service's test suite, catching schema
drift at test time rather than runtime. This is enforced in CI on every pull request that
modifies entity classes or migration files.

---

## Replication Timeline Summary

| Component | Time to Replicate | Complexity Driver |
|-----------|-------------------|-------------------|
| Event sourcing + CQRS | 18-24 months | Architecture redesign, event store, projections, replay |
| FHIR R4 native execution | 9-12 months | CQL engine integration, HAPI FHIR compliance |
| 80+ HEDIS CQL measures | 12-18 months | Clinical logic, certification, reference validation |
| Database-per-service (29 DBs) | 12 months | Schema design, migration tooling, isolation patterns |
| Revenue cycle integration | 12 months | Claims, remittance, price transparency on event backbone |
| Gateway modularization | 6 months | 4 gateways, shared core, trust auth, ops framework |
| Observability stack | 6 months | OpenTelemetry, custom spans, dashboards, alerting |
| Security hardening | 6 months | CVE remediation, penetration testing, HIPAA verification |
| CI/CD pipeline (7 phases) | 4 months | Parallel execution, change detection, caching |
| **Total (integrated)** | **2+ years** | **Integration across clinical, regulatory, and distributed systems** |

The individual components are well-understood patterns. The moat is in their integration:
event sourcing that understands FHIR resources, tenant isolation that extends from database
through cache through gateway, CQL execution that publishes results as domain events, and
a CI pipeline that validates all of it in under 25 minutes. Replicating this requires not
just engineering time but deep domain expertise in healthcare interoperability, quality
measurement, and regulatory compliance -- a combination that is exceptionally rare in the
market.

---

*Confidential -- For Investor Use Only | March 2026*
