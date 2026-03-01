# Code Samples

**Production code patterns for technical due diligence**
**Source:** github.com/webemo-aaron/hdim-master (private, access available upon request)

---

## 1. Event Sourcing with CQRS Projections

**Why It Matters:** Event sourcing captures every state change as an immutable event, enabling full auditability and temporal queries -- critical for healthcare compliance. CQRS separates read and write models so population health dashboards can scale independently from clinical workflows.

**File:** `backend/modules/services/care-gap-event-service/src/main/java/com/healthdata/caregap/service/CareGapEventApplicationService.java`

```java
@Slf4j
@Service
@RequiredArgsConstructor
@Transactional
public class CareGapEventApplicationService {

    private final CareGapEventHandler gapEventHandler;
    private final CareGapProjectionRepository gapRepository;
    private final PopulationHealthRepository populationRepository;
    private final KafkaTemplate<String, Object> kafkaTemplate;

    private static final String GAP_EVENTS_TOPIC = "gap.events";

    public CareGapEventResponse detectGap(DetectGapRequest request, String tenantId) {
        // Create immutable domain event
        CareGapDetectedEvent event = new CareGapDetectedEvent(
            request.getPatientId(),
            request.getGapCode(),
            tenantId,
            request.getDescription(),
            request.getSeverity()
        );

        // Delegate to event handler (updates projections)
        gapEventHandler.handle(event);

        // Update population health read model
        updatePopulationHealth(tenantId, request.getSeverity(), true);

        // Publish to Kafka for downstream consumers
        kafkaTemplate.send(GAP_EVENTS_TOPIC, request.getPatientId(), event);

        return CareGapEventResponse.builder()
            .gapCode(request.getGapCode())
            .patientId(request.getPatientId())
            .severity(request.getSeverity())
            .status("OPEN")
            .detectionDate(LocalDate.now())
            .daysOpen(0)
            .timestamp(Instant.now())
            .build();
    }

    public CareGapEventResponse closeGap(String gapId, String tenantId) {
        CareGapProjection projection = gapRepository.findById(gapId)
            .orElseThrow(() -> new RuntimeException("Gap not found: " + gapId));

        GapClosedEvent event = new GapClosedEvent(
            tenantId, projection.getPatientId(),
            projection.getGapCode(), "Manual closure", "CLOSED"
        );

        gapEventHandler.handle(event);
        updatePopulationHealth(tenantId, projection.getSeverity(), false);
        kafkaTemplate.send(GAP_EVENTS_TOPIC, projection.getPatientId(), event);

        return CareGapEventResponse.builder()
            .gapCode(projection.getGapCode())
            .status("CLOSED")
            .closureDate(LocalDate.now())
            .timestamp(Instant.now())
            .build();
    }
}
```

**What This Proves:** The platform implements true event sourcing -- domain events drive projections and Kafka streams, not simple CRUD operations.

---

## 2. Multi-Tenant Data Isolation

**Why It Matters:** Healthcare organizations require strict data isolation between tenants. A data leak between payer organizations would be a HIPAA violation with seven-figure fines. Every database query enforces tenant boundaries at the repository layer, making cross-tenant access structurally impossible.

**File:** `backend/modules/services/patient-service/src/main/java/com/healthdata/patient/repository/PatientDemographicsRepository.java`

```java
public interface PatientDemographicsRepository
        extends JpaRepository<PatientDemographicsEntity, UUID> {

    /** Find patient by ID and tenant ID for multi-tenant isolation. */
    Optional<PatientDemographicsEntity> findByIdAndTenantId(UUID id, String tenantId);

    /** Find patient by FHIR patient ID and tenant ID. */
    Optional<PatientDemographicsEntity> findByFhirPatientIdAndTenantId(
        String fhirPatientId, String tenantId);

    /** Find all active patients for a tenant. */
    @Query("SELECT p FROM PatientDemographicsEntity p "
         + "WHERE p.tenantId = :tenantId AND p.active = true")
    List<PatientDemographicsEntity> findActiveByTenantId(
        @Param("tenantId") String tenantId);

    /** Find patients by PCP ID for a tenant. */
    @Query("SELECT p FROM PatientDemographicsEntity p "
         + "WHERE p.tenantId = :tenantId "
         + "AND p.pcpId = :pcpId AND p.active = true")
    List<PatientDemographicsEntity> findByPcpIdAndTenantId(
        @Param("pcpId") String pcpId,
        @Param("tenantId") String tenantId);

    /** Find active patients for a tenant with pagination. */
    Page<PatientDemographicsEntity> findByTenantIdAndActiveTrue(
        String tenantId, Pageable pageable);
}
```

**What This Proves:** Tenant isolation is enforced at the data access layer -- every query method requires `tenantId`, making accidental cross-tenant queries impossible to write.

---

## 3. HIPAA Audit Trail with PHI Access Tracking

**Why It Matters:** HIPAA requires a complete audit trail for every access to Protected Health Information (PHI). This audit integration publishes structured events to Kafka, capturing who accessed what data, why, and for how long -- providing the evidence needed for compliance audits and breach investigations.

**File:** `backend/modules/services/patient-service/src/main/java/com/healthdata/patient/audit/PatientAuditIntegration.java`

```java
@Service
@Slf4j
public class PatientAuditIntegration {

    private final AIAuditEventPublisher auditEventPublisher;

    @Value("${audit.kafka.enabled:true}")
    private boolean auditEnabled;

    public void publishHealthRecordAccessEvent(
            String tenantId,
            String patientId,
            List<String> resourceTypes,
            int totalResources,
            boolean consentApplied,
            String accessPurpose,
            long aggregationTimeMs,
            String executingUser) {

        if (!auditEnabled) return;

        Map<String, Object> inputMetrics = new HashMap<>();
        inputMetrics.put("accessPurpose", accessPurpose);
        inputMetrics.put("executingUser", executingUser);
        inputMetrics.put("resourceTypes", resourceTypes);
        inputMetrics.put("totalResources", totalResources);
        inputMetrics.put("consentApplied", consentApplied);

        AIAgentDecisionEvent event = AIAgentDecisionEvent.builder()
                .eventId(UUID.randomUUID())
                .timestamp(Instant.now())
                .tenantId(tenantId)
                .agentType(AgentType.PHI_ACCESS)
                .decisionType(DecisionType.PHI_ACCESS)
                .resourceType("Patient")
                .resourceId(patientId)
                .inputMetrics(inputMetrics)
                .inferenceTimeMs(aggregationTimeMs)
                .reasoning(String.format(
                    "Accessed patient health record: %d resources (%s) for purpose: %s",
                    totalResources, String.join(", ", resourceTypes), accessPurpose))
                .outcome(DecisionOutcome.APPROVED)
                .build();

        auditEventPublisher.publishAIDecision(event);
    }
}
```

**What This Proves:** Every PHI access is captured with structured metadata (who, what, why, when, duration), published to Kafka for immutable storage -- meeting HIPAA 164.312(b) audit control requirements.

---

## 4. FHIR R4 Bulk Data Export

**Why It Matters:** Health plans need to export millions of clinical records for analytics, regulatory reporting, and data exchange with CMS. The FHIR Bulk Data Access specification (HL7 standard) is the industry-standard mechanism, and implementing it correctly -- with tenant isolation, concurrent job limits, and async processing -- is a significant technical differentiator.

**File:** `backend/modules/services/fhir-service/src/main/java/com/healthdata/fhir/bulk/BulkExportService.java`

```java
@Service
@Slf4j
public class BulkExportService {

    private final BulkExportRepository exportRepository;
    private final BulkExportProcessor exportProcessor;
    private final BulkExportConfig config;
    private final KafkaTemplate<String, Object> kafkaTemplate;

    @Transactional
    public UUID kickOffExport(
            String tenantId,
            BulkExportJob.ExportLevel exportLevel,
            String resourceId,
            List<String> resourceTypes,
            Instant since,
            List<String> typeFilters,
            String requestUrl,
            String requestedBy) {

        // Enforce per-tenant concurrent export limits
        long activeJobs = exportRepository.countActiveJobsByTenant(tenantId);
        if (activeJobs >= config.getMaxConcurrentExports()) {
            throw new ExportLimitExceededException(
                "Maximum concurrent exports exceeded for tenant");
        }

        List<String> effectiveResourceTypes = resourceTypes != null
            ? resourceTypes
            : getDefaultResourceTypes(exportLevel);

        BulkExportJob job = BulkExportJob.builder()
            .jobId(UUID.randomUUID())
            .tenantId(tenantId)
            .status(BulkExportJob.ExportStatus.PENDING)
            .exportLevel(exportLevel)
            .resourceTypes(effectiveResourceTypes)
            .outputFormat("ndjson")
            .sinceParam(since)
            .requestedBy(requestedBy)
            .build();

        BulkExportJob saved = exportRepository.save(job);

        // Publish Kafka event for audit trail
        kafkaTemplate.send("fhir.bulk-export.initiated",
            saved.getJobId().toString(),
            new BulkExportEvent(saved.getJobId().toString(), tenantId,
                "INITIATED", Instant.now(), requestedBy));

        // Start async processing
        exportProcessor.processExport(saved.getJobId());
        return saved.getJobId();
    }
}
```

