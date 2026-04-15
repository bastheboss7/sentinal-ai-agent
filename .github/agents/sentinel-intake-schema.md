# Sentinel Intake Contract Schema

This schema defines the required request and response contract for the Sentinel Intake Agent.

## 1. Input contract

### Intake transport

Sentinel supports two intake transport paths for BrowserStack case discovery:

| Path | Tool / Class | When used | Handles auth | Handles pagination |
|------|-------------|-----------|:---:|:---:|
| **MCP (preferred)** | `mcp_browserstack_listTestCases` | BrowserStack VS Code extension installed and MCP enabled | Yes | Yes |
| **Java client (fallback)** | `BrowserStackClient.fetchManualCasesByFolderId()` | MCP tool unavailable; also used for CI/pipeline execution | Yes | Manual |

When the MCP tool is available, use it with `project_identifier` (default: `PR-87`) and optional `folder_id`. The MCP tool returns structured case data directly — no additional parsing or endpoint management is needed.

When the MCP tool is unavailable, the Java transport layer at `src/test/java/uk/co/evri/apiautomation/integrations/` provides equivalent functionality via the BrowserStack TM v2 REST API.

### Required fields

| Field | Type | Required | Description |
|---|---|---|---|
| `source.projectKey` | string | Yes | BrowserStack project key. Default: `PR-87`. |
| `source.rootPath` | string | Yes | Root folder path supplied by user. Example: `Level 1 > Route service`. |
| `selection.caseId` | string | Yes | Confirmed manual case identifier selected after shortlist. Example: `QA-T6504`. |
| `selection.caseTitle` | string | Yes | Confirmed manual test case title from BrowserStack. |
| `selection.automationStatus` | string | Yes | Must be `not_automated` for intake mapping flow. |
| `scenario.description` | string | Yes | Human-readable behavior under test. |
| `scenario.expectedOutcome` | string | Yes | Expected business and API outcome. |
| `filters.endpointFamily` | string | No | Optional narrowing hint for shortlist. |
| `filters.priorityBand` | string | No | Optional narrowing hint. Suggested values: `High`, `Medium`, `Low`. |
| `filters.keyword` | string | No | Optional keyword-based narrowing hint. |
| `context.businessPriority` | string | No | Suggested values: `High`, `Medium`, `Low`. |
| `context.notes` | string | No | Reviewer or domain context notes. |

### Input validation rules

1. `selection.caseId` must be non-empty and start with `QA-`.
2. `selection.automationStatus` must equal `not_automated`.
3. `scenario.description` and `scenario.expectedOutcome` must be non-empty.
4. If `source.projectKey` is omitted, default to `PR-87`.
5. If `source.rootPath` is omitted, default to `Level 1 > Route service`.
6. A shortlist step is mandatory before `selection.caseId` is finalized.
7. Selection confirmation is required before coverage mapping starts.

### Example input payload

```json
{
  "source": {
    "projectKey": "PR-87",
    "rootPath": "Level 1 > Route service"
  },
  "filters": {
    "endpointFamily": "POST /routeDeliveryCreatePreadviceAndLabel",
    "priorityBand": "Medium",
    "keyword": "ParcelShop"
  },
  "selection": {
    "caseId": "QA-T6504",
    "caseTitle": "API - POST - /routeDeliveryCreatePreadviceAndLabel - ParcelShop delivery",
    "automationStatus": "not_automated"
  },
  "scenario": {
    "description": "Validate ParcelShop routing response and label data",
    "expectedOutcome": "ParcelShop delivery method, sort levels, and label content are valid"
  },
  "context": {
    "businessPriority": "Medium",
    "notes": "Route service critical path"
  }
}
```

## 2. Output contract

### Required fields

| Field | Type | Required | Description |
|---|---|---|---|
| `intake.rootPath` | string | Yes | Root folder path used for discovery. |
| `intake.shortlistSummary` | string | Yes | Summary of discovered candidates and narrowing logic. |
| `intake.selectionConfirmed` | boolean | Yes | Must be `true` before decisioning continues. |
| `decision.type` | string | Yes | One of: `Net-new automation`, `Extend existing coverage`, `Traceability-only gap`. |
| `decision.rationale` | string | Yes | Evidence-based rationale for decision. |
| `mapping.nearestSuite` | string | Yes | Closest suite name identified. |
| `mapping.nearestTarget` | string | Yes | Closest class/test/step target path. |
| `mapping.coverageEvidence` | string[] | Yes | Evidence references used to make the decision. |
| `risks` | string[] | No | Risks and unknowns requiring follow-up. |
| `nextAction` | string | Yes | Immediate next action for implementation or traceability. |
| `approval.status` | string | Yes | `Pending`, `Approved`, or `Rejected`. |
| `approval.requiredReviewers` | string[] | Yes | Required review roles per governance. |
| `coverageDraft` | object | Yes | Draft row content proposed for `.sentinel/output/coverage.md` review before mutation. |

