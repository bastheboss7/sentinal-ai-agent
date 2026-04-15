# Sentinel Architecture

## 1. Purpose

Sentinel is an enterprise QA decision layer for `[service-tests]` that converts BrowserStack manual intent into deterministic, governed automation decisions.

This architecture defines:
- system boundaries,
- component responsibilities,
- data contracts and mutation behavior,
- governance and control points,
- non-functional expectations for enterprise operation.

## 2. Scope And Non-Goals

### In scope
- BrowserStack intake discovery and case selection
- Coverage mapping against existing workspace tests
- Governed decision classification (`Net-new automation`, `Extend existing coverage`, `Traceability-only gap`)
- Controlled mutation of `.sentinel/output/coverage.md` using validation-gated direct write

### Out of scope
- Replacing the existing Java test framework
- Ungoverned or silent test-code generation
- Hardcoded secrets, endpoints, or environment assumptions
- Non-deterministic mutation of coverage artifacts

## 3. System Context

### External systems
- BrowserStack Test Management (`[ProjectKey]`)
- Copilot custom-agent runtime

### Internal systems
- Java test framework (`tests -> steps -> models -> utils -> database`)
- Sentinel governance and operations docs
- Sentinel coverage store: `.sentinel/output/coverage.md`

### Primary actors
- QA/Automation engineers
- Domain/service engineers
- Product/business stakeholders

## 4. Architecture Principles

1. Evidence-first decisions
2. Nearest-existing-path-first mapping
3. Deterministic and auditable outputs
4. Smallest-safe-change implementation
5. Framework fidelity to existing repository design

## 5. Logical Components

## 5.1 Intake Orchestrator
Responsibilities:
- accept root path and optional filters,
- shortlist manual cases,
- require explicit case confirmation before mapping.

Primary artifacts:
- `.github/agents/sentinel-intake.agent.md`
- `.github/agents/sentinel-intake-schema.md`

## 5.2 BrowserStack Transport Layer
Responsibilities:
- query BrowserStack project data,
- filter to `not_automated` cases,
- normalize intake metadata.

Transport strategy (hybrid):
- **MCP (preferred):** `mcp_browserstack_listTestCases` via BrowserStack VS Code extension. Handles authentication, pagination, and endpoint routing automatically. Used when agent is invoked in Copilot Chat.
- **Java client (fallback):** `[ExternalTestManagementClient]` at `src/test/java/[company]/[service]/automation/integrations/`. Uses BrowserStack TM v2 REST API (`GET /api/v2/projects/{projectKey}/test-cases`). Configured via `src/test/resources/browserstack.properties`. Available for CI/pipeline execution and when MCP is unavailable.

Primary implementation location:
- `src/test/java/[company]/[service]/automation/integrations/`

## 5.3 Coverage Mapper
Responsibilities:
- scan workspace test assets,
- infer nearest suite and target,
- produce deterministic score and mapping evidence chain.

## 5.4 Governance Engine
Responsibilities:
- apply governance criteria,
- classify disposition,
- produce rationale and risk posture.

Governance source of truth:
- `.github/agents/sentinel-intake.agent.md` (Governance Criteria section)

## 5.5 Coverage Mutation Gate
Responsibilities:
- validate contract and markdown table shape,
- enforce append-only and idempotent row behavior,
- block malformed or non-deterministic writes.

Write target:
- `.sentinel/output/coverage.md`

## 5a. Tech Stack

| Technology | Version / Reference | Role |
|---|---|---|
| Java | 21 | Core language for test framework and transport layer |
| Maven | 3.x (wrapper) | Build, dependency management, lifecycle |
| JUnit 5 | Current | Test runner; `BaseTest`/`BaseSteps` inheritance required |
| Rest Assured | Current | HTTP execution and API assertion |
| Jackson XML | Current | XML deserialization; `[XmlValidationUtils].validateXmlByXsd` for contract validation |
| Allure | Current | Test execution reporting |
| Checkstyle | Current | Code quality gate; `checkstyle.xml` enforced in CI |
| BrowserStack Test Management | [ProjectKey] | External intake source for manual test cases |
| BrowserStack MCP tool | VS Code extension | Preferred intake transport — `mcp_browserstack_listTestCases` |
| Copilot custom agent runtime | VS Code | Agent invocation via `.github/agents/*.agent.md` |

## 5b. Implementation Layers

| Layer | Path | Responsibility |
|---|---|---|
| Tests | `src/test/java/[company]/[service]/automation/tests/` | Scenario execution; JUnit 5 entry points |
| Steps | `src/test/java/[company]/[service]/automation/steps/` | Reusable step definitions via `BaseSteps` |
| Models | `src/test/java/[company]/[service]/automation/models/` | Request/response POJOs and XML schema objects |
| Utils | `src/test/java/[company]/[service]/automation/utils/` | Shared helpers including `[XmlValidationUtils]` |
| Database | `src/test/java/[company]/[service]/automation/database/` | DB setup, teardown, and assertion support |
| Sentinel integrations | `src/test/java/[company]/[service]/automation/integrations/` | BrowserStack transport layer and governance pipeline (Phase 2C); fallback when MCP unavailable |
| Sentinel governance / operations | `.sentinel/` | Coverage store, planner, roadmap, templates, architecture |
| Agent definitions and contracts | `.github/agents/` | Intake agent spec and I/O schema contracts |

