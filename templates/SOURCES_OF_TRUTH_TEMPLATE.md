# Sources of Truth Index Template

Use this template to establish which file is authoritative for each concept in your project. When a document and a source of truth disagree, the source of truth wins.

---

## Instructions

Copy the template below into your project as `SOURCES_OF_TRUTH.md`. Fill in your project's actual canonical sources. Update this file whenever you add a new authoritative source.

---

## Template

```markdown
# Sources of Truth

**Purpose:** When two files disagree, this index tells you which one is right.

**Rule:** If a fact appears in multiple files, the source of truth listed here is canonical. All other files containing that fact are mirrors that may be stale.

## Code & Architecture

| Concept | Source of Truth | NOT authoritative |
|---------|---------------|-------------------|
| Current test count | `pytest` output (run it) | Any document citing a test count |
| API contracts | Source code (schemas/, interfaces/) | Design documents describing the API |
| Feature availability | Source code + tests | Roadmap documents, planning files |
| Dependencies | `[package.json / requirements.txt / Cargo.toml]` | Prose documents listing dependencies |

## Project State

| Concept | Source of Truth | NOT authoritative |
|---------|---------------|-------------------|
| Current branch & status | `git status` output | State digest documents |
| What's deployed | `[deployment system / CI dashboard]` | Release notes (may be stale) |
| Test health | `pytest` / `npm test` output | Documents claiming "all tests pass" |
| Build status | `[build command]` output | Documents claiming "builds clean" |

## Process & Governance

| Concept | Source of Truth | NOT authoritative |
|---------|---------------|-------------------|
| Coding conventions | `[AGENT_DEVELOPMENT_GUIDELINES.md]` | Individual file comments |
| Architectural constraints | `[COHERENCE_DOCTRINE.md]` | Design discussion documents |
| Work order status | `[CHECKLIST.md or tracking file]` | Individual WO files (may not update status) |
| Agent onboarding | `[AGENT_ONBOARDING_CHECKLIST.md]` | README orientation sections |

## Domain-Specific

| Concept | Source of Truth | NOT authoritative |
|---------|---------------|-------------------|
| [Your domain concept 1] | `[file]` | [what to ignore] |
| [Your domain concept 2] | `[file]` | [what to ignore] |

## Resolution Protocol

When you find a contradiction:

1. Check this index. The source of truth wins.
2. If the source of truth is a command (e.g., `pytest`), run it. Don't trust cached output.
3. Update the stale file to match the source of truth.
4. If the stale file is in a different agent's write set, flag it — don't update it yourself.
5. If neither file is listed here, escalate to the coordinator. One of them needs to become the source of truth.

## Maintenance

- Update this file when you add a new authoritative source
- Review quarterly (or after major milestones) to remove stale entries
- This file itself is authoritative — it's the source of truth about sources of truth
```

---

## Customization Notes

- **Machine-verifiable sources beat documents.** Prefer `pytest` output over a document claiming test counts. Prefer `git status` over a state digest. Documents drift; commands don't.
- **The "NOT authoritative" column is as important as the source column.** It tells agents what to distrust. Without it, agents treat every file as equally authoritative.
- **Keep it short.** If this index exceeds 30 rows, you have too many sources of truth or too fine-grained a categorization. Consolidate.
- **Domain-specific section is project-dependent.** A game engine might list "SRD rules interpretation" → specific research file. A web app might list "auth flow" → specific module. Fill in what matters for your project.