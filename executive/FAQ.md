# HDIM FAQ

## What is HDIM?

HDIM is a healthcare platform for real-time quality measurement, care-gap workflows, interoperability, and related operational services. It is best understood as a platform with multiple service domains rather than a single application. See [../platform/PLATFORM-OVERVIEW.md](../platform/PLATFORM-OVERVIEW.md).

The platform is organized as an expanding-wedge product model: **Data Quality Monitor (DQM) → Data Motion Platform → Atlas Nexus**. See the next two questions and [../platform/PLATFORM-OVERVIEW.md](../platform/PLATFORM-OVERVIEW.md).

## What is Data Quality Monitor (DQM)?

DQM is the on-premises data-quality trust authority and the product's entry wedge. It scores inbound and outbound healthcare feeds across five dimensions — completeness, conformance, accuracy, consistency, timeliness — detects baseline drift, and gates feed/identity readiness. DQM runs inside the customer boundary, holds the identified-data authority, and de-identifies before emitting; it sends only operator-safe aggregates outward. Feed-grading is a need common to every data-sharing healthcare organization, which is why it anchors the wedge.

## What is Atlas Nexus?

Atlas Nexus is the cloud operator-evidence tier. It ingests operator-safe aggregates from customer HDIM deployments — care-gap closures, quality signals, integration readiness, and syndromic indicators — so operators can review and triage cross-customer evidence without ever seeing patient-level PHI. Privacy is enforced on both sides: a deny-list of sensitive keys, de-identification before emit, and a small-cell floor (k ≥ 11). Between DQM and Atlas Nexus sits the **Data Motion Platform**, the customer-boundary runtime that runs care-gap and HEDIS quality workflows on DQM-validated signals under identity and consent controls.

## What is actually built today?

The current code-validated inventory supports a substantial multi-service platform: 77 Gradle-registered backend service modules, 80 service directories, 250 backend controller classes, and 481 Liquibase changelog files. Measured by script from `git ls-tree` at commit `ed65a2c9`. Counting rules for each figure are in [FOUNDER.md](FOUNDER.md); they supersede an earlier published set of 59 / 171 / 362, which understated the platform by roughly thirty percent.

## What is technically distinctive?

The strongest current differentiators are standards-native healthcare architecture, event-driven execution patterns, shared multi-tenant platform controls, and evidence-backed engineering discipline. See [../platform/ARCHITECTURE.md](../platform/ARCHITECTURE.md).

## Is this a public package or a diligence package?

This repository is the public-safe package. It is designed for first review and avoids licensed content, sensitive internal details, and deeper diligence synthesis that should be shared under NDA.

## Why not include deeper technical detail here?

Because the strongest current process separates public-safe material from NDA-protected diligence. That reduces IP leakage, avoids licensed-content issues, and keeps the public story aligned with approved evidence. See [../platform/SECURITY-COMPLIANCE.md](../platform/SECURITY-COMPLIANCE.md).

## What should a technical reviewer focus on first?

Start with:

1. [../platform/PLATFORM-OVERVIEW.md](../platform/PLATFORM-OVERVIEW.md)
2. [../platform/ARCHITECTURE.md](../platform/ARCHITECTURE.md)
3. [../traction/PRODUCTION-READINESS.md](../traction/PRODUCTION-READINESS.md)

## Is the platform fully contract-coherent?

Not yet. The current internal review concluded that the repo has strong building blocks but still needs more contract governance and frontend/API normalization. That work is underway, and the public package avoids overstating maturity in that area.

## What should not be inferred from this repo?

Do not infer:

- unrestricted rights to licensed clinical content
- stale fundraising asks or market-size claims from older materials
- that every internal API is already published as a public contract artifact

## What is available under NDA?

The NDA package can include a technical brief, diligence FAQ, readiness materials, and deeper architecture/risk framing. This public repo is only the first layer.