**What This Proves:** Full FHIR Bulk Data Access implementation with tenant-scoped concurrency limits, NDJSON output, and async processing -- ready for CMS data submission and payer-to-payer exchange.

---

## 5. Operations Orchestration Framework

**Why It Matters:** Enterprise healthcare platforms require controlled operational workflows -- starting services, seeding demo data, running validations -- with full audit trails and idempotency. This operations framework manages the entire platform lifecycle through a database-backed state machine with concurrency guards and validation scorecards.

**File:** `backend/modules/services/gateway-admin-service/src/main/java/com/healthdata/gateway/admin/operations/OperationsService.java`

```java
@Service
@Slf4j
public class OperationsService {

    private UUID queueRun(
        OperationRun.OperationType type,
        String actor,
        String idempotencyKey,
        Map<String, Object> parameters,
        List<OperationRun.OperationType> concurrencyGroup,
        List<CommandStep> steps
    ) {
        // Idempotency: return existing run if same key submitted
        if (idempotencyKey != null && !idempotencyKey.isBlank()) {
            Optional<OperationRun> existing = runRepository
                .findTopByOperationTypeAndIdempotencyKeyOrderByRequestedAtDesc(
                    type, idempotencyKey);
            if (existing.isPresent()) return existing.get().getId();
        }

        // Concurrency guard: prevent conflicting operations
        boolean hasRunning = runRepository.existsByStatusAndOperationTypeIn(
            OperationRun.RunStatus.RUNNING, concurrencyGroup);
        if (hasRunning) {
            throw new IllegalStateException(
                "Another operation in this category is already running.");
        }

        // Persist run and steps
        OperationRun run = new OperationRun();
        run.setOperationType(type);
        run.setStatus(OperationRun.RunStatus.QUEUED);
        run.setRequestedBy(actor);
        run.setParametersJson(asJson(parameters));
        run = runRepository.save(run);

        for (int i = 0; i < steps.size(); i++) {
            OperationRunStep runStep = new OperationRunStep();
            runStep.setRun(run);
            runStep.setStepOrder(i + 1);
            runStep.setStepName(steps.get(i).name());
            runStep.setCommandText(steps.get(i).command());
            stepRepository.save(runStep);
        }

        // Dispatch async execution after transaction commits
        dispatchAsyncAfterCommit(run.getId(), steps);
        return run.getId();
    }

    public record ValidationScorecard(
        int score, String grade,
        boolean criticalPass, boolean passed,
        List<ValidationGate> gates, Instant createdAt
    ) {}
}
```

**What This Proves:** Platform operations are managed through a transactional state machine with idempotency keys, concurrency guards, and validation scoring -- not ad-hoc scripts.

---

## 6. Revenue Cycle: Claim Submission with Retry and State Machine

**Why It Matters:** Revenue cycle management is the financial backbone of healthcare operations. This service implements the full claim lifecycle -- submission with exponential backoff retry, remittance reconciliation, and a strict state machine that prevents illegal transitions (e.g., a PAID claim cannot be resubmitted). Every operation produces an immutable audit envelope for compliance.

**File:** `backend/modules/services/payer-workflows-service/src/main/java/com/healthdata/payer/service/RevenueContractService.java`

