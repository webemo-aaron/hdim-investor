# Security & Compliance

**HIPAA compliance engineered into every architectural layer -- not retrofitted.**

Most healthcare software vendors bolt security on after the fact. HDIM was designed from day one with HIPAA controls embedded in the architecture itself. Every service, every query, every API call enforces the same security invariants -- not because a compliance checklist said so, but because the code physically cannot operate any other way.

---

## Security Posture Summary

| Domain | Status | Evidence |
|--------|--------|----------|
| HIPAA SS 164.312 | Compliant | Engineered controls at every layer (5 sub-requirements) |
| CVE Remediation | Active | Wave-based burn-down, NVD closeout automation, dependency-check gate |
| Security Scanning | Active | OWASP ZAP baselines, dependency-check aggregate, npm audit |
| Platform Assurance | Verified | 360 checklist with 28/28 checks passing, evidence sign-off |
| Audit Coverage | 100% | HTTP interceptor on all API calls, session timeout logging |
| Multi-Tenant Isolation | Enforced | Database, cache, query-level, and application-layer separation |
| Session Management | HIPAA-compliant | 15-minute automatic logoff with full audit trail |
| Encryption | In transit | TLS 1.3 for all service-to-service and client communication |
| Data Access Security | Grade A | 19/19 controls passing (validated February 2026) |

---

## HIPAA SS 164.312 Technical Safeguards

HIPAA was not added as a compliance layer. It is the architecture. Each sub-requirement maps directly to enforced, testable controls in the codebase.

### SS 164.312(a) -- Access Control

**Requirement:** Implement technical policies and procedures to allow access only to authorized persons.

**How HDIM enforces it:**

- **Role-based access control (RBAC)** with five hierarchical roles: SUPER_ADMIN, ADMIN, EVALUATOR, ANALYST, VIEWER. Every API endpoint is protected with `@PreAuthorize` annotations -- there are no unprotected endpoints.
- **Multi-tenant isolation at the database level.** Every table includes a `tenant_id` column defined as `NOT NULL`. Every repository query filters by tenant. A user authenticated for Tenant A physically cannot retrieve Tenant B data because the SQL WHERE clause makes it impossible.
- **Automatic session logoff** after 15 minutes of inactivity, satisfying SS 164.312(a)(2)(iii). The frontend tracks idle time via click, keypress, mousemove, and scroll listeners. A 2-minute warning gives users the option to extend. If no action is taken, the session terminates and the event is logged.
- **Session timeout audit logging** differentiates between automatic idle timeout and explicit user logout. Each event captures: user ID, timestamp, idle duration, timeout reason, and whether a warning was displayed.

**Evidence artifacts:**
- `TrustedHeaderAuthFilter.java` -- gateway trust validation on every request
- `TrustedTenantAccessFilter.java` -- tenant access enforcement
- Data access security matrix: 19/19 controls passing, Grade A

### SS 164.312(b) -- Audit Controls

**Requirement:** Implement hardware, software, and procedural mechanisms to record and examine activity in systems that contain PHI.

**How HDIM enforces it:**

- **100% API call audit coverage.** An HTTP audit interceptor automatically logs every backend API call. No developer action required -- the interceptor is registered globally.
- **Structured audit records** capture: action type (CREATE, READ, UPDATE, DELETE), resource type, resource ID, user ID, tenant ID, timestamp, request duration, and success/failure outcome.
- **Fire-and-forget batching** ensures audit logging never blocks request processing. Audit records are batched and persisted asynchronously.
- **`@Audited` annotation** on all PHI access methods provides an additional explicit audit layer beyond the automatic interceptor. 85+ annotation instances across the codebase.
- **Session timeout events** are logged separately, capturing automatic timeout vs. explicit logout with idle duration tracking.
- **6-year retention capability** supports HIPAA's minimum retention requirements for audit logs.

