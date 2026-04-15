# Sentinel Agent Invocation Runbook

This guide explains how to invoke the Sentinel Intake Agent, interpret its output, and complete the approval workflow.

---

## Prerequisites

| Requirement | Detail |
|---|---|
| VS Code | With GitHub Copilot Chat extension |
| Workspace | Same VS Code workspace (multi-root) with both repositories open: Sentinel bundle repo and automation repo |
| BrowserStack access | BrowserStack VS Code extension signed in with MCP enabled and project access available |
| Agent file | `.github/agents/sentinel-intake.agent.md` must exist in workspace root |

Mapping quality depends on both roots being open in one workspace. If only the Sentinel bundle repo is open, shortlist retrieval may work but nearest-suite and nearest-target evidence can be incomplete.

## Quick start

### 1. Open Copilot Chat and invoke the agent

In VS Code Copilot Chat, type:

```
@sentinel-intake-agent Map BrowserStack case [QA-Case-XXX] from [ProjectKey] to existing coverage
```

Or use the full intake format:

```
@sentinel-intake-agent
Project: [ProjectKey]
Root path: [ServiceRootPath] > POST > /<endpoint>
Case ID: [QA-Case-001]
Scenario: [Business scenario description]
Expected outcome: [Expected business/API outcome]
```

### 2. Review the discovery shortlist

The agent will:
1. Load workspace context from `<consumer-repo>/docs/product.md` and `<consumer-repo>/docs/structure.md`
2. Query BrowserStack for manual cases under the specified root path
3. Present a shortlist of `not_automated` cases for your confirmation

**Important:** The agent will not proceed without explicit case confirmation.

### 3. Receive the decision package

After confirming a case, the agent produces:

| Output field | Description |
|---|---|
| `decision.type` | `Extend existing coverage`, `Net-new automation`, or `Traceability-only gap` |
| `decision.rationale` | Evidence-based explanation citing existing tests |
| `mapping.nearestSuite` | Closest existing test suite (e.g., `[EndpointSpecificTests]`) |
| `mapping.nearestTarget` | Specific test method or class path |
| `risks` | Identified risks for the recommendation |
| `approval.status` | `Pending` — requires human review |
| `coverage.DraftRow` | Proposed row for `.sentinel/output/coverage.md` |

### 4. Approve or reject the proposal

Review the decision package with your team:

**Required reviewers:**
- QA / Automation lead — validates technical fit
- Domain / service engineer — confirms endpoint behavior
- Product / business stakeholder — verifies priority alignment

After review, tell the agent:

```
@sentinel-intake-agent Approve the proposal for [QA-Case-001], approver: qa-lead
```

Or reject:

```
@sentinel-intake-agent Reject the proposal for [QA-Case-001] — rationale conflicts with current API contract
```

### 5. Coverage file update

On approval, the agent:
1. Validates the proposed row against markdown schema
2. Checks idempotency (no duplicate case IDs)
3. Appends the row to `.sentinel/output/coverage.md`
4. Returns an audit-stamped confirmation

### 6. POC Completion Gate (Sentinel sign-off)

Use this gate to formally close the Sentinel POC stage.

| Gate item | Requirement | Evidence source |
|---|---|---|
| POC case validated | At least one approved intake case is mapped and validated in environment | `.sentinel/output/coverage.md` |
| Decision recorded | Phase 1 decision type and rationale are present | `.sentinel/output/coverage.md` |
| Governance reviewers named | QA lead, domain engineer, product stakeholder identified | This runbook + approval record |
| Approval state | Gate marked `Approved` with approver and date | PR comment / release note |

**POC completion milestone label:**

`Sentinel POC Complete - External Delivery Ready (V1 Git Package)`

**Sign-off template:**

```text
Sentinel POC Completion Gate
Status: Approved
Date: YYYY-MM-DD
Case(s): [QA-Case-002]
Approvers:
- QA/Automation lead: <name>
- Domain/service engineer: <name>
- Product/business stakeholder: <name>
Release owner: <name>
```

### 7. External Team Handoff Pack

After POC sign-off, package Sentinel as a decoupled delivery bundle for external teams.

Bundle composition and external delivery boundaries are owned by `.sentinel/BUNDLE_DELIVERY_README.md`.
Use this runbook for operator workflow, invocation steps, and troubleshooting only.

---

## Governance decision criteria

The agent applies deterministic scoring to classify each case:

| Score range | Condition | Decision |
|---|---|---|
| ≥ 70 | — | **Extend existing coverage** |
| 50–69 | + traceability match | **Traceability-only gap** |
| < 50 | — | **Net-new automation** |
| Tie at top | Ambiguous candidates | **Pending** (manual review required) |

Scoring components:
- **Endpoint match** (0–50): exact = 50, contains = 35, shared token = 20
- **Keyword overlap** (0–20): 10 per matching keyword, capped at 20
- **Traceability link** (0–30): 30 if case ID appears in `@Link` annotation

---

## Input contract

The full input schema is defined in `.github/agents/sentinel-intake-schema.md`.

| Field | Type | Required | Default |
|---|---|---|---|
| `source.projectKey` | string | Yes | `[ProjectKey]` |
| `source.rootPath` | string | Yes | `[ServiceRootPath]` |
| `selection.caseId` | string | Yes | — |
| `selection.automationStatus` | string | Yes | `not_automated` |
| `scenario.description` | string | Yes | — |
| `scenario.expectedOutcome` | string | Yes | — |
| `filters.endpointFamily` | string | No | — |
| `filters.priorityBand` | string | No | — |
| `filters.keyword` | string | No | — |

---

## Output contract

The response follows `.sentinel/templates/intake-response-template.md` and includes:

| Section | Key fields |
|---|---|
| Intake summary | project, root path, case ID, automation status |
| Coverage mapping | nearest suite, nearest target, evidence references |
| Decision | type, rationale |
| Risks & assumptions | identified risks, stated assumptions |
| Approval gate | status, required reviewers |
| coverage.md draft | formatted row for append |

---

## Coverage generation (workspace scan mode)

For bulk workspace scanning:

```
@sentinel-intake-agent generate-coverage-from-workspace
```

This will:
1. Scan `src/test/java/.../tests/` for `*Tests.java` files
2. Extract `@Link`, `@DisplayName`, and method signatures
3. Normalize into `CoverageCandidate` objects
4. Map and score against current coverage
5. Write validated rows to `.sentinel/output/coverage.md`

---

## Pipeline components

The agent internally uses these components (all in `src/test/java/.../integrations/`):

| Component | Role |
|---|---|
| `[ExternalTestManagementClient]` | HTTP transport for BrowserStack TM API v2 |
| `BrowserStackIntakeStage` | Stage 1 orchestrator — fetches manual candidates |
| `CoverageIntake` | Normalized intake model bridging BrowserStack to scoring |
| `CoverageMapper` | Deterministic match scoring engine |
| `WorkspaceCoverageScanner` | Workspace file scanner for test candidates |
| `GovernanceGate` | Policy enforcement — classifies disposition |
| `MutationProposal` | Immutable mutation request with audit metadata |
| `CoverageMutationGate` | Validation-gated approval controller for coverage writes |

---

## Guardrails

- The agent **never** auto-picks a case without user confirmation
- The agent **prefers extending** existing tests over creating parallel coverage
- The agent **blocks PENDING** decisions from mutating coverage.md
- The agent **enforces idempotency** — duplicate case IDs are rejected
- The agent **requires a named approver** for every mutation
- All mutations are **append-only** — no deletions from coverage.md

---

## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Agent not found in chat | Agent file missing or renamed | Verify `.github/agents/sentinel-intake.agent.md` exists |
| 401 from BrowserStack | Invalid or expired token | Check `BROWSERSTACK_ACCESS_KEY` env variable or `browserstack.properties` |
| "Case already exists" rejection | Idempotency guard fired | Case was already processed — check `.sentinel/output/coverage.md` |
| PENDING decision blocking | Ambiguous scoring tie | Manually review candidates and re-invoke with narrowing filters |
| Empty shortlist | No `not_automated` cases in folder | Verify BrowserStack folder ID and automation status |

---

## Example walkthrough

**Scenario:** Map `[QA-Case-001]` (ParcelShop delivery) to existing coverage.

1. Invoke: `@sentinel-intake-agent Map [QA-Case-001] from [ProjectKey] > [ServiceRootPath] > POST > /<endpoint>`
2. Agent discovers case, confirms with user
3. Agent scores: endpoint match (50) + keyword match (20) = **70** → **Extend existing coverage**
4. Agent recommends: extend `[EndpointSpecificTests]#[testMethodName]()`
5. QA lead approves: `@sentinel-intake-agent Approve [QA-Case-001], approver: qa-lead`
6. Agent validates schema, checks idempotency, appends row to coverage.md
7. Audit trail: proposal ID, timestamp, approver identity, decision type recorded


