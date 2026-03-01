# Competitive Analysis

**HDIM vs. incumbent healthcare quality measurement platforms**

---

## Executive Summary

HDIM's competitive advantage is architectural, not incremental. The combination of event sourcing, FHIR R4 native execution, and database-per-service isolation creates a 2+ year technical moat that competitors cannot close through feature additions alone. Where incumbents bolt quality measurement onto legacy systems designed for other purposes, HDIM was purpose-built from day one as an event-driven quality measurement engine. This document details the head-to-head comparison across 8 dimensions.

---

## Head-to-Head Comparison

| Capability | HDIM | Epic Healthy Planet | Optum/UHG | Inovalon | Arcadia | Salesforce Health |
|------------|------|---------------------|-----------|----------|---------|-------------------|
| Evaluation Speed | <2 sec | 24 hours | 24 hours | 24 hours | 12 hours | 24 hours |
| Implementation | 90 days | 18-24 months | 12-18 months | 6-12 months | 6-9 months | 9-12 months |
| FHIR Preservation | 100% | 30-40% | 40-50% | 40-50% | 50-60% | 50-60% |
| Custom Measures | Yes (UI) | No | Limited API | Limited | No | No |
| Revenue Cycle | Integrated | Separate product | Separate (Optum) | No | No | Separate |
| Multi-Tenant | Database-level | App-level | App-level | App-level | App-level | App-level |
| Pricing | $150-300K/yr | $1-5M/yr | $500K-2M/yr | $500K-1M/yr | $300K-800K/yr | $500K-2M/yr |
| EHR Agnostic | Yes (FHIR R4) | Epic only | UHG affiliated | Limited | Multi-EHR | Multi-EHR |
| CQL Execution | Native engine | Proprietary | Proprietary | Proprietary | None | None |
| Event Sourcing | Full CQRS | None | None | None | None | None |
| CI/CD Maturity | 21-path detection | Unknown | Enterprise CI | Standard | Standard | Salesforce DX |
| Test Coverage | 1,171 classes | Closed source | Closed source | Closed source | Closed source | Closed source |

---

## Competitive Dimensions

### 1. Real-Time vs. Batch Processing

Every incumbent quality measurement platform operates on a batch processing model. Clinical data is extracted overnight, transformed into a proprietary format, loaded into an analytics warehouse, and evaluated against quality measures the following morning. This means care gaps discovered on Monday night are not actionable until Tuesday at the earliest, and often not until the next scheduled patient interaction weeks later. For a patient with a missed HbA1c screening, this delay can mean the difference between an intervention during an already-scheduled visit and a costly outreach campaign months later.

HDIM evaluates quality measures through an event-driven architecture built on Apache Kafka and the CQRS pattern. When a clinical event occurs (lab result, diagnosis, procedure), it triggers immediate CQL evaluation against all applicable HEDIS measures. Care gaps are identified in under 2 seconds and surfaced to clinical staff while the patient is still in the care setting. This is not a performance optimization layered onto a batch system. It is a fundamentally different architecture where events flow through the system in real time, projections are updated continuously, and quality status is always current.

The business impact is measurable. Health plans that close care gaps during active encounters avoid the $50-150 per-member outreach cost of retrospective gap closure. For a plan with 500,000 members and a 15% care gap rate, same-day closure eliminates $3.75M-$11.25M in annual outreach spend. No batch-processing platform can deliver this outcome regardless of how fast their nightly job runs.

### 2. FHIR R4 Native vs. ETL Translation

Incumbent platforms extract FHIR data from EHR systems and translate it into proprietary internal models before processing. This translation step is lossy by design. A FHIR Observation resource contains 30+ attributes (status, category, code, value, effective date, interpretation, reference ranges, components). Typical ETL pipelines extract 10-15 of these attributes into flat relational tables, discarding the rest. When a quality measure depends on an attribute that was dropped during translation, it evaluates incorrectly.

HDIM processes FHIR R4 resources natively. The CQL engine operates directly on FHIR resource structures with zero intermediate translation. An Observation arrives as FHIR JSON, is stored as FHIR JSON, and is evaluated as a FHIR resource. This preserves 100% of clinical attributes and eliminates an entire category of measure evaluation errors. When CMS updates a HEDIS measure to reference a previously unused FHIR attribute, HDIM evaluates it correctly on day one. Competitors must update their ETL mappings, reprocess historical data, and validate the new pipeline, a process that typically takes 3-6 months.

The accuracy difference compounds across measure portfolios. With 80+ HEDIS measures each referencing dozens of FHIR attributes, even a 5% attribute loss rate introduces systematic evaluation errors. Health plans subject to CMS Star Ratings audits cannot afford systematic measurement bias. HDIM's native FHIR processing eliminates this risk category entirely.

### 3. Implementation Speed

HDIM deploys in 90 days. Incumbents require 6-24 months. This is not a sales promise; it is an architectural consequence.

Traditional implementations are slow because they require extensive data mapping. Each customer's EHR stores data differently, and the ETL pipeline must be customized for every source system. HDIM bypasses this entirely by consuming standard FHIR R4 resources. If the EHR supports FHIR R4 (and CMS now mandates it), the data arrives in the format HDIM expects. No custom mapping, no data warehouse schema design, no ETL pipeline construction.

HDIM's deployment infrastructure further compresses timelines. Docker Compose orchestrates 59 microservices with automated database migrations (Liquibase), self-service API documentation (OpenAPI 3.0 with Swagger UI), and pre-configured monitoring (Prometheus, Grafana). A hospital IT team receives a deployment package, runs a single command, and has a functioning quality measurement platform. The 90-day timeline includes integration testing, user training, and a pilot evaluation period. The actual technical deployment takes days, not months.

For customers, implementation speed translates directly to time-to-value. A health plan that deploys HDIM in Q1 captures a full year of quality measurement data for CMS reporting. A plan that begins an 18-month Epic implementation in Q1 misses the entire reporting year and does not see value until the following calendar year.

### 4. Platform Breadth

HDIM spans quality measurement, care gap detection, revenue cycle management, and custom measure creation on a single event-driven backbone. These capabilities share the same data model, the same event bus, and the same security infrastructure. A clinical event that triggers a care gap evaluation also triggers a claims eligibility check. A measure evaluation result feeds directly into revenue cycle reporting without a separate data integration.

Competitors offer these capabilities as separate products with separate implementations, separate data models, and separate vendor contracts. Epic's quality measurement lives in Healthy Planet while revenue cycle lives in Resolute. Optum separates quality analytics from claims processing. Salesforce Health Cloud handles CRM but requires partners for clinical quality. Each boundary between products introduces integration complexity, data latency, and additional cost.

For customers evaluating total cost of ownership, platform breadth matters more than individual feature depth. A single HDIM deployment replaces 2-3 separate vendor relationships, eliminates cross-product data synchronization, and reduces the internal IT team needed to manage the quality measurement stack. The total cost difference is not just the license fee. It is the $200K-500K in annual integration maintenance that disappears when quality measurement and revenue cycle share a platform.

### 5. Security Posture

HDIM provides evidence-based security at the architecture level. Every service undergoes CVE scanning with burn-down tracking. ZAP (OWASP) security scanning runs on every pull request. Platform assurance reviews produce auditable artifacts with sign-off documentation. The security posture is not a compliance checkbox; it is a continuous, automated process embedded in the CI/CD pipeline. Investors and customers can review the actual CVE remediation timeline, the scan results, and the assurance sign-off chain.

Competitors claim HIPAA compliance without providing architectural evidence. "SOC 2 certified" and "HIPAA compliant" are table stakes. They describe audit outcomes, not engineering practices. HDIM's security approach is transparent: 29 isolated databases with tenant-level separation, encrypted data at rest and in transit, JWT-based authentication with role hierarchy enforcement, and a complete audit trail via event sourcing. Every state change is recorded as an immutable event. There is no way to modify patient data without creating an auditable record.

### 6. Custom Measure Creation

HDIM's measure builder UI allows health plans to create, configure, and deploy custom quality measures without engineering involvement. The workflow includes a metadata editor, CQL logic builder, patient population criteria, and deployment pipeline, all accessible through a web interface. A quality team can define a proprietary measure, test it against sample data, and deploy it to production in hours.

No competitor offers self-service measure creation. Health plans with custom measures (state-specific requirements, proprietary quality programs, value-based contract metrics) submit requests to vendor engineering queues with 3-6 month turnaround times. This creates a dependency bottleneck that slows quality program innovation. HDIM eliminates this dependency entirely, enabling health plans to iterate on measure definitions at the speed of their clinical programs rather than the speed of their vendor's engineering backlog.

### 7. Multi-Tenant Isolation

HDIM implements database-per-service architecture with 29 independent databases, each enforcing tenant isolation at the schema level. Cross-tenant data access is architecturally impossible. There is no shared table where a missing WHERE clause could expose one tenant's data to another. Each tenant's data resides in a physically separate schema with separate connection pools and separate access credentials. The application layer adds a second enforcement boundary via `TrustedTenantAccessFilter`, but the database architecture means that even a complete application-layer failure cannot produce cross-tenant data leakage.

Competitors rely on application-level row filtering: a single `tenant_id` column on shared tables with application code responsible for including the filter on every query. This model has produced data breaches across the software industry. A single missing filter, a single ORM misconfiguration, a single raw SQL query without the tenant predicate, and data crosses tenant boundaries. For healthcare organizations subject to HIPAA breach notification requirements, the difference between architectural isolation and application-level filtering is the difference between a design that prevents breaches and a design that hopes to prevent them.

### 8. Pricing

HDIM targets $150-300K per year, positioning it at 30-60% below the nearest competitor and 90% below enterprise incumbents. This pricing is sustainable, not subsidized. HDIM's architecture requires fewer professional services hours for implementation (90 days vs. 12-24 months), lower infrastructure costs (Docker-based deployment vs. dedicated hardware), and less ongoing vendor engineering support (self-service API documentation, automated migrations, custom measure builder).

The pricing structure opens the mid-market segment that incumbents cannot serve profitably. Regional health plans with 50,000-500,000 members need quality measurement capabilities but cannot justify $1-5M annual license fees plus 18-month implementation costs. HDIM's price point and implementation speed make quality measurement accessible to organizations that have historically relied on manual spreadsheet-based processes. This is a market expansion opportunity, not just competitive displacement. The 2,000+ potential customers in this segment represent uncontested market space where incumbents have no presence and no viable path to compete on price.

---

## Replication Timeline

Why competitors cannot catch up:

| Component | Time to Replicate | Why It's Hard |
|-----------|-------------------|---------------|
| Event sourcing + CQRS | 18-24 months | Requires complete architecture redesign; cannot retrofit onto batch systems |
| FHIR R4 native execution | 9-12 months | CQL engine integration, FHIR resource model throughout stack, compliance certification |
| 80+ HEDIS CQL measures | 12-18 months | Clinical logic authoring, edge case validation, CMS certification per measure |
| Revenue cycle integration | 12 months | Claims, remittance, price transparency on shared event backbone |
| Database-per-service (29 DBs) | 12 months | Schema design, migration tooling (Liquibase), isolation testing, connection management |
| Custom measure builder | 6-9 months | UI workflow, metadata framework, CQL generation, deployment pipeline |
| Security hardening | 6 months | CVE remediation process, ZAP scanning integration, assurance artifact chain |
| CI/CD maturity | 6 months | 21-path change detection, parallel test workflows, performance monitoring |
| OpenTelemetry tracing | 3-6 months | Cross-service trace propagation (HTTP, Kafka), span instrumentation |
| 1,171 test classes | 6-12 months | Unit, integration, contract, entity-migration validation across 59 services |

**Cumulative replication estimate: 24-36 months of focused engineering effort.**

Replicating any single component is feasible for a well-funded engineering team. Replicating all of them working together as an integrated platform, where events flow through quality measurement into revenue cycle into custom measures with database-level tenant isolation and real-time evaluation, takes 2+ years. By that point, HDIM will have production customers, validated outcomes data, and the next generation of features already shipping.

---

## Structural Barriers by Competitor

### Epic Healthy Planet

Epic will never build an EHR-agnostic quality measurement platform. Doing so would undermine the lock-in that makes Epic's core EHR business valuable. Every feature Epic adds to Healthy Planet reinforces the dependency on Epic's EHR, not the portability that health plans need when they work with providers across multiple EHR systems.

Epic implementations take 18-24 months because the complexity is inherent to their monolithic architecture, not because they lack engineering resources. Epic has 10,000+ engineers. The implementation timeline reflects integration complexity with their tightly coupled system, not capacity constraints.

Epic's captive customer base (representing ~35% of US hospital beds) creates complacency. There is no competitive pressure to innovate on quality measurement when customers are contractually locked in for 10+ years. HDIM competes for the health plans that work across Epic, Cerner, Athena, and other EHR systems simultaneously, a use case Epic cannot address by design.

### Optum / UnitedHealth Group

