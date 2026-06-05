# Platform Overview

## Summary

HDIM is a healthcare platform organized around gateways, domain services, and shared control layers. The current external story should focus on platform shape and operational discipline rather than older pitch-era metrics.

## Platform Shape

At a high level, HDIM operates across these layers:

1. External edge and domain gateways
2. Clinical, patient, quality, interoperability, analytics, workflow, and platform services
3. Event-driven coordination patterns in key domains
4. Shared security, audit, tracing, persistence, and API-contract modules

This structure matters because the platform is designed as a coordinated service system rather than a single-purpose point product.

## Current Scope

Current code-validated inventory includes:

- 59 Gradle-managed backend service modules
- 61 backend service directories
- gateway services for admin, clinical, and FHIR ingress
- 4 shared API-contract modules
- 15 shared infrastructure modules

## Capability Areas

### Clinical and Patient

Patient, clinical workflow, nursing workflow, prior auth, HCC, and related services support the patient and clinical operating model.

### Quality and Care Gap

Quality-measure, care-gap, and CQL-engine services support quality evaluation and related workflows.

### Interoperability and Ingestion

FHIR, EHR connector, data ingestion, and related adapter services support standards-native integration and data movement.

### Analytics and Workflow

Analytics, predictive analytics, payer workflows, migration workflows, and related services support reporting and operational processes.

### Platform and Controls

Gateways, approval, notification, admin, audit, tracing, persistence, and security modules provide the shared operating foundation.

## Public View of Value

The public-safe value proposition is not a single narrow feature. It is the combination of:

- standards-native healthcare architecture
- event-driven execution in important service domains
- shared controls across a broad service platform
- evidence-backed documentation and validation work

## How To Read The Rest Of This Repo

- Architecture deep dive: [ARCHITECTURE.md](ARCHITECTURE.md)
- Security and disclosure boundary: [SECURITY-COMPLIANCE.md](SECURITY-COMPLIANCE.md)
- Product and capability evolution: [../traction/PRODUCT-MILESTONES.md](../traction/PRODUCT-MILESTONES.md)
- Execution proof and readiness: [../traction/PRODUCTION-READINESS.md](../traction/PRODUCTION-READINESS.md)
