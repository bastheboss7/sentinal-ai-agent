# Sentinel V1 Bundle Delivery Guide

This document defines the external-team handoff entrypoint for Sentinel V1.

This is the single source of truth for external bundle boundaries and delivery composition.

## What this bundle includes

Sentinel V1 is a governed BrowserStack-to-coverage decision package for Copilot custom agent workflows.

## Current release status

- POC stage is complete with validated intake decision evidence (`QA-T6504`, `QA-T8666`).
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
- `docs/product.md` (current-domain business context)
- `docs/structure.md` (current-domain framework conventions)

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
- `.sentinel/archive/poc-phase-1-example-QA-T6504.md`
