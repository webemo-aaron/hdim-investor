# HDIM FAQ

## What is HDIM?

HDIM is a healthcare platform for real-time quality measurement, care-gap workflows, interoperability, and related operational services. It is best understood as a platform with multiple service domains rather than a single application. See [../platform/PLATFORM-OVERVIEW.md](../platform/PLATFORM-OVERVIEW.md).

## What is actually built today?

The current code-validated inventory supports a substantial multi-service platform: 59 Gradle-managed backend service modules, 61 service directories, domain gateways, and shared infrastructure for security, audit, tracing, and persistence. See [../traction/PRODUCT-MILESTONES.md](../traction/PRODUCT-MILESTONES.md).

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