**Evidence artifacts:**
- HTTP Audit Interceptor (Angular): automatic coverage of all API calls
- `@Audited` annotation: 85+ usages across backend services
- Session timeout audit: 6/6 compliance tests passing
- HIPAA controls validation: all checks passing

### SS 164.312(c) -- Integrity Controls

**Requirement:** Implement policies and procedures to protect PHI from improper alteration or destruction.

**How HDIM enforces it:**

- **Event sourcing architecture.** Four dedicated event services (patient-event-service, care-gap-event-service, clinical-workflow-event-service, quality-measure-event-service) maintain append-only event logs. Events are never modified or deleted. The current state is derived by replaying the event stream, providing a complete, immutable history of every data change.
- **Liquibase-managed schema migrations** with 100% rollback coverage across 199+ changesets and 29 independent databases. No migration is deployed without a corresponding rollback directive.
- **Entity-migration validation** catches schema drift at test time, not at runtime. Every service runs `EntityMigrationValidationTest` to verify that JPA entity definitions match the actual database schema produced by Liquibase migrations.
- **Hibernate `validate` mode** in all environments. The application refuses to start if entity definitions do not match the database schema.

**Evidence artifacts:**
- 4 event services with append-only event stores
- 199+ Liquibase changesets with 100% rollback coverage
- 29 independent database schemas
- Entity-migration validation in CI/CD pipeline

### SS 164.312(d) -- Person or Entity Authentication

**Requirement:** Implement procedures to verify that a person or entity seeking access to PHI is the one claimed.

**How HDIM enforces it:**

- **JWT token validation at the gateway.** The API gateway validates every JWT token before routing requests to backend services. Invalid or expired tokens are rejected at the edge.
- **Gateway trust header injection.** After validating the JWT, the gateway injects trusted headers: `X-Auth-User-ID`, `X-Auth-Roles`, `X-Tenant-ID`, and an HMAC signature. The gateway strips any externally-provided `X-Auth-*` headers to prevent header spoofing.
- **HMAC signature verification.** Backend services verify the gateway's HMAC signature on every request. A request that did not pass through the gateway is rejected. Direct service-to-service authentication bypass is architecturally impossible.
- **No credential storage in services.** Backend services never handle raw credentials. They only consume pre-validated, gateway-signed identity assertions.

**Evidence artifacts:**
- `TrustedHeaderAuthFilter.java` -- HMAC verification, header extraction
- `GatewaySecurityConfig.java` -- gateway security configuration
- `GatewayForwarder.java` -- header injection and signing
- 4-gateway modular architecture (gateway-core shared module)

### SS 164.312(e) -- Transmission Security

**Requirement:** Implement technical security measures to guard against unauthorized access to PHI transmitted over an electronic communications network.

**How HDIM enforces it:**

- **TLS 1.3** for all client-to-gateway and service-to-service communication.
- **Kafka encryption** for event stream data between services.
- **No PHI in URL parameters or query strings.** All PHI is transmitted in request bodies or headers, never in URLs that could be logged by proxies or load balancers.
- **Cache-Control headers** on all PHI responses: `no-store, no-cache, must-revalidate`. PHI is never cached by intermediate proxies.
- **Redis cache TTL enforced at 5 minutes maximum** for any data containing PHI.

**Evidence artifacts:**
- HIPAA cache compliance configuration
- Cache-Control header enforcement in all PHI controllers
- Redis TTL configuration validation

---

## CVE Remediation

HDIM maintains an active vulnerability management program, not a one-time scan.

**Current posture:**

- **Wave-based remediation** with prioritized burn-down tracking. Vulnerabilities are triaged by severity, grouped into remediation waves, and resolved with evidence manifests documenting each closure.
- **Dependency-check aggregate scanning** across all 51 backend services using OWASP Dependency-Check with NVD feed integration.
- **npm audit enforcement** on frontend dependencies. Post-remediation scans show zero high-severity vulnerabilities.
- **Immutable evidence manifests** with SHA-256 checksums for each remediation wave. Evidence is tamper-evident and audit-ready.
- **Pre-NVD CVE packet tooling** enables rapid vulnerability assessment even before NVD enrichment data is available.
- **CI/CD compliance gate** blocks merges when unresolved critical CVEs are detected.