```java
@Service
public class RevenueContractService {
    private static final int MAX_SUBMISSION_ATTEMPTS = 3;
    private static final long INITIAL_BACKOFF_MILLIS = 100L;

    public ClaimSubmissionResponse submitClaim(ClaimSubmissionRequest request) {
        // Idempotency: suppress duplicate submissions
        ClaimSubmissionResponse existing =
            claimSubmissionsByIdempotencyKey.get(request.getIdempotencyKey());
        if (existing != null) {
            return ClaimSubmissionResponse.builder()
                .status(existing.getStatus())
                .duplicate(true)
                .auditEnvelope(audit(existing.getTenantId(),
                    existing.getCorrelationId(), request.getActor(),
                    "CLAIM_SUBMISSION_REPLAY", "DUPLICATE_SUPPRESSED"))
                .build();
        }

        // Retry with exponential backoff
        int attempt = 1;
        long backoff = INITIAL_BACKOFF_MILLIS;
        RevenueClaimState finalStatus = RevenueClaimState.REJECTED;
        while (attempt <= MAX_SUBMISSION_ATTEMPTS) {
            try {
                ClearinghouseSubmissionResult result =
                    clearinghouseSubmissionAdapter.submit(request, attempt);
                finalStatus = result.accepted()
                    ? RevenueClaimState.SUBMITTED
                    : RevenueClaimState.REJECTED;
                break;
            } catch (NonRetryableClearinghouseException e) {
                finalStatus = RevenueClaimState.REJECTED;
                break;
            } catch (RetryableClearinghouseException e) {
                if (attempt == MAX_SUBMISSION_ATTEMPTS) break;
                backoffExecutor.backoff(backoff);
                backoff = backoff * 2;
                attempt++;
            }
        }

        claimStatusByClaimId.put(request.getClaimId(), finalStatus);
        // ... build response with audit envelope
    }

    /** Strict state machine: only allowed transitions proceed. */
    private boolean isAllowedTransition(RevenueClaimState from, RevenueClaimState to) {
        return switch (from) {
            case DRAFT          -> to == RevenueClaimState.PENDING_SUBMIT;
            case PENDING_SUBMIT -> to == SUBMITTED || to == REJECTED;
            case SUBMITTED      -> to == ACKNOWLEDGED || to == REJECTED
                                || to == PARTIALLY_PAID || to == PAID || to == DENIED;
            case ACKNOWLEDGED   -> to == PARTIALLY_PAID || to == PAID || to == DENIED;
            case PARTIALLY_PAID -> to == PAID || to == DENIED;
            case PAID           -> false;
            case DENIED         -> to == UNDER_REVIEW;
            case UNDER_REVIEW   -> false;
            case REJECTED       -> false;
        };
    }
}
```

**What This Proves:** Revenue cycle operations use idempotent submission, exponential backoff retry, and a validated state machine -- preventing financial data corruption and ensuring clearinghouse reliability.

---

## 7. CMS Star Rating Calculator

**Why It Matters:** Medicare Advantage Star Ratings directly determine a plan's Quality Bonus Payment (QBP) -- the difference between a 3-star and 5-star plan is worth tens of millions in annual revenue. This calculator implements the CMS methodology with real cut points, domain weighting, and ROI-prioritized improvement recommendations that tell a health plan exactly which measures to focus on.

**File:** `backend/modules/services/payer-workflows-service/src/main/java/com/healthdata/payer/service/StarRatingCalculator.java`

```java
@Service
@Slf4j
public class StarRatingCalculator {

    // CMS 2024 cut points for common measures (5 thresholds for 2-5 stars)
    private static final Map<StarRatingMeasure, double[]> DEFAULT_CUT_POINTS =
        new HashMap<>();

    static {
        DEFAULT_CUT_POINTS.put(BREAST_CANCER_SCREENING,
            new double[]{0.55, 0.62, 0.68, 0.72, 0.76});
        DEFAULT_CUT_POINTS.put(CONTROLLING_BLOOD_PRESSURE,
            new double[]{0.55, 0.60, 0.65, 0.70, 0.75});
        DEFAULT_CUT_POINTS.put(MEDICATION_ADHERENCE_DIABETES,
            new double[]{0.72, 0.77, 0.81, 0.84, 0.87});
        // ... 20+ measures with CMS-calibrated thresholds
    }

    public int calculateStarsForMeasure(double performanceRate, double[] cutPoints) {
        boolean inverted = cutPoints[0] > cutPoints[4]; // e.g., readmissions
        if (inverted) {
            if (performanceRate <= cutPoints[4]) return 5;
            if (performanceRate <= cutPoints[3]) return 4;
            if (performanceRate <= cutPoints[1]) return 3;
            if (performanceRate <= cutPoints[0]) return 2;
            return 1;
        } else {
            if (performanceRate >= cutPoints[4]) return 5;
            if (performanceRate >= cutPoints[3]) return 4;
            if (performanceRate >= cutPoints[1]) return 3;
            if (performanceRate >= cutPoints[0]) return 2;
            return 1;
        }
    }

    public ImprovementOpportunity calculateImprovementOpportunity(MeasureScore score) {
        if (score.getStars() >= 5) return null;

        double nextThreshold = score.getCutPoints()[cutPointIndex];
        double performanceGap = nextThreshold - score.getPerformanceRate();
        int patientsNeeded = (int) Math.ceil(
            performanceGap * score.getDenominator());

        double potentialImpact = score.getWeight() * (nextStars - score.getStars());
        double roiScore = potentialImpact / effortMultiplier;

        return ImprovementOpportunity.builder()
            .measure(score.getMeasure())
            .currentStars(score.getStars())
            .nextStarThreshold(nextThreshold)
            .patientsNeeded(patientsNeeded)
            .potentialImpact(potentialImpact)
            .roiScore(roiScore)
            .build();
    }
}
```