### Example output payload

```json
{
  "intake": {
    "rootPath": "Level 1 > Route service",
    "shortlistSummary": "11 manual candidates found under root; narrowed by endpoint + keyword to 3 candidates.",
    "selectionConfirmed": true
  },
  "decision": {
    "type": "Extend existing coverage",
    "rationale": "Case behavior overlaps current ParcelShop positive path and should extend nearest existing test to avoid duplication."
  },
  "mapping": {
    "nearestSuite": "CreatePreadviceAndLabelTests",
    "nearestTarget": "CreatePreadviceAndLabelTests#checkParcelShopIdInResponseLabelTest",
    "coverageEvidence": [
      ".sentinel/output/coverage.md",
      "CreatePreadviceAndLabelTests#checkParcelShopIdInResponseLabelTest"
    ]
  },
  "risks": [
    "Fixture drift in SIT may affect validation"
  ],
  "nextAction": "Update nearest existing test and re-run focused validation",
  "coverageDraft": {
    "areaOrEndpoint": "POST /routeDeliveryCreatePreadviceAndLabel",
    "browserStackIntake": "QA-T6504 - ParcelShop delivery",
    "phaseDecision": "Extend existing coverage",
    "decisionRationale": "Reuses nearest ParcelShop scenario without introducing duplicate path."
  },
  "approval": {
    "status": "Pending",
    "requiredReviewers": [
      "QA/Automation lead",
      "Domain/service engineer",
      "Product/business stakeholder"
    ]
  }
}
```

## 3. Decision taxonomy

The agent must classify each intake into exactly one decision class:

1. `Net-new automation`
2. `Extend existing coverage`
3. `Traceability-only gap`

Any ambiguous case must remain `Pending` and be escalated for review.

## 4. Coverage generation contract (workspace scan -> direct write)

This contract defines how Sentinel reviews workspace tests and generates `.sentinel/output/coverage.md` in direct-write mode.

### Coverage generation input

| Field | Type | Required | Description |
|---|---|---|---|
| `command` | string | Yes | Must be `generate-coverage-from-workspace`. |
| `scan.root` | string | Yes | Root path to scan for test suites. Default: `src/test/java/uk/co/evri/apiautomation`. |
| `scan.includePatterns` | string[] | No | Optional include globs for focused scans. |
| `scan.excludePatterns` | string[] | No | Optional exclude globs. |
| `scan.discoveryMode` | string | Yes | `full` or `focused`. |
| `write.mode` | string | Yes | Must be `direct`. |
| `write.target` | string | Yes | Must be `.sentinel/output/coverage.md`. |
| `source.projectKey` | string | Yes | BrowserStack project key. Default: `PR-87`. |
| `source.rootPath` | string | Yes | BrowserStack root path. Default: `Level 1 > Route service`. |

### Coverage generation discovery output

| Field | Type | Required | Description |
|---|---|---|---|
| `generation.summary.generatedAt` | string | Yes | ISO-8601 timestamp for scan run. |
| `generation.summary.classesFound` | number | Yes | Number of test classes discovered. |
| `generation.summary.methodsFound` | number | Yes | Number of test methods discovered. |
| `generation.summary.linkedCasesFound` | number | Yes | Number of discovered `@Link` IDs. |
| `generation.suites` | object[] | Yes | Normalized suite metadata list. |
| `generation.suites[].className` | string | Yes | Suite class name. |
| `generation.suites[].classPath` | string | Yes | Workspace-relative class path. |
| `generation.suites[].endpointOrDomain` | string | Yes | Derived endpoint or domain area. |
| `generation.suites[].methods` | object[] | Yes | Method metadata for mapping. |
| `generation.suites[].methods[].name` | string | Yes | Method name. |
| `generation.suites[].methods[].linkIds` | string[] | No | Found traceability IDs from annotations. |
| `generation.suites[].methods[].keywords` | string[] | No | Derived behavior keywords. |
| `generation.warnings` | string[] | No | Discovery warnings (missing link tags, ambiguous endpoint, etc.). |