**Evidence artifacts:**
- Wave-1 local assurance: 28/28 checks passing
- Local evidence manifest with SHA-256 checksums
- OWASP Dependency-Check reports (HTML, JSON, SARIF)
- npm audit: 0 vulnerabilities (post-remediation)

---

## Security Scanning

Automated scanning runs continuously, not on a schedule.

| Scan Type | Tool | Trigger | Output |
|-----------|------|---------|--------|
| DAST (Dynamic) | OWASP ZAP | PR and release gates | HTML/JSON reports with alert triage |
| SCA (Backend) | OWASP Dependency-Check | CI/CD pipeline | SARIF, JSON, HTML reports |
| SCA (Frontend) | npm audit | CI/CD pipeline | Vulnerability count with severity |
| SAST (Static) | ESLint security rules | Every build | PHI exposure prevention, no-console enforcement |

**ZAP baseline triage** classifies findings by severity with documented ownership and disposition status. High and medium findings are tracked to resolution with evidence artifacts.

**Evidence artifacts:**
- ZAP baseline reports (HTML, JSON, alerts summary)
- ZAP triage documentation with finding disposition
- Dependency-check aggregate reports
- ESLint no-console enforcement (build fails on violation)

---

## Platform 360 Assurance

HDIM maintains a comprehensive assurance checklist covering 8 control domains, validated against auditable evidence artifacts.

| Domain | Controls | Status |
|--------|----------|--------|
| A: Feature Completeness | Service inventory, API health, smoke tests, gate orchestrations | Passing |
| B: Data Model Integrity | Entity-schema alignment, migration consistency, tenant partitioning | Passing |
| C: Performance | SLO definitions, load tests, P95 thresholds in CI | Passing |
| D: HIPAA Safeguards | PHI cache TTL, HIPAA controls script, audit trail, DR/retention | Passing |
| E: SOC2 Security Criteria | Control-evidence mapping, access/auth controls, change management | Passing |
| F: Testing | 5,852 tests, 0 failures, 0 errors across 48 service modules | Passing |
| G: Operational Readiness | Runbooks, monitoring, alerting, incident response | Passing |
| H: Deployment | Release gates, evidence packs, deployment validation | Passing |

**Assurance process:**
- External audit-ready review mode
- Named control owners for every domain
- Evidence artifacts linked to each control
- Sign-off process for release approval
- Monthly assurance cadence with evidence refresh

**Evidence artifacts:**
- Platform 360 Assurance Checklist (dated, signed)
- SOC2 CC Control-Evidence Matrix
- Compliance evidence retention and cadence policy
- Release evidence packs with deployment summaries

---

## Frontend Security

The Angular Clinical Portal handles PHI directly. Frontend security controls are as rigorous as backend controls.

**Enforced controls:**

- **No console.log in production.** ESLint rules fail the build if any `console.log`, `console.error`, `console.warn`, or `console.debug` statements are detected. This prevents PHI from being exposed in browser DevTools. 98.2% of legacy violations eliminated (109/111), with ESLint enforcement preventing new ones.
- **LoggerService with PHI filtering.** All logging goes through a centralized `LoggerService` that automatically strips PHI patterns in production builds. 30+ files migrated from console statements to LoggerService.
- **Global error handler.** Unhandled exceptions are caught, logged with PHI filtering, and reported as security incidents. The application never crashes from an unhandled error.
- **Session management.** 15-minute idle timeout with 2-minute warning. Activity tracked via click, keypress, mousemove, and scroll listeners. All timeout events audited with reason, duration, and user context.
- **No hardcoded PHI.** Templates use data binding exclusively. Static patient data in templates is a build-time lint violation.