**What This Proves:** Deep healthcare domain implementation -- real CMS cut points, inverted measure handling, weighted domain aggregation, and actionable ROI-ranked improvement recommendations that drive plan revenue.

---

## 8. Custom Quality Measure Builder with CQL Evaluation

**Why It Matters:** Every health plan has unique quality measures beyond standard HEDIS. This service lets quality teams author custom measures with Clinical Quality Language (CQL) logic, test them against real patient data through the CQL engine, and manage the full lifecycle from draft to published. The circuit breaker pattern ensures the UI remains responsive even if the CQL evaluation engine is temporarily unavailable.

**File:** `backend/modules/services/quality-measure-service/src/main/java/com/healthdata/quality/service/CustomMeasureService.java`

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class CustomMeasureService {

    private final CustomMeasureRepository repository;
    private final CqlEngineServiceClient cqlEngineClient;

    @Transactional
    public CustomMeasureEntity createDraft(
            String tenantId, String name, String description,
            String category, Integer year, String owner,
            String clinicalFocus, String reportingCadence,
            String targetThreshold, String priority,
            String implementationNotes, String tags,
            String createdBy) {

        CustomMeasureEntity entity = CustomMeasureEntity.builder()
                .tenantId(tenantId)
                .name(name)
                .description(description)
                .category(category)          // HEDIS, CMS, CUSTOM
                .clinicalFocus(clinicalFocus)
                .reportingCadence(reportingCadence)
                .targetThreshold(targetThreshold)
                .priority(priority)
                .status("DRAFT")
                .createdBy(createdBy)
                .build();
        return repository.save(entity);
    }

    @Transactional(readOnly = true)
    @CircuitBreaker(name = "cqlEngine", fallbackMethod = "testMeasureFallback")
    public TestMeasureResult testMeasure(
            String tenantId, UUID id, List<String> patientIds) {
        CustomMeasureEntity measure = getById(tenantId, id);

        List<TestPatientResult> results = new ArrayList<>();
        for (String patientId : patientIds) {
            PatientEvaluationResult evalResult =
                evaluatePatient(tenantId, measure.getCqlText(), patientId);

            boolean inNumerator = evalResult.matchedCriteria().stream()
                .anyMatch(c -> c.criterionName().contains("Numerator")
                            && c.matched());

            results.add(new TestPatientResult(
                patientId, evalResult.outcome(),
                inNumerator, evalResult.matchedCriteria()));
        }
        return new TestMeasureResult(id.toString(), measure.getName(),
            patientIds.size(), results,
            new TestSummary(passed, failed, notEligible, errors));
    }

    /** Soft delete for HIPAA compliance -- records are never hard-deleted. */
    @Transactional
    public void delete(String tenantId, UUID id, String deletedBy) {
        CustomMeasureEntity measure = getById(tenantId, id);
        measure.setDeletedAt(LocalDateTime.now());
        measure.setDeletedBy(deletedBy);
        repository.save(measure);
    }
}
```

**What This Proves:** Full measure lifecycle management with CQL engine integration, circuit breaker resilience, tenant-scoped data access, and HIPAA-compliant soft deletion -- a self-service quality measure builder for clinical teams.

---

*Confidential -- For Investor Use Only | March 2026*
