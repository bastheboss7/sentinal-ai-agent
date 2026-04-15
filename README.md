# Sentinel V1 External Bundle

## Executive Summary

Sentinel V1 is an enterprise QA decision package that maps manual test intent to existing automation coverage using a governed, auditable workflow.

This repository is the external distribution bundle and should be treated as the release source for Sentinel documentation, contracts, runbooks, and coverage decision evidence.

## Quick Start (5 Steps)

1. Install VS Code, GitHub Copilot, GitHub Copilot Chat, and BrowserStack VS Code extension (MCP enabled).
2. Clone this Sentinel bundle repository and your automation repository.
3. Open both repositories in one VS Code multi-root workspace.
4. Confirm agent files exist:
	- `.github/AGENTS.md`
	- `.github/agents/sentinel-intake.agent.md`
	- `.github/agents/sentinel-intake-schema.md`
5. Run first smoke command in Copilot Chat:

```text
@sentinel-intake-agent Map BrowserStack case [QA-Case-XXX] from [ProjectKey] to existing coverage
```

Expected first-run outcome:
- case shortlist is returned
- explicit case confirmation is requested
- mapping output references nearest suite/target from the automation repository

## Intended Audience

- QA and automation leads
- Domain and service engineers
- Product and business stakeholders
- Consumer teams onboarding Sentinel into their own automation repositories

## Bundle Purpose

Sentinel provides a controlled process to:

1. Discover manual cases from external test management
2. Select and confirm one intake case
3. Map to nearest existing automation path
4. Classify disposition using deterministic taxonomy
5. Produce approval-ready rationale and coverage draft

Decision taxonomy:

- Net-new automation
- Extend existing coverage
- Traceability-only gap

## Repository Role and Boundaries

This repository is:

- External delivery package for Sentinel V1
- Contract and governance source for intake-to-decision workflow
- Documentation and runbook distribution point

This repository is not:

- A test execution repository
- A replacement for consumer automation codebases
- A source for internal helper artifacts outside declared bundle scope

## Architecture and Contracts

Use the following entrypoints in order:

1. .github/AGENTS.md
2. copilot-instructions.md
3. .sentinel/BUNDLE_DELIVERY_README.md
4. .sentinel/architecture.md
5. .sentinel/runbooks/using-sentinel-agent.md

Core contracts:

- .github/agents/sentinel-intake.agent.md
- .github/agents/sentinel-intake-schema.md
- .sentinel/templates/intake-response-template.md

Evidence store:

- .sentinel/output/coverage.md

## Enterprise Operating Model

### Workspace model (mandatory)

For production-quality mapping outcomes, open both repositories in one VS Code multi-root workspace:

- Root 1: this Sentinel bundle repository
- Root 2: consumer automation repository containing test suites and implementation paths

If only this repository is open, intake discovery can still work but nearest-suite and nearest-target mapping quality may degrade.

### Transport model

- Preferred: MCP path for external test case discovery
- V1 onboarding: MCP installation path only
- Fallback transport references may appear as architectural context and are outside V1 installation baseline

## Security and Compliance Controls

1. No secret values in committed artifacts
2. No unreviewed mutation of coverage decisions
3. Evidence-first outputs with explicit rationale chains
4. Human approval gate required before implementation progression
5. Append-only and idempotent decision-store behavior

## Governance and Approval Model

Required reviewers:

- QA and automation lead
- Domain and service engineer
- Product and business stakeholder

No-go conditions:

- Missing required reviewer role
- Ambiguous mapping without explicit pending disposition
- Contract validation failure for coverage mutation draft

## Installation and First Run

For full installation and first-run steps, use:

- .sentinel/BUNDLE_DELIVERY_README.md
- .sentinel/runbooks/using-sentinel-agent.md

These documents include:

- Tooling prerequisites
- MCP onboarding flow
- Multi-root workspace requirement
- First-run smoke command and troubleshooting signals

## Release and Versioning Guidance

Recommended release practice for enterprise consumers:

1. Tag bundle releases with immutable semantic versions
2. Publish release notes with contract-impact summary
3. Preserve backward compatibility for decision taxonomy and required output fields
4. Treat schema and runbook changes as controlled change-management events

## Support and Escalation

Support path:

- L1: consumer team champion
- L2: Sentinel maintainers
- L3: domain and service engineer

Escalate when:

- Intake mapping repeatedly returns ambiguous outcomes
- Contract behavior differs from schema definitions
- Consumer environment cannot satisfy MCP onboarding requirements

## Known Constraints

Known constraints and external handoff assumptions are maintained in:

- .sentinel/BUNDLE_DELIVERY_README.md

## Out-of-Scope Artifacts

Out-of-bundle and internal-only artifacts are defined in:

- .sentinel/BUNDLE_DELIVERY_README.md

Consumer teams should not depend on excluded internal helper content.

## Quick Validation Checklist

Before adoption, verify:

1. Required agent and schema files are present
2. Installation prerequisites are satisfied
3. Multi-root workspace includes both Sentinel and automation repositories
4. First-run invocation returns shortlist and explicit selection flow
5. Output package contains decision type, rationale, mapping target, and approval state

