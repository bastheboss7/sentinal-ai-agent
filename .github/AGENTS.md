# Workspace Agents

## Sentinel Intake Agent

- File: `.github/agents/sentinel-intake.agent.md`
- Use when: mapping BrowserStack manual test cases to existing Java automation coverage and deciding whether to extend existing coverage, add net-new automation, or update traceability only.
- Scope: Route service intake under `PR-87 > Level 1 > Route service` unless explicitly expanded.

## Operating expectations

- Follow Sentinel governance in `.github/agents/sentinel-intake.agent.md` (Governance Criteria section).
- Prioritize nearest-existing-path-first decisions before any net-new coverage.
- Keep implementation aligned to existing structure: `tests -> steps -> shared models/utils`.
- Update tracking artifacts with clear rationale and evidence links.
