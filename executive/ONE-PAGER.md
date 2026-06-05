# HDIM One-Pager

## What It Is

HDIM is a healthcare platform for real-time quality measurement, care-gap workflows, interoperability, and related operational services.

## Why It Matters

Most healthcare quality and operational workflows remain fragmented across legacy tools, delayed batch processing, and vendor-specific integration models. HDIM is positioned around a more integrated operating model: standards-native data handling, event-driven execution, and shared platform controls across multiple service domains.

## What Exists Today

- 59 Gradle-managed backend service modules
- 61 service directories under the backend services tree
- gateway services for admin, clinical, and FHIR ingress
- clinical, patient, quality, consent, interoperability, analytics, workflow, and platform services
- shared security, audit, tracing, persistence, and API-contract infrastructure

## What Makes It Distinctive

- Standards-native healthcare architecture with FHIR-centered service patterns
- Event-driven and event-sourced patterns in key domains
- Shared multi-tenant platform controls rather than ad hoc per-service foundations
- Stronger-than-usual engineering evidence through inventories, validation artifacts, and architecture documentation

## How To Review

- Executive summary: [PITCH-DECK.md](PITCH-DECK.md)
- Platform walkthrough: [../platform/PLATFORM-OVERVIEW.md](../platform/PLATFORM-OVERVIEW.md)
- Technical architecture preview: [../platform/ARCHITECTURE.md](../platform/ARCHITECTURE.md)
- Security and disclosure boundary: [../platform/SECURITY-COMPLIANCE.md](../platform/SECURITY-COMPLIANCE.md)

## Share Boundary

This one-pager is public-safe. Deeper technical diligence, readiness evidence, and controlled-content discussions should move to the NDA package rather than be expanded in this public repo.