---

## Multi-Tenant Security

Multi-tenant isolation is enforced at four independent layers. Compromising one layer is insufficient -- all four would need to be bypassed simultaneously.

| Layer | Mechanism | Enforcement Point |
|-------|-----------|-------------------|
| Database | `tenant_id` column on every table, `NOT NULL` constraint | JPA `@Column(nullable = false)`, repository query filters |
| Cache | Tenant-prefixed Redis keys, 5-minute maximum TTL for PHI | Cache configuration, key generation, eviction policy |
| Application | `TrustedTenantAccessFilter` on every request | Gateway header injection, service-level validation |
| Query | `WHERE tenant_id = :tenantId` on every data access | Repository method signatures, no exceptions permitted |

**Why this matters for enterprise buyers:**

Cross-tenant data access is not a configuration error waiting to happen. It is architecturally impossible. A query that omits the tenant filter will not compile (repository method signatures require it). A request without a valid tenant header will be rejected by the filter chain before reaching any business logic. A cache lookup without the tenant prefix will return a cache miss, not another tenant's data.

**Data access security validation:** 19/19 controls passing, Grade A (validated February 2026).

---

## Compliance Evidence Trail

Every compliance requirement maps to an auditable evidence artifact. Evidence is not documentation about what we intend to do -- it is proof of what the system actually does.

| Requirement | Evidence Type | Validation Method |
|-------------|--------------|-------------------|
| Access controls | Role definitions, `@PreAuthorize` annotations | Automated security tests, data access matrix |
| Audit logs | HTTP interceptor records, session timeout logs | Audit trail queries, compliance test suite |
| Data integrity | Event sourcing event store, migration rollback coverage | Entity-migration validation tests, Liquibase audit |
| Authentication | JWT validation, HMAC-signed gateway headers | Gateway integration tests, header spoofing tests |
| Encryption | TLS 1.3 configuration, Kafka encryption | Configuration validation, network inspection |
| CVE remediation | Burn-down reports, SHA-256 evidence manifests | Dependency-check gate, npm audit gate |
| Security scanning | ZAP reports, dependency-check SARIF | CI/CD gate results, triage documentation |
| Tenant isolation | Query-level enforcement, filter chain | 19/19 data access security controls |
| Platform assurance | 360 checklist, release evidence packs | 28/28 assurance checks, signed evidence bundles |

### Evidence Bundle Index

HDIM maintains a signed evidence bundle index that consolidates all compliance artifacts into a single auditable package:

- Security and vulnerability evidence (CVE scans, dependency reports)
- HIPAA operational control evidence (PHI controls, data access matrix)
- SOC2 operational and change evidence (release gates, deployment packs)
- Compliance documentation (control matrices, retention policies)
- Commit traceability (every evidence artifact linked to its commit SHA)
- Bundle sign-off with named compliance and technical owners

The evidence bundle is refreshed with each release and maintains full traceability from compliance requirement to code commit to runtime verification.

---

## What This Means for Enterprise Buyers

**For the CISO:** HDIM provides defense-in-depth with controls at gateway, application, database, and cache layers. Security is not a feature that can be misconfigured -- it is a structural property of the architecture.

**For the compliance team:** Every HIPAA sub-requirement maps to specific, testable controls with evidence artifacts. The compliance evidence bundle is audit-ready and maintained with each release cycle.

**For the procurement team:** HDIM's security posture reduces the risk profile of the vendor assessment. Multi-tenant isolation, CVE management, and audit controls are already validated -- not promised in a future roadmap.

**For technical due diligence:** The codebase is the evidence. `@PreAuthorize` on every endpoint, `tenant_id` on every table, `@Audited` on every PHI method, HMAC signatures on every gateway header. These are not aspirational controls -- they are compile-time and runtime invariants.

---

*Confidential -- For Investor Use Only | March 2026*
