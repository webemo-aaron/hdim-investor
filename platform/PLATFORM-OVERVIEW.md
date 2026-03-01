# HDIM Platform Overview

**Every layer built. Every layer tested. Every layer hardened.**

---

## Executive Summary

HDIM is a complete healthcare quality measurement and operations platform spanning data ingestion through clinical dashboards. Every layer -- from FHIR R4 data handling to CMO onboarding workflows -- is production-tested with 1,171+ test classes across 694 test files and zero regressions. This document walks through each platform capability and its current status.

---

## Platform Architecture

HDIM is a six-layer platform. Each layer is independently deployable, independently testable, and connected through an event-driven backbone that provides real-time processing and complete audit trails.

```
Clinical Portal & Dashboards (Angular 17 -- 12 frontend applications)
    |
API Gateways (4 modularized gateways with shared core)
    |
Application Services (59 microservices -- Java 21, Spring Boot 3.x)
    |
Event Backbone (Apache Kafka + Event Sourcing -- 4 event service pairs)
    |
Data Layer (PostgreSQL 16 -- 29 independent databases, Redis 7 cache)
    |
External Systems (FHIR R4 -- Epic, Cerner, Athena, any R4-compliant EHR)
```

Every layer communicates through well-defined contracts. Services publish events to Kafka, event handlers project those events into read-optimized stores, and gateways enforce authentication, multi-tenancy, and rate limiting at the boundary. The result is a platform where adding a new capability does not require modifying existing services.

---

## Platform Capabilities

### 1. Quality Measure Engine
**Status: Production**

The Quality Measure Engine is the core of HDIM. It evaluates 80+ HEDIS quality measures using direct CQL (Clinical Quality Language) execution against FHIR R4 clinical resources. Each evaluation completes in under 2 seconds per patient, with population-level batch evaluation supporting 10 concurrent processing threads. This is not a reporting tool that runs overnight -- it evaluates measures in real time as clinical data arrives.

This capability matters because it eliminates the intervention delay that costs health plans revenue. Traditional quality measurement platforms run batch processes nightly or weekly, meaning a care gap identified on Monday might not surface until the following week. HDIM evaluates the moment a clinical event is recorded. For health plans managing value-based contracts, the difference between same-day and next-week intervention translates directly to compliance rates and revenue.

The engine supports 7 custom measure metadata columns for flexible measure configuration, enabling health plans to extend standard HEDIS measures with organization-specific parameters. The quality-measure-service, cql-engine-service, and quality-measure-event-service are all independently tested with dedicated integration test suites. All 1,171 tests pass with zero regressions.

### 2. Care Gap Detection
**Status: Production**

HDIM automatically identifies gaps in patient care based on quality measure evaluation results. Unlike retrospective systems that analyze claims data months after the fact, HDIM detects care gaps in real time as clinical events flow through the platform. When a patient misses a screening or a lab result indicates a needed intervention, the system identifies the gap immediately.

This capability addresses a direct revenue problem. For health plans operating under value-based contracts, each 1% gap in compliance can represent $500K to $2M in lost revenue. Real-time detection means clinical teams can intervene on the same day rather than discovering the gap during a quarterly review. The care gap service includes closure workflows so clinical teams can track interventions from identification through resolution.

The care-gap-service, care-gap-event-service, and care-gap-event-handler-service form a complete event-sourced pipeline. Patient engagement integration enables automated outreach for identified gaps. The service includes 17 documented API endpoints with full OpenAPI 3.0 specifications and interactive Swagger UI testing.

### 3. Revenue Cycle (Wave-1)
**Status: Complete (February 2026)**

The first wave of revenue cycle capabilities extends HDIM from quality measurement into revenue operations. This includes claims processing with clearinghouse integration, remittance reconciliation via ERA/835 processing, price transparency estimation and publishing APIs, and ADT (Admit-Discharge-Transfer) event handling. The clearinghouse communication layer includes configurable retry backoff behavior to handle transient failures without data loss.

This expansion matters strategically because competitors offer quality measurement and revenue cycle as separate products requiring separate integrations, separate data pipelines, and separate contracts. HDIM delivers both on the same event-driven backbone. A clinical event that triggers a quality measure evaluation simultaneously flows through revenue cycle processing. Health plans get a single platform instead of stitching together point solutions.

Revenue cycle services are built on the same event sourcing architecture as quality measurement, ensuring complete audit trails for every financial transaction. The payer-workflows-service, prior-auth-service, and cost-analysis-service each maintain independent databases with Liquibase-managed schemas and full rollback coverage.

### 4. FHIR R4 Interoperability
**Status: Production**

HDIM handles clinical data natively as FHIR R4 resources. Data enters the system as FHIR, is processed as FHIR, and exits as FHIR. There is no ETL translation layer that degrades clinical fidelity. This approach preserves 100% of clinical attributes -- compared to the 30-50% attribute preservation typical of platforms that translate FHIR into proprietary internal formats.

FHIR R4 compliance is now federally mandated for healthcare interoperability. HDIM's native approach means zero data degradation during integration with Epic, Cerner, Athena, or any FHIR R4-compliant EHR system. For quality measurement specifically, this matters because CQL evaluation requires complete clinical resources. A platform that loses attributes during translation produces inaccurate measure results.

The FHIR service exposes 26 documented API endpoints through OpenAPI 3.0, with interactive Swagger UI for development and testing. The data ingestion pipeline handles ADT events and clinical data feeds through the ehr-connector-service and data-ingestion-service. A total of 62 production-ready API endpoints are documented across all services with JWT Bearer authentication integration, request/response examples in FHIR R4 format, and HIPAA compliance notes.

### 5. Custom Measure Builder
**Status: Production (February 2026)**

The Custom Measure Builder is a web-based UI that enables health plan quality teams to create, configure, and deploy custom quality measures without engineering involvement. The interface includes a metadata dialog with 7 configurable fields, allowing teams to define measure parameters, population criteria, and reporting requirements through a guided workflow.

This capability exists because health plans have proprietary quality measures beyond the standard HEDIS set. A health plan managing a specific chronic disease management program or a regional quality initiative needs measures that do not exist in any standard library. Without a self-service builder, every custom measure requires a vendor engineering engagement -- typically weeks of lead time and significant cost.

The measure builder is built as a micro-frontend (mfe-measure-builder) within the Angular application shell. It includes a Create Measure page, metadata configuration dialog, and API error parsing for clear user feedback. End-to-end test coverage validates the complete workflow from measure creation through deployment. A focused CI workflow ensures measure builder changes are validated independently.

### 6. CMO Onboarding
**Status: Production (February 2026)**

The CMO Onboarding system provides structured dashboard workflows for health plan Chief Medical Officers evaluating and adopting HDIM. Rather than relying on ad-hoc product demos and manual setup, CMOs follow a guided evaluation-to-adoption path with defined milestones, acceptance playbooks, and automated validation hooks that verify each onboarding stage is complete.

This capability reduces sales-to-deployment friction. In enterprise healthcare sales, the CMO is typically the final decision-maker. A structured evaluation workflow with clear milestones gives the CMO confidence in the platform while reducing the vendor's sales cycle from months to weeks. Each onboarding milestone is verifiable, creating a shared understanding of progress between the buyer and the HDIM team.

The onboarding implementation includes dashboard workflow components, acceptance playbooks for structured evaluation, and validation hooks that programmatically verify milestone completion. This system is designed to scale -- each new health plan customer follows the same structured path, ensuring consistent onboarding quality regardless of team size.

### 7. Clinical Portal
**Status: Production**

The Clinical Portal is an Angular 17 web application built for clinical quality teams who interact with HDIM daily. It includes 12 frontend applications organized as a micro-frontend architecture: a shell application, clinical portal, admin portal, and dedicated micro-frontends for care gaps, patients, quality measures, reports, and measure building.

HIPAA compliance is engineered into the frontend, not bolted on after the fact. ESLint enforces a zero-tolerance policy on console.log statements -- the build fails if any are detected. All logging routes through a LoggerService that automatically filters Protected Health Information in production. The session timeout system enforces 15-minute idle logout with 2-minute warning, compliant with HIPAA section 164.312(a)(2)(iii). Every HTTP API call is automatically audit-logged via an interceptor, providing 100% coverage without requiring developers to add manual logging.

The portal includes a global error handler that prevents application crashes while logging security incidents. Accessibility coverage includes 343 ARIA attributes across 53 files, establishing a baseline for WCAG 2.1 Level A compliance. Operations dashboards provide real-time quality monitoring for clinical teams managing patient populations.

### 8. Operations Orchestration
**Status: Production (February 2026)**

The operations orchestration layer consists of a 16-class gateway framework distributed across 4 modularized API gateways: API Gateway, Clinical Gateway, Admin Gateway, and FHIR Gateway. These gateways share a common gateway-core module that eliminates code duplication while allowing each gateway to enforce domain-specific routing and security policies.

This architecture matters because enterprise healthcare operations require consistent security, authentication, and multi-tenancy enforcement across all service boundaries. When a new microservice is added to the platform, it automatically inherits the full operations stack -- JWT validation, tenant isolation, rate limiting, header security hardening, and audit logging -- without any per-service configuration. The gateway-core module is referenced by 9 services across the platform.

Header security hardening ensures that all gateway traffic meets enterprise security requirements. Authentication flows validate JWT tokens at the gateway boundary and inject trusted headers downstream, so individual services never handle raw authentication tokens. Multi-tenancy is enforced at the gateway level, ensuring that every request is associated with a verified tenant before reaching any application service.

### 9. Security and Compliance
**Status: Hardened**

Security in HDIM is not a feature -- it is an architectural constraint enforced at every layer. HIPAA section 164.312 compliance is built into the platform through engineering controls, not policy documents. Every PHI cache entry expires within 5 minutes. Every PHI access method carries an audit annotation. Every database query filters by tenant ID. Every API endpoint requires role-based authorization. These constraints are enforced by automated tests, not manual review.

The platform implements a defense-in-depth approach to vulnerability management. CVE remediation follows a wave-based burn-down tracking process, ensuring that vulnerabilities are systematically addressed rather than triaged into a backlog. ZAP security scanning baselines run on every pull request, catching regressions before they reach production. A 360-degree platform assurance process requires evidence sign-off across security, compliance, and operational readiness dimensions.

Multi-tenant isolation operates at three levels: database (separate schemas with tenant-filtered queries), cache (tenant-scoped Redis keys with enforced TTLs), and application (gateway-enforced tenant headers on every request). The HTTP audit interceptor provides 100% API call coverage, creating a complete audit trail of every clinical data access. Healthcare buyers evaluating HDIM receive auditable evidence of security posture at every layer -- not marketing claims, but test results, scan reports, and compliance artifacts.

For full security architecture and compliance details, see the [Security and Compliance](SECURITY-COMPLIANCE.md) document.

---

## Integration Architecture

HDIM connects its layers through an event-driven backbone built on Apache Kafka. Clinical events -- patient admissions, lab results, care gap closures, measure evaluations -- flow through an immutable event store that enables real-time processing and maintains a complete audit trail. The event sourcing architecture uses dedicated event service pairs (producer and handler) for each domain: patient events, care gap events, quality measure events, and clinical workflow events. This design means every state change in the system is captured as an immutable event, enabling replay, audit, and temporal queries.

Data enters and exits the platform as native FHIR R4 resources. The EHR connector service and data ingestion service accept clinical data from external systems without format translation, preserving the full clinical context that downstream services -- particularly the CQL evaluation engine -- require for accurate processing. Internal service communication uses both synchronous REST (via Feign clients with automatic OpenTelemetry trace propagation) and asynchronous Kafka events, choosing the appropriate pattern based on whether the interaction requires immediate response or eventual consistency.

---

## What's Next

- **Q2 2026:** First pilot deployments with health plan partners, targeting 1-2 production go-lives with dedicated success management.
- **EHR connector expansion:** Additional FHIR R4 integrations beyond Epic, Cerner, and Athena to cover regional and specialty EHR systems.
- **Expanded HEDIS measure library:** Target 100+ standard measures with validated CQL definitions, up from the current 80+.
- **Population health analytics dashboards:** Advanced cohort analysis, trend visualization, and predictive risk stratification for health plan leadership.
- **Additional revenue cycle modules:** Prior authorization automation, eligibility verification, and enhanced claims analytics.

---

*Confidential -- For Investor Use Only | March 2026*
