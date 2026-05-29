# Changelog

All notable changes to this project will be documented in this file.

This project adheres to [Semantic Versioning](https://semver.org/) and [Conventional Commits](https://www.conventionalcommits.org/).

---

## [v1.2.1] — 2026-04-18

**Focus: Version consistency and community engagement**

### Fixed
- Resolved version misalignment across configuration files:
  - README badges showed `v1.0.1`
  - `marketplace.json` showed `1.1.0`
  - `plugin.json` showed `1.2.0`
  - All files unified to `v1.2.0`

### Added
- "Harness factory" positioning to differentiate from single-agent frameworks
- `CONTRIBUTING.md` with community SLA (72 h PR response, 48 h issue triage)
- `docs/` directory for long-form documentation
- Expanded plugin keywords from 5 to 17 entries

---

## [v1.2.0] — 2026-04-08

### Changed
- Simplified `CLAUDE.md` registration policy: eliminated full-context embedding in favour of pointer-based management, removing duplication

---

## [v1.1.0] — 2026-04-05

### Added
- **Phase 0** — Status audit (detect existing agents/skills before building)
- **Hybrid execution modes** — switch between Agent Teams and sub-agents per phase
- **Phase 7** — Harness evolution mechanism with feedback-driven automation

---

## [v1.0.0] — 2026-03-27

### Added
- Foundation release
- 6-phase workflow (Domain Analysis → Team Design → Agent Definitions → Skills → Orchestration → Validation)
- 6 agent architecture patterns (Pipeline, Fan-out/Fan-in, Expert Pool, Producer-Reviewer, Supervisor, Hierarchical Delegation)
- Practical team composition examples