Optum's affiliation with UnitedHealth Group is simultaneously its greatest asset and its most limiting constraint. Health plans that compete directly with UHG (which is most of them) are structurally unwilling to send clinical data to a subsidiary of their largest competitor. This creates a permanent addressable market ceiling for Optum's quality measurement tools.

Optum's quality analytics and claims processing are separate products with separate implementations, separate data models, and separate pricing. Integrating them into a unified platform would require the same architectural redesign that HDIM was purpose-built to avoid. Optum's enterprise-only pricing ($500K-2M/year) further excludes the mid-market segment where HDIM is positioned.

### Inovalon

Inovalon processes clinical data through batch ETL pipelines with overnight evaluation cycles. This is an architectural limitation embedded in their data warehouse design, not a performance optimization they can apply incrementally. Moving to real-time evaluation would require replacing their core data infrastructure.

Inovalon offers no custom measure creation capability. Health plans with proprietary quality programs must work through Inovalon's professional services team, creating bottlenecks and dependency. Their FHIR R4 support is limited to data ingestion; internal processing uses proprietary models with the same attribute loss described in Section 2.

### Arcadia

Arcadia provides multi-EHR data aggregation, which positions them as a data platform rather than a quality measurement engine. Their 12+ hour evaluation cycles reflect a batch architecture optimized for data warehousing, not clinical decision support. Quality measurement is a feature of their analytics platform, not the core product.

Arcadia offers no revenue cycle integration and no custom measure creation. Their multi-EHR connectivity, while broader than Epic's single-EHR approach, still relies on ETL translation rather than native FHIR processing. This means the same attribute loss and measure accuracy issues apply, even though the data sources are more diverse.

### Salesforce Health Cloud

Salesforce Health Cloud is a CRM platform with healthcare extensions, not a clinical quality measurement system. Quality measurement is an add-on capability that requires extensive customization, third-party integrations, and Salesforce consulting partner involvement. There is no native CQL execution engine, no HEDIS measure library, and no care gap detection workflow.

Salesforce's strength in patient engagement and care coordination does not translate to quality measurement. The data model is optimized for relationship management (contacts, accounts, activities), not clinical evaluation (observations, conditions, procedures). Building a quality measurement capability on Salesforce would require constructing the entire clinical data layer from scratch, at which point the customer is paying Salesforce licensing fees for a CRM they are not using for CRM purposes.

---

## Market Positioning

HDIM occupies a unique position in the competitive landscape: real-time quality measurement combined with revenue cycle integration and custom measure creation, delivered at mid-market pricing. No competitor combines all three capabilities. Incumbents force customers to choose between real-time evaluation (unavailable), platform breadth (requires multiple vendors), and affordable pricing (excluded by enterprise minimums).

```
                    Real-Time Evaluation
                           |
                           |
              HDIM *       |
                           |
                           |
   Affordable  ────────────┼──────────── Enterprise-Only
                           |
                           |          * Epic    * Optum
              * Arcadia    |
                           |          * Inovalon
              * Salesforce |
                           |
                    Batch Processing
```

The quadrant that HDIM occupies (real-time, affordable) is empty of competitors. This is not because incumbents have not considered it. It is because their existing architectures cannot reach it without a ground-up rebuild. Event sourcing, FHIR-native processing, and database-per-service isolation are prerequisites for this position, and none of them can be retrofitted onto batch ETL systems.

HDIM's go-to-market strategy targets the 2,000+ mid-market health plans that incumbents cannot serve profitably. These organizations need quality measurement capabilities to comply with CMS mandates and compete for value-based contracts, but they cannot justify $1-5M annual fees and 18-month implementations. HDIM's $150-300K price point and 90-day deployment open this segment for the first time.

---

## Key Takeaways for Investors

1. **The moat is architectural, not feature-based.** Competitors cannot close the gap through incremental improvements. Matching HDIM requires rebuilding core systems.

2. **The replication timeline is 24-36 months.** Even with unlimited engineering resources, the integration complexity of event sourcing + FHIR-native + database isolation takes years to build and validate.

3. **The mid-market is uncontested.** Incumbents are structurally unable to serve the $150-300K price point. HDIM expands the market rather than just competing for existing customers.

4. **Platform breadth compounds the advantage.** Each additional capability (revenue cycle, custom measures, real-time evaluation) increases switching costs and widens the competitive gap.

5. **Time is the critical variable.** Every quarter that HDIM operates in production with paying customers builds data, validation, and customer relationships that further widen the moat.

---

*Confidential -- For Investor Use Only | March 2026*
