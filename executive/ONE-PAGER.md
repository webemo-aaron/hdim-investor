# HDIM One-Pager

## What It Is

HDIM is a healthcare platform for real-time quality measurement, care-gap workflows, interoperability, and related operational services.

## Why It Matters

Most healthcare quality and operational workflows remain fragmented across legacy tools, delayed batch processing, and vendor-specific integration models. HDIM is positioned around a more integrated operating model: standards-native data handling, event-driven execution, and shared platform controls across multiple service domains.

## The Product Wedge

HDIM grows along an expanding-wedge model — start with data-quality validation, expand into care-gap and quality workflows, then surface operator-safe intelligence:

- **Data Quality Monitor (DQM)** — the entry wedge. An on-premises data-quality trust authority that scores inbound and outbound healthcare feeds across five dimensions (completeness, conformance, accuracy, consistency, timeliness) and gates feed/identity readiness. DQM runs inside the customer boundary and de-identifies before emitting; no PHI leaves the environment during scoring.
- **Data Motion Platform** — the expansion. A customer-boundary runtime for care-gap detection and HEDIS quality-measure workflows, powered by DQM's validated signals and gated by identity and consent controls.
- **Atlas Nexus** — the operator tier. A cloud evidence layer that ingests operator-safe aggregates (care-gap closures, quality signals, integration readiness, syndromic indicators) so operators triage cross-customer evidence without seeing patient-level PHI (deny-list enforced; small-cell floor, k ≥ 11).

The wedge lets HDIM begin with something every healthcare organization needs — feed quality — and expand into coordinated workflows and cross-customer intelligence without moving PHI outside the customer boundary. The through-line: move the question, not the data.

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
