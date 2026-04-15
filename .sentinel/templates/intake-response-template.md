# Sentinel Intake Response Template

Use this template for every Phase 2B+ intake decision package.

## Intake summary

- Project: `<projectKey>`
- Root path: `<rootPath>`
- Narrowing filters:
  - Endpoint family: `<endpointFamily or N/A>`
  - Priority band: `<priorityBand or N/A>`
  - Keyword: `<keyword or N/A>`
- Case ID: `<caseId>`
- Case title: `<caseTitle>`
- Automation status: `<automationStatus>`
- Priority: `<businessPriority or N/A>`

## Discovery and selection

- Manual candidates discovered under root: `<manualCandidateCount>`
- Shortlist summary: `<how the list was narrowed>`
- Selection confirmed by user: `<true | false>`

## Scenario

- Description: `<scenarioDescription>`
- Expected outcome: `<expectedOutcome>`

## Coverage mapping

- Nearest suite: `<nearestSuite>`
- Nearest target path: `<nearestTarget>`
- Evidence references:
  - `<evidenceRef1>`
  - `<evidenceRef2>`

## Decision

- Decision type: `<Net-new automation | Extend existing coverage | Traceability-only gap>`
- Rationale: `<evidence-based rationale>`

## Risks and assumptions

- Risks:
  - `<risk1>`
  - `<risk2>`
- Assumptions:
  - `<assumption1>`
  - `<assumption2>`

## Recommended next action

- `<implementation or traceability next step>`

## Approval gate

- Status: `<Pending | Approved | Rejected>`
- Required reviewers:
  - QA/Automation lead
  - Domain/service engineer
  - Product/business stakeholder
- Reviewer notes:
  - `<note1>`

## coverage.md update draft

Provide the row content proposed for `.sentinel/output/coverage.md`:

- Area / Endpoint: `<endpoint>`
- BrowserStack / Manual Intake: `<caseId and title>`
- Phase 1 Decision: `<decision type>`
- Decision rationale: `<short rationale>`

## Coverage generation result (direct write mode)

Use this block when `command=generate-coverage-from-workspace`:

- Write status: `<Success | Failed>`
- Rows added: `<number>`
- Rows updated: `<number>`
- Rows skipped: `<number>`
- Diff summary: `<short human-readable summary of changes>`
- Validation checks:
  - Schema validation: `<Pass | Fail>`
  - Markdown table integrity: `<Pass | Fail>`
  - Idempotency key handling: `<Pass | Fail>`
