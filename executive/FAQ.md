# Investor FAQ

**HDIM — HealthData-in-Motion**
*Common questions from investors, organized by category*

---

## Business & Market

**Q: What is HDIM?**
A: HDIM is a production-ready SaaS platform that evaluates HEDIS healthcare quality measures in real-time and detects care gaps as they occur. Unlike incumbent solutions that batch-process overnight, HDIM delivers results in under 2 seconds via event-driven architecture. [Full platform overview →](../platform/PLATFORM-OVERVIEW.md)

**Q: What's the market opportunity?**
A: $18B annual spend on healthcare quality measurement, growing 12%+ annually as CMS expands value-based care mandates. HDIM targets the $2-3B automation opportunity across 2,000+ health plans, ACOs, and health systems. [Pitch deck, Slide 4 →](PITCH-DECK.md)

**Q: What's the business model?**
A: SaaS platform at $150-300K/year per customer, scaling with patient volume and measures. 70-80% gross margins, <$100K CAC, $2-4M LTV. Implementation fee of $50-100K one-time. [Pitch deck, Slide 6 →](PITCH-DECK.md)

**Q: How much are you raising?**
A: $5-7M Series A for sales team buildout, customer success, and 3-5 pilot deployments. 18-month runway to profitability. [Pitch deck, Slide 10 →](PITCH-DECK.md)

**Q: What's the customer ROI?**
A: $722K annual savings per customer (1,720% ROI) with 22-day payback period. Savings come from FTE reduction in manual quality measurement and increased HEDIS incentive capture.

**Q: What are your financial projections?**
A: Year 1: $2-4M ARR (3-5 customers), Year 3: $20-30M ARR (25-35 customers), Year 5: $40-60M ARR. Break-even by Year 2-3. [Pitch deck, Slide 9 →](PITCH-DECK.md)

---

## Technical & Product

**Q: Is this actually production-ready?**
A: Yes. 59 microservices, 1,171 test classes (8,000+ methods), 62 documented API endpoints, zero regressions. This is not a prototype — it's an enterprise platform with 7 completed infrastructure phases. [Production readiness checklist →](../traction/PRODUCTION-READINESS.md)

**Q: What's your security posture?**
A: HIPAA compliance was engineered into the architecture, not retrofitted. We run CVE remediation waves with burn-down tracking, ZAP security scanning on every PR, and 360 platform assurance with evidence sign-off. [Security & compliance →](../platform/SECURITY-COMPLIANCE.md)

**Q: How do you handle HIPAA?**
A: Every architectural layer enforces HIPAA: 5-minute cache TTL for PHI, encryption in transit (TLS 1.3), automatic session audit logging, PHI filtering on browser console, 100% API call audit coverage, multi-tenant database isolation. [Security & compliance →](../platform/SECURITY-COMPLIANCE.md)

**Q: What's the technical moat?**
A: Event sourcing + CQRS architecture takes 18-24 months to replicate. Add FHIR R4 native execution (9-12 months), 80+ CQL measures (12-18 months), revenue cycle integration (12 months), and database-per-service (12 months). Replicating any one component is possible — replicating all of them working together takes 2+ years. [Architecture →](../platform/ARCHITECTURE.md)

**Q: How fast can you evaluate quality measures?**
A: Under 2 seconds for a single patient evaluation. Performance breakdown: FHIR retrieval (10ms cached), CQL execution (~1.2s), event publishing (15ms). Supports 10 concurrent threads for population-level processing.

**Q: Can customers create their own measures?**
A: Yes. The custom measure builder UI shipped in February 2026. Health plans can create, configure metadata, and deploy custom quality measures without engineering involvement. [Product milestones →](../traction/PRODUCT-MILESTONES.md)

---

## New Capabilities (February 2026)

**Q: What is Wave-1 Revenue Cycle?**
A: HDIM now handles claims processing, remittance reconciliation, and price transparency — shipped in February 2026. This expands the platform beyond quality measurement into revenue operations, a capability competitors offer as separate products (if at all). [Platform overview →](../platform/PLATFORM-OVERVIEW.md)