### Coverage generation mapping output

| Field | Type | Required | Description |
|---|---|---|---|
| `generation.decisions` | object[] | Yes | One decision per mapped intake area. |
| `generation.decisions[].decisionType` | string | Yes | `Net-new automation`, `Extend existing coverage`, `Traceability-only gap`, or `Pending`. |
| `generation.decisions[].decisionScore` | number | Yes | Deterministic score from 0 to 100. |
| `generation.decisions[].nearestSuite` | string | Yes | Closest suite selected. |
| `generation.decisions[].nearestTarget` | string | Yes | Closest class/method target. |
| `generation.decisions[].evidenceChain` | string[] | Yes | Verifiable evidence references used in mapping. |
| `generation.decisions[].coverageDraft` | object | Yes | Row payload for coverage.md mutation. |

### Direct-write mutation rules

1. Preserve existing coverage document headings and both table schemas.
2. Apply append-only behavior for new decisions (no row deletion).
3. Use idempotency key `browserStackIntake + nearestTarget`.
4. If idempotency key already exists, update only decision rationale/evidence text for that row.
5. Reject write if required table columns are missing or malformed.
6. Fail-safe behavior: if validation fails, do not write partial changes.

### Deterministic scoring policy

Use fixed weighted scoring to classify disposition:

1. Endpoint/domain match: 0-50
2. Scenario keyword overlap: 0-20
3. Existing link/traceability overlap: 0-30

Decision thresholds:

- `>= 70`: `Extend existing coverage`
- `50-69` with existing link overlap: `Traceability-only gap`
- `< 50`: `Net-new automation`
- Ambiguous tie across top candidates: `Pending`

### Coverage generation output summary

| Field | Type | Required | Description |
|---|---|---|---|
| `writeResult.status` | string | Yes | `Success` or `Failed`. |
| `writeResult.rowsAdded` | number | Yes | New rows appended. |
| `writeResult.rowsUpdated` | number | Yes | Existing rows updated via idempotency key. |
| `writeResult.rowsSkipped` | number | Yes | Rows skipped due to validation or ambiguity. |
| `writeResult.diffSummary` | string | Yes | Human-readable write summary. |

## 5. POC sign-off and external handoff contract

This section defines additional metadata used when Sentinel is released to external teams after POC closure.

### Handoff metadata fields

| Field | Type | Required | Description |
|---|---|---|---|
| `release.phase` | string | Yes (for handoff) | Phase label for the release event. Example: `POC Complete`. |
| `release.status` | string | Yes (for handoff) | One of: `Draft`, `Approved`, `Published`. |
| `release.milestoneLabel` | string | Yes (for handoff) | Human-readable milestone label used in approvals and release notes. |
| `handoff.packageVersion` | string | Yes (for handoff) | Version identifier for the delivery package. Example: `v1.0.0`. |
| `handoff.ownedBy` | string | Yes (for handoff) | Team or owner accountable for package maintenance. |
| `handoff.supportModel` | string | Yes (for handoff) | Support path. Suggested value: `L1 consumer champion -> L2 sentinel maintainers -> L3 domain engineer`. |
| `handoff.consumers` | string[] | Yes (for handoff) | External teams/repositories adopting the package. |
| `handoff.approvers` | string[] | Yes (for handoff) | Names/roles for QA lead, domain engineer, product stakeholder. |
| `handoff.approvalDate` | string | Yes (for handoff) | ISO date the sign-off was approved. |

### Handoff validation rules

1. Handoff metadata is required when `release.phase = POC Complete`.
2. `release.status` must be `Approved` before external publication.
3. `handoff.approvers` must include all three reviewer roles:
   - QA/Automation lead
   - Domain/service engineer
   - Product/business stakeholder
4. `handoff.packageVersion` must be immutable for a published artifact.

### Example handoff metadata payload

```json
{
  "release": {
    "phase": "POC Complete",
    "status": "Approved",
    "milestoneLabel": "Sentinel POC Complete - External Delivery Ready (V1 Git Package)"
  },
  "handoff": {
    "packageVersion": "v1.0.0",
    "ownedBy": "Sentinel Maintainers",
    "supportModel": "L1 consumer champion -> L2 sentinel maintainers -> L3 domain engineer",
    "consumers": [
      "external-team-a",
      "external-team-b"
    ],
    "approvers": [
      "QA/Automation lead",
      "Domain/service engineer",
      "Product/business stakeholder"
    ],
    "approvalDate": "2026-04-15"
  }
}
```
