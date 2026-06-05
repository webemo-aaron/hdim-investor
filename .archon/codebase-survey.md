# Codebase Survey: hdim-investor

## Executive Summary

This repository is a static, documentation-only investor materials package for the HDIM healthcare platform. It contains 13 Markdown files organized into audience-specific directories (executive, platform, technical, traction) with no source code, dependencies, or build system. The repo serves as a structured diligence package for investors and technical evaluators, hosted on dual Git remotes (Gitea and origin).

## Tech Stack

| Category | Details |
|----------|---------|
| Languages | Markdown (100%) |
| Frameworks | None |
| Build tools | None |
| Static site generator | None |
| CI/CD | None configured |
| Hosting | Git-based (Gitea + origin dual remote) |
| Package managers | None |
| Linting/formatting | None |

## Architecture

**Pattern:** Topic-based folder structure with audience routing.

```
README.md (entry point / landing page)
    |
    +-- executive/     Business stakeholders
    |     FAQ, One-Pager, Pitch Deck
    |
    +-- platform/      Technical due-diligence
    |     Architecture, Overview, Security/Compliance
    |
    +-- technical/     Engineering-depth evaluation
    |     Code Samples, Competitive Analysis, Deployment Guide
    |
    +-- traction/      Progress proof
          Dev Velocity, Milestones, Production Readiness
```

**Data flow:** There is no runtime data flow. Information flows from README (entry point) to topic directories via relative Markdown links. Readers are routed by role/interest to the appropriate section.

## Code Organization

### Directory Structure Rationale

Directories map to investor due-diligence audiences:
- `executive/` — C-suite, non-technical stakeholders needing business context
- `platform/` — Technical architects evaluating system design
- `technical/` — Engineers performing code-level diligence
- `traction/` — Anyone validating execution and progress claims

### Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Directories | lowercase, role-based | `executive/`, `traction/` |
| Files | UPPER-CASE-KEBAB.md | `PITCH-DECK.md`, `SECURITY-COMPLIANCE.md` |
| Root | Standard README.md | — |

## Development Workflow

### Build Process

None. Documents are authored directly in Markdown and committed to Git.

### Test Process

None automated. No link checking, spell checking, or markdown linting is configured.

### Deploy Process

No deployment pipeline detected. Documents are consumed directly from the repository.

### Commit Convention

Conventional commits format:
- `docs: <lowercase description>` for content changes
- `chore: <lowercase description>` for structural changes

## Key Patterns

1. **Document internal structure:** H1 title → Summary paragraph → Themed H2 sections → Limits/Boundaries section → Related Reading (cross-references)
2. **Cross-references:** Always use relative paths (`../platform/ARCHITECTURE.md`)
3. **Content governance:**
   - Public-safe only — no licensed HEDIS content, customer data, or source internals
   - Evidence-backed claims — traceable to generated inventories
   - NDA escalation — deeper detail is pointed to, not embedded
   - Conservative positioning — no marketing hyperbole
4. **Prose style:** Declarative, third-person, bullet-list-heavy, no frontmatter/YAML
5. **No images or media** — pure text with occasional fenced `text` blocks for diagrams

## Dependencies

None. No `package.json`, `requirements.txt`, `Cargo.toml`, `Makefile`, or any dependency manifest exists in the repository.

## Team & Activity

| Metric | Value |
|--------|-------|
| Contributors | 1 (single author) |
| Total commits | 6 |
| Branch strategy | Single branch (`master`), no tags |
| Last commit | 2026-03-01 |
| Months inactive | ~3 (as of 2026-05-25) |
| Review process | None (no PRs, single contributor) |
| Remotes | `origin/master`, `gitea/master` (dual-hosted) |

## Risks & Recommendations

| Concern | Severity | Detail | Recommendation |
|---------|----------|--------|----------------|
| Uncommitted changes across all 13 files | Medium | Every content file is modified but not committed — risk of work loss | Commit or stash immediately |
| No CI/CD or linting | Low | No automated link checking, spell checking, or Markdown validation | Add markdownlint + link checker as pre-commit or CI step |
| No versioning strategy | Low | No tags or release branches — impossible to snapshot a deck version for a specific meeting | Tag releases by investor meeting or quarter |
| 3-month inactivity | Medium | Repository may be stale or abandoned; unclear if modifications represent active work or drift | Clarify status; commit or discard uncommitted changes |
| Single contributor, no review | Low | No code review or approval process; content accuracy depends on one person | Consider adding a reviewer for factual claims |
| No CLAUDE.md or CONTRIBUTING.md | Low | No onboarding documentation for contributors | Add contributor guide if team grows |
| No .gitignore | Info | Potential for accidental commits of OS/editor artifacts | Add standard .gitignore |
