# Deployment Guide

**Enterprise deployment procedures for HDIM platform**
**Target deployment time: 90 days from contract to production**

---

## Deployment Overview

HDIM deploys as a containerized microservices platform using Docker Compose or Kubernetes. The 90-day deployment timeline includes environment setup, EHR integration, data migration, validation, and training.

### Deployment Timeline

| Phase | Duration | Activities |
|-------|----------|------------|
| Week 1-2 | Environment Setup | Infrastructure provisioning, network configuration, security review |
| Week 3-4 | Platform Deployment | Docker/K8s deployment, database initialization, service validation |
| Week 5-8 | EHR Integration | FHIR R4 connection, data mapping, initial data load |
| Week 9-10 | Validation | Quality measure testing, care gap verification, performance benchmarking |
| Week 11-12 | Training & Go-Live | Staff training, operational handoff, production cutover |

---

## System Requirements

### Minimum Hardware

| Component | Requirement |
|-----------|-------------|
| CPU | 16+ cores |
| RAM | 64 GB |
| Storage | 500 GB SSD |
| Network | 100 Mbps minimum |

### Software Prerequisites

| Component | Version |
|-----------|---------|
| Docker | 24.0+ |
| Docker Compose | 2.20+ |
| PostgreSQL | 16 |
| Java Runtime | 21 (LTS) |
| Node.js | 18+ (for Clinical Portal) |

---

## Platform Components

59 microservices organized across functional domains:

| Domain | Services | Key Components |
|--------|----------|----------------|
| Clinical Quality | 8 | Quality Measure Engine, CQL Engine, Care Gap Service |
| Patient Data | 6 | Patient Service, Health Record, Demographics |
| FHIR Interoperability | 4 | FHIR Service, Data Ingestion, Resource Validation |
| Revenue Cycle | 4 | Payer Workflows, Claims, Remittance, Price Transparency |
| Event Processing | 4 | Patient Events, Care Gap Events, Quality Events, Risk Events |
| Clinical Portal | 3 | Angular Frontend, API Gateway, Operations Dashboard |
| Security & Auth | 4 | Gateway Auth, Audit Service, Consent Service, Identity |
| Infrastructure | 8 | Config Server, Service Discovery, Monitoring, Logging |
| Analytics | 4 | Reporting, Population Health, Risk Stratification, Analytics |
| Administration | 4 | Admin Portal, User Management, Tenant Management, System Config |

---

## Quick Start Deployment

### Step 1: Clone and Configure

```bash
git clone <repository-url>
cd hdim-platform
cp .env.example .env
# Configure: database credentials, Kafka, Redis, tenant settings
```

### Step 2: Start Infrastructure

```bash
docker compose up -d postgres redis kafka
# Wait for health checks
docker compose ps
```

### Step 3: Initialize Databases

```bash
# Liquibase migrations run automatically on service startup
# 29 databases, 199 changesets, all with rollback support
docker compose up -d <service-name>
```

### Step 4: Deploy Services

```bash
# Start all services
docker compose up -d

# Verify health
curl http://localhost:8001/actuator/health
```

### Step 5: Validate Deployment

```bash
# Run validation script
./scripts/validate-system.sh

# Check API documentation
open http://localhost:8084/patient/swagger-ui/index.html
```

---

## EHR Integration

### Supported EHR Systems

| EHR | Integration Method | Status |
|-----|-------------------|--------|
| Epic | FHIR R4 API | Production-ready |
| Cerner (Oracle Health) | FHIR R4 API | Production-ready |
| Athena | FHIR R4 API | Production-ready |
| Any FHIR R4 compliant | FHIR R4 API | Compatible |

### Integration Architecture

```
EHR System --> FHIR R4 API --> HDIM Data Ingestion --> Event Processing --> Quality Evaluation
```

HDIM consumes FHIR R4 resources natively -- no ETL layer, no data translation. Integration requires configuring the FHIR endpoint URL and authentication credentials.

---

## Security Configuration

### HIPAA Compliance Setup

- [x] TLS 1.3 for all service communication
- [x] JWT authentication at API gateway
- [x] Multi-tenant isolation configured
- [x] Audit logging enabled (100% API coverage)
- [x] Session timeout: 15-minute idle
- [x] PHI cache TTL: 5 minutes maximum
- [x] Encryption at rest for database volumes

### Security Verification

```bash
# Run security validation
./scripts/validate-security.sh

# Verify ZAP scan baseline
# Verify CVE remediation status
# Verify compliance evidence gates
```

---

## Wave-1 Revenue Cycle Deployment

Additional configuration for revenue cycle capabilities:

- Claims processing endpoint configuration
- Clearinghouse connection setup (with retry backoff)
- Remittance reconciliation (ERA/835) mapping
- Price transparency API configuration
- ADT event listener setup

### Validation

```bash
# Wave-1 assurance runner
./scripts/wave-1-assurance.sh

# Validates:
# - p95 performance budgets
# - Clearinghouse connectivity
# - ADT payload handling
# - Preflight health checks
```

---

## Performance Benchmarks

| Operation | Target | Measured |
|-----------|--------|----------|
| Single patient HEDIS evaluation | <2 seconds | ~1.2 seconds |
| Population batch (100 patients) | <30 seconds | ~25 seconds |
| Care gap detection (per event) | <500ms | ~200ms |
| API response (cached) | <100ms | ~50ms |
| Service startup | <30 seconds | ~15 seconds |

---

## Monitoring Setup

| Component | Tool | Access |
|-----------|------|--------|
| Metrics | Prometheus | http://localhost:9090 |
| Dashboards | Grafana | http://localhost:3001 |
| Tracing | OpenTelemetry + Jaeger | http://localhost:16686 |
| Logs | Centralized logging | Docker logs / ELK stack |
| Alerts | Prometheus AlertManager | Configurable thresholds |

---

## Support & SLAs

| Priority | Response Time | Resolution Time |
|----------|--------------|-----------------|
| P1 (Critical) | 1 hour | 4 hours |
| P2 (High) | 4 hours | 1 business day |
| P3 (Medium) | 1 business day | 3 business days |
| P4 (Low) | 2 business days | 1 week |

---

## Customer ROI

| Metric | Value |
|--------|-------|
| Annual savings | $722K per organization |
| ROI | 1,720% |
| Payback period | 22 days |
| FTE reduction | 40-60% quality team automation |
| HEDIS incentive capture | Improved by real-time gap closure |

---

*Confidential -- For Investor Use Only | March 2026*
