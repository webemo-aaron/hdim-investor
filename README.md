# HDIM

Real-time healthcare quality, interoperability, and operational workflows.

This repository is the public-facing investor and prospective-company package for HDIM. It is intentionally narrower than the internal diligence set: it focuses on current, evidence-backed platform shape and keeps licensed content, sensitive internals, and customer-specific materials out of the public package.

## What HDIM Is

HDIM is a multi-service healthcare platform organized around:

- standards-native clinical and interoperability services
- event-driven backend execution patterns
- gateway-mediated ingress and shared platform controls
- multi-tenant security, audit, tracing, and persistence foundations
- operational workflows spanning quality, patient, analytics, and related services

The current code-validated platform inventory behind this package includes:

- 77 Gradle-registered backend service modules
- 80 service directories under `backend/modules/services`
- 250 backend controller classes
- 481 Liquibase changelog files (61 of them per-module aggregators, leaving 420 migration changelogs)

Measured by script from `git ls-tree` at commit `ed65a2c9`. Counting rules for each figure are in [executive/FOUNDER.md](executive/FOUNDER.md); they supersede an earlier published set of 59 / 171 / 362, which understated the platform by roughly thirty percent.

This package is designed for fast external review. Deeper architecture and diligence materials are available separately under NDA.

## Start Here

| Audience | Read First | Time |
|---|---|---:|
| Who stands it up, and who runs it after | [executive/FOUNDER.md](executive/FOUNDER.md) | 4 min |
| Executive or investor intro | [executive/ONE-PAGER.md](executive/ONE-PAGER.md) | 2 min |
| General business review | [executive/PITCH-DECK.md](executive/PITCH-DECK.md) | 10 min |
| Platform understanding | [platform/PLATFORM-OVERVIEW.md](platform/PLATFORM-OVERVIEW.md) | 10 min |
| Technical diligence preview | [platform/ARCHITECTURE.md](platform/ARCHITECTURE.md) | 15 min |
| Security and disclosure boundary | [platform/SECURITY-COMPLIANCE.md](platform/SECURITY-COMPLIANCE.md) | 8 min |

## Package Map

### Executive

- [FOUNDER.md](executive/FOUNDER.md) — who stands the deployment up, what is measured, what is not claimed
- [ONE-PAGER.md](executive/ONE-PAGER.md)
- [PITCH-DECK.md](executive/PITCH-DECK.md)
- [FAQ.md](executive/FAQ.md)

### Platform

- [PLATFORM-OVERVIEW.md](platform/PLATFORM-OVERVIEW.md)
- [ARCHITECTURE.md](platform/ARCHITECTURE.md)
- [SECURITY-COMPLIANCE.md](platform/SECURITY-COMPLIANCE.md)

### Traction

- [DEVELOPMENT-VELOCITY.md](traction/DEVELOPMENT-VELOCITY.md)
- [PRODUCT-MILESTONES.md](traction/PRODUCT-MILESTONES.md)
- [PRODUCTION-READINESS.md](traction/PRODUCTION-READINESS.md)

### Technical

- [CODE-SAMPLES.md](technical/CODE-SAMPLES.md)
- [COMPETITIVE-ANALYSIS.md](technical/COMPETITIVE-ANALYSIS.md)
- [DEPLOYMENT-GUIDE.md](technical/DEPLOYMENT-GUIDE.md)

## Public Package Rules

- This repo is public-facing and should remain public-safe.
- It should not include licensed HEDIS content, restricted standards text, customer data, or sensitive source-level internals that are not necessary for external understanding.
- It should not repeat stale fundraising asks, outdated TAM claims, or older platform counts that conflict with the current code-validated inventory.
- Where deeper diligence is appropriate, this package should point readers to NDA-protected follow-up materials rather than embedding restricted detail here.
- **No named prospect organizations.** Prior professional history and the target list overlap, so naming an organization here would disclose pipeline and invite the inference that a commercial relationship exists. Vendor and product names (InterSystems, IRIS/HealthShare, Mirth, Rhapsody, IBM Initiate) are technology, not pipeline, and are allowed.
- **No ask, and no financing content of any kind** — no round, terms, valuation, cap table, use of funds, runway, or investment call to action. This package is written to establish credibility, not to solicit; that is deliberate and is what lets it be published at all.
- **Every published figure carries its counting rule and the commit it was measured at.** A number without one cannot be checked, and this package has already had to retract a stale set that understated the platform by roughly thirty percent.

## Current Positioning

The strongest external narrative for HDIM today is:

1. A substantial healthcare platform exists now, not just a concept.
2. The platform is standards-native and operationally disciplined.
3. The engineering story is stronger when it is evidence-backed and transparent about gaps.
4. Public materials should be conservative; deeper architecture and readiness materials should be shared under NDA.

## Contact

For current diligence materials, product review, or a technical walkthrough, contact the HDIM team directly.

Confidential sharing beyond this package is handled separately.
