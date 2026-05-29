# Contributing to Harness

Thank you for your interest in contributing! **Harness** is a Claude Code meta-skill factory for designing agent teams and generating skills.

---

## Response Commitments

| Event | SLA |
|-------|-----|
| Pull request acknowledgement | 72 hours |
| Issue triage | 48 hours |
| Critical bug fix | 14 days |
| Security report initial response | 7 days |

---

## Getting Started

```bash
# Enable experimental Agent Teams
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

# Link the plugin locally for development
claude plugin link ./harness
```

The repository accepts:
- **Bug reports** — use the bug report issue template
- **Feature requests** — use the feature request template
- **Questions & discussions** — open a Discussion

---

## Pull Request Standards

**Branch naming:** `type/description`
Examples: `feat/expert-pool-variance-mode`, `fix/supervisor-task-leak`

**Commit messages:** Both Korean and English are accepted.

**PR checklist:**
- [ ] Summary of changes
- [ ] Motivation / problem being solved
- [ ] Scope checklist (agents, skills, orchestrator, docs)
- [ ] Test description
- [ ] SemVer impact (patch / minor / major)

---

## Versioning

This project follows [Conventional Commits](https://www.conventionalcommits.org/) and [Semantic Versioning](https://semver.org/).

| Commit prefix | Version bump |
|---------------|-------------|
| `feat!:` | Major |
| `feat:` | Minor |
| `fix:` | Patch |

Releases are published biweekly unless no changes warrant publication.

---

## Code of Conduct

All contributors must follow the [Contributor Covenant v1.4](https://www.contributor-covenant.org/version/1/4/code-of-conduct/).

- Treat all participants with respect
- Critique ideas, not individuals

---

## Security

Report security issues privately to `robin.hwang@kakaocorp.com` with the subject line prefix `[harness-security]`.
