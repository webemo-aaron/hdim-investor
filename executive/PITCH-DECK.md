# HDIM Pitch Deck

## Slide 1: HDIM

Real-time healthcare quality, interoperability, and operational workflows.

## Slide 2: The Problem

- Healthcare operations are still fragmented across quality, patient, interoperability, and workflow systems.
- Many operating models depend on delayed, batch-oriented processing and brittle vendor-specific integrations.
- Technical and compliance due diligence is slowed when platform truth is spread across multiple partial systems.

## Slide 3: The Platform

HDIM should be understood as a platform, not a single application.

- standards-native clinical and interoperability services
- event-driven backend execution
- domain gateways for ingress
- shared security, audit, tracing, and persistence controls
- supporting analytics, workflow, and AI-adjacent services

## Slide 3b: The Product Wedge

An expanding-wedge model: land with feed quality, expand into workflows, surface operator-safe intelligence.

- **Data Quality Monitor (DQM)** — on-prem data-quality trust authority; scores inbound/outbound feeds across five dimensions and gates feed/identity readiness. Holds the identified-data authority; de-identifies before emitting. *(The land: feed-grading is a need every data-sharing organization shares.)*
- **Data Motion Platform** — customer-boundary runtime for care-gap detection and HEDIS quality workflows, on DQM-validated signals, under identity and consent controls. *(The expand.)*
- **Atlas Nexus** — cloud operator-evidence tier; cross-customer operator inbox, syndromic surveillance, and integration-readiness signals. No patient-level PHI: de-identified, deny-list + small-cell floor (k ≥ 11). *(The operating layer.)*

Through-line: move the question, not the data — PHI never has to leave the customer boundary.

## Slide 4: What Is Built

Current code-validated platform inventory:

- 77 Gradle-registered backend service modules
- 80 backend service directories
- 250 backend controller classes
- 481 Liquibase changelog files (420 migration changelogs plus 61 aggregators)
- shared domain, infrastructure, and API-contract modules

## Slide 5: Why It Is Credible

- generated service inventory exists
- dependency mapping exists
- shared OpenAPI validation infrastructure exists
- consumer/provider contract testing exists in the patient domain
- release and readiness materials exist outside the public package

## Slide 6: Why It Is Technically Different

- standards-native architecture rather than a purely proprietary internal model
- event-driven and event-sourced patterns in key service domains
- shared multi-tenant platform controls
- code-validated architecture documentation rather than pitch-only architecture copy

## Slide 7: Product Scope

HDIM currently spans:

- patient and clinical services
- quality and care-gap services
- consent and gateway services
- interoperability and ingestion services
- analytics and workflow services
- platform and admin services

## Slide 8: Public Versus NDA Story

Public-safe story:

- platform exists
- platform breadth is real
- architecture themes are defensible

NDA story:

- deeper architecture synthesis
- readiness evidence
- contract-governance cleanup details
- diligence FAQ and technical brief

## Slide 9: Governance and Transparency

A credible external story requires:

- evidence-backed claims
- explicit licensing boundaries
- disclosure of documentation-governance gaps instead of hiding them
- separation of public-safe and NDA-protected materials

## Slide 10: Current Cleanup Work

Recent internal work has focused on:

- contract publication truthfulness
- patient-service route-family normalization
- shared contract and shared infrastructure documentation
- docs-to-artifact validation gates

## Slide 11: What Comes Next

- frontend endpoint normalization
- stronger shared contract module governance
- broader contract drift automation
- runtime-adjacent inventory integration

## Slide 12: Next Step

Use this repo for first review. Move to NDA-protected materials for deeper technical diligence.