**Q: What is CMO Onboarding?**
A: Dashboard workflows and acceptance playbooks designed for health plan Chief Medical Officers. Provides a structured path from platform evaluation to operational adoption, reducing the sales-to-deployment friction.

**Q: What is the Operations Orchestration framework?**
A: A 16-class gateway operations framework that standardizes enterprise operations patterns across all 4 API gateways. This provides consistent security, rate limiting, and multi-tenancy enforcement without per-service configuration.

---

## Competition

**Q: What if Epic/Optum/Salesforce decide to compete directly?**
A: They would need 18-24 months minimum to replicate the event-sourcing architecture, FHIR-native execution, and 80+ pre-built CQL measures. By then, HDIM would have 10+ customers and $1-2M ARR. Their core business models also create structural barriers — Epic won't build EHR-agnostic solutions, Optum bundles with UnitedHealth, and Salesforce lacks clinical depth. [Competitive analysis →](../technical/COMPETITIVE-ANALYSIS.md)

**Q: How are you different from Arcadia or Inovalon?**
A: Arcadia and Inovalon rely on batch ETL processing — they extract, transform, and load data overnight. HDIM evaluates in real-time on native FHIR resources with zero data transformation loss. Arcadia also lacks revenue cycle integration and custom measure creation. [Competitive analysis →](../technical/COMPETITIVE-ANALYSIS.md)

**Q: Why can't a hospital just build this internally?**
A: Some try. The event sourcing architecture alone takes 18-24 months with a senior team of 5-10 engineers. Add FHIR R4 compliance, HIPAA engineering, 80+ CQL measures, and regulatory certification — internal builds typically cost $3-5M and take 3+ years. HDIM delivers comparable capability in 90 days at $150-300K/year.

---

## Risk

**Q: What's the biggest risk?**
A: Go-to-market execution — hiring the right VP Sales and converting pilots to contracts. Technical risk is low (product exists, tested, HIPAA-compliant). Market risk is low (problem is urgent, CMS mandates creating demand).

**Q: Why should we believe this is achievable?**
A: The hard part is done — production-ready code with 1,171 test classes, HIPAA compliance, 59 services, and now revenue cycle capabilities. What remains is repeatable go-to-market execution with proven enterprise sales motions.

**Q: What's your exit strategy?**
A: Likely acquirers include UnitedHealth (Optum), Elevance Health, Cigna, or standalone IPO. Target exits of $200-400M at 9-13x return on Series A. Timeline: 5-7 years. [Pitch deck, Slide 10 →](PITCH-DECK.md)

---

## Response Matrix

| Question Category | Primary Document | Section |
|-------------------|-----------------|---------|
| What is HDIM? | [Platform Overview](../platform/PLATFORM-OVERVIEW.md) | Full document |
| Market opportunity | [Pitch Deck](PITCH-DECK.md) | Slide 4 |
| Business model / pricing | [Pitch Deck](PITCH-DECK.md) | Slide 6 |
| Financial projections | [Pitch Deck](PITCH-DECK.md) | Slide 9 |
| Funding ask | [Pitch Deck](PITCH-DECK.md) | Slide 10 |
| Technical architecture | [Architecture](../platform/ARCHITECTURE.md) | Full document |
| Security / HIPAA | [Security & Compliance](../platform/SECURITY-COMPLIANCE.md) | Full document |
| Production readiness | [Production Readiness](../traction/PRODUCTION-READINESS.md) | Full document |
| Competitive positioning | [Competitive Analysis](../technical/COMPETITIVE-ANALYSIS.md) | Full document |
| Code quality | [Code Samples](../technical/CODE-SAMPLES.md) | Full document |
| Deployment speed | [Deployment Guide](../technical/DEPLOYMENT-GUIDE.md) | Full document |
| Recent progress | [Product Milestones](../traction/PRODUCT-MILESTONES.md) | Full document |
| Development velocity | [Development Velocity](../traction/DEVELOPMENT-VELOCITY.md) | Full document |

---

*Confidential — For Investor Use Only | March 2026*