## 6. Runtime Lifecycle

Sentinel executes in staged order:

1. Context loading
2. Intake fetching
3. Coverage mapping
4. Governance decision
5. Recommendation package
6. Coverage generation and direct write (when requested)

Execution rule:
- no forward stage execution without completion/validation of the current stage.

## 7. Data Contracts

Contract source of truth:
- `.github/agents/sentinel-intake-schema.md`

### Intake contract
- BrowserStack source fields
- case selection fields
- scenario fields
- optional business and filter context

### Decision contract
- `decision.type`, `decision.rationale`
- `mapping.nearestSuite`, `mapping.nearestTarget`, evidence references
- `approval.status`, required reviewers

### Coverage generation contract
- `command=generate-coverage-from-workspace`
- scan scope and discovery mode
- direct write target and mode
- deterministic decision scores and write summary

## 8. Coverage Store Design

Coverage store file:
- `.sentinel/output/coverage.md`

Model:
- table-driven baseline + intake decision view
- append-only update posture for new decision rows

Idempotency:
- mutation key = `browserStackIntake + nearestTarget`
- duplicate key updates rationale/evidence content only

Mutation safeguards:
1. schema validation pass required
2. markdown table integrity pass required
3. no partial write on validation failure

## 9. Security And Compliance

1. Credentials externalized; no plaintext secrets in code or docs
2. Least-privilege mutation scope limited to Sentinel tracking artifacts
3. Decision outputs include evidence chain for auditability
4. Unknown evidence is flagged, never fabricated
5. Governance reviewers remain mandatory for controlled rollout

## 10. Reliability And Failure Modes

### Key failure modes
1. BrowserStack transport unavailable
2. Ambiguous mapping across multiple high-score candidates
3. Drift between coverage store and workspace reality
4. malformed coverage mutation payload

### Mitigation strategy
1. degrade gracefully to manual confirmation flow when upstream is unavailable
2. emit `Pending` disposition on ambiguous ties
3. require periodic workspace re-scan for consistency
4. block writes that fail schema/table validation

## 11. Observability And Operational Metrics

Track at minimum:
- intake-to-decision cycle time
- distribution of decision types
- rows added/updated/skipped per write
- validation failures (schema/table/idempotency)
- pending/ambiguous case count

Operational outputs should include:
- deterministic write summary (`rowsAdded`, `rowsUpdated`, `rowsSkipped`, `diffSummary`)
- evidence chain for each decision

## 12. Non-Functional Requirements

Initial enterprise targets:

1. Determinism
- identical inputs produce identical decision output and write result summary.

2. Integrity
- coverage file structure must remain valid after each write.

3. Safety
- malformed or incomplete payloads must not mutate coverage state.

4. Traceability
- each mutation maps to explicit intake, target, and rationale.

5. Maintainability
- architecture remains compatible with existing test framework layers.

## 13. Delivery Topology And Evolution

### V1 (current)
- Git package delivery of agent/config/docs in workspace

### V2 (planned)
- Maven-delivered shared Java components (transport, mapper, governance gate)

### V3 (optional)
- npm wrapper for orchestration/bootstrap

Migration principle:
- keep contracts stable and backward-compatible between delivery models.

## 14. Governance And Ownership Model

Required reviewers:
- QA/Automation lead
- Domain/service engineer
- Product/business stakeholder

Escalation path:
- ambiguous or conflicting decisions remain `Pending` and are escalated for explicit human resolution.

## 15. Architectural Decision Records

### ADR-001: Append-only coverage decision store
Decision:
- keep `.sentinel/output/coverage.md` as persistent decision evidence.

Rationale:
- maintains traceability over time and supports review audits.

### ADR-002: Nearest-existing-path-first strategy
Decision:
- prefer extension of existing coverage before net-new paths.

Rationale:
- minimizes duplication and preserves suite cohesion.

### ADR-003: Deterministic weighted disposition scoring
Decision:
- use fixed weighted scoring for decision classification.

Rationale:
- provides repeatable and reviewable governance outcomes.

### ADR-004: Validation-gated direct write
Decision:
- allow direct coverage mutation only after schema + table + idempotency checks pass.

Rationale:
- enables automation speed without silent corruption risk.

## 16. Current Phase Alignment

This architecture aligns with current delivery status:
- Phase 2A-2B complete
- Phase 2C active
- Phase 2D, 2E, 2F pending

Phase progression for internal POC development is controlled by review gates in:
- `.sentinel/operations/planner.md`
- `.sentinel/operations/roadmap.md`

These operations files are internal helper artifacts and are not part of the external delivery bundle.

## 17. Related Documents

- `.sentinel/BUNDLE_DELIVERY_README.md`
- `.sentinel/product_sentinel.md`
- `.github/agents/sentinel-intake.agent.md`
- `.github/agents/sentinel-intake-schema.md`
- `<consumer-repo>/docs/product.md`
- `<consumer-repo>/docs/structure.md`


