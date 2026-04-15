# Sentinel V1 Bundle Delivery Guide

This document defines the external-team handoff entrypoint for Sentinel V1.

This is the single source of truth for external bundle boundaries and delivery composition.

## What this bundle includes

Sentinel V1 is a governed BrowserStack-to-coverage decision package for Copilot custom agent workflows.

## Current release status

- POC stage is complete with validated intake decision evidence (`[QA-Case-001]`, `[QA-Case-002]`).
- Next active milestone is external delivery decoupling as a versioned V1 Git package.
- Phase 2C remains constrained by BrowserStack Java transport authentication (`401`) until credentials are provisioned.
- Phase 2F is functionally complete in synthetic validation; live external pilot execution is pending.

### Core files (read in this order)

1. `.github/AGENTS.md` - workspace agent index and discovery.
2. `copilot-instructions.md` - workspace instruction file that must live at consumer repo root.
3. `.sentinel/architecture.md` - boundaries, components, contracts, and controls.
4. `.sentinel/runbooks/using-sentinel-agent.md` - invocation, approval flow, troubleshooting.

### Contract files

- `.github/agents/sentinel-intake.agent.md`
- `.github/agents/sentinel-intake-schema.md`
- `.sentinel/templates/intake-response-template.md`

### Evidence file

- `.sentinel/output/coverage.md`

## Optional reference files

- `.sentinel/product_sentinel.md` (internal product deep-dive)
- `<consumer-repo>/docs/product.md` (consumer-domain business context)
- `<consumer-repo>/docs/structure.md` (consumer-domain framework conventions)

External teams can onboard without the optional reference files above.

## Adoption prerequisites

1. BrowserStack project access for target intake scope.
2. Credentials configured for your selected transport path.
3. Named local approvers for governance gate:
   - QA/Automation lead
   - Domain/service engineer
   - Product/business stakeholder
4. Agreement to append-only coverage mutation behavior.
5. Agreement to keep the decision taxonomy unchanged:
   - `Net-new automation`
   - `Extend existing coverage`
   - `Traceability-only gap`

## Installation (external teams, MCP path)

Use this installation flow for Sentinel V1 external onboarding.

### 1. Install required tooling

1. Install Visual Studio Code.
2. Install GitHub Copilot and GitHub Copilot Chat extensions.
3. Install BrowserStack VS Code extension with MCP support enabled.

### 2. Clone and open this bundle repository

1. Clone this external Sentinel bundle repository to a local workspace.
2. Open the repository root in VS Code.
3. Confirm these files exist at root-level paths:
   - `.github/AGENTS.md`
   - `.github/agents/sentinel-intake.agent.md`
   - `.github/agents/sentinel-intake-schema.md`
   - `copilot-instructions.md`
4. Add your automation repository to the same VS Code workspace (multi-root):
   - root 1: Sentinel bundle repository (this package)
   - root 2: automation repository where test suites/classes live
5. Keep both roots open during intake mapping so the agent can discover real coverage targets.

### 3. Configure BrowserStack access for MCP

1. Sign in to BrowserStack from the VS Code extension.
2. Verify Test Management access to project `[ProjectKey]` (or your approved project).
3. Keep MCP as the active intake transport for V1.

### 4. First-run validation

In Copilot Chat, run:

```text
@sentinel-intake-agent Map BrowserStack case [QA-Case-XXX] from [ProjectKey] to existing coverage
```

Expected successful signals:
1. Agent loads and responds as `@sentinel-intake-agent`.
2. Agent discovers cases under the requested root path and shows a shortlist.
3. Agent asks for explicit case confirmation before mapping.
4. Mapping output can reference suites/classes from the automation repository root.

If first-run fails:
1. `Agent not found`: verify `.github/agents/sentinel-intake.agent.md` path and workspace root.
2. BrowserStack auth error: re-check BrowserStack extension sign-in and project access.
3. Missing shortlist: verify folder/path selection and that target cases are `not_automated`.
4. Missing or weak coverage mapping: confirm Sentinel bundle repo and automation repo are both open in the same VS Code workspace.

### 5. V1 installation scope note

Installation guidance in this package is MCP-only for V1 external onboarding.
Any non-MCP transport path is outside this installation baseline.

## Known constraints

1. Java BrowserStack transport may return `401` until API access is provisioned correctly.
2. Preferred intake path is MCP (`mcp_browserstack_listTestCases`) when available.
3. Phase 2F evidence is synthetic/workspace-complete; live external pilot execution is a separate adoption activity.
4. Phase 3 items are exploratory and not part of V1 commitments.

## Support model

- L1: consumer team champion (usage and triage)
- L2: Sentinel maintainers (contract and mutation behavior)
- L3: domain/service engineer (endpoint and business-rule disputes)

## Out of bundle scope

The following are intentionally excluded from external delivery:
- `.sentinel/operations/planner.md` (internal helper for POC product development)
- `.sentinel/operations/roadmap.md` (internal helper for POC product development)

The following are archived and excluded from external delivery:
- `.sentinel/archive/pilot-package-template.md`
- `.sentinel/archive/poc-phase-1-example-[QA-Case-001].md`


