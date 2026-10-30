# DS-2 — Documentation Drift Enforcement & Reporting

| Field | Value |
|---|---|
| Issue key | DS-2 |
| Type | Story |
| Epic | Developer Experience / Agentic SDLC |
| Priority | High |
| Reporter | Tech Lead |
| Assignee | Not Found |
| Status | To Do |
| Source | Local ticket file (stands in for JIRA/Confluence) |

## User Story

**As a** tech lead whose repositories now generate `PROJECT_DOCS.md` with `docsync` (DS-1),
**I want** documentation drift to be reliably caught in CI and clearly explained to the
person who caused it,
**so that** out-of-date docs can never be merged and fixing them takes seconds, not guesswork.

## Problem

DS-1 delivered `docsync generate` and `docsync check`, but in day-to-day use the
sync guarantee is weaker than it looks:

- Our CI pipeline runs `generate` immediately before `check`, so `check` compares the
  document with itself and can never fail. Stale docs still get merged.
- When `check` does report drift, it only says "out of sync". Developers have to
  regenerate and diff by hand to find out what changed, and some just regenerate and
  commit without looking.
- Running with a token and running without one produce different documents (hosted
  metadata resolved vs `Not Found`), so a developer's local run reports drift against
  the CI-generated file even when nothing in the code changed.
- Public repositories cannot show hosted metadata at all unless someone provides a
  token, even though the platform API allows anonymous reads.
- CI systems and PR bots have no machine-readable result to act on; they can only read
  the exit code.

## Scope

Extend the existing `docsync` CLI so that drift is enforced correctly in CI, explained
to humans, reported to machines, and consistent across environments. The document
format delivered in DS-1 stays the same unless a change is required to meet the
criteria below.

## Acceptance Criteria

- **AC1** — The repository's own CI must fail when the committed documentation is out
  of sync with the code, and pass when it is in sync.
- **AC2** — When drift is detected, the tool must tell the user which parts of the
  document are out of date and what changed, without them having to run anything else.
- **AC3** — Drift output must be available in a machine-readable form that a CI job or
  PR bot can consume.
- **AC4** — Two people running the tool on the same commit with the same settings must
  get the same result, regardless of whether one of them has an API token set.
- **AC5** — Hosted metadata for public repositories must be retrievable without a token.
- **AC6** — Secrets, tokens and credentials must never appear in any drift report,
  machine-readable output, or console output (the DS-1 AC3 guarantee extends to all new
  output).
- **AC7** — Existing DS-1 usage (`generate`, `check`, current flags and exit codes) must
  keep working; anyone already using DS-1 in CI must not be broken by this change.
- **AC8** — A team adopting `docsync` in a new repository must be able to add the sync
  gate to their own CI by following the project documentation.

## Non-Functional Expectations

- Drift reports should be readable at a glance in a CI log.
- Should stay fast; checking must not be noticeably slower than generating.
- Should remain safe to run in CI, including on pull requests from forks.
- Should be well tested, without real network calls.

## Notes / Open Questions from Refinement

- We did not agree on the drift report format (unified diff, per-section summary, or both)
  or how long a report may get before it is truncated.
- We did not define the machine-readable format or where it is written (stdout, a file,
  or both).
- We did not decide how to reconcile token / no-token runs: always offline in CI,
  anonymous API calls by default, or recording the mode in the document.
- Anonymous API rate limits were not discussed, nor what happens when they are hit.
- DS-1 design review rejected diff output (DR-10) as a leakage risk; refinement needs to
  confirm how AC2 and AC6 are satisfied together.
- Nobody specified whether a new flag, a new subcommand, or a changed default is
  preferred for the new behaviour.

## Out of Scope

- Automatically committing or pushing regenerated documentation.
- Posting comments to pull requests (only producing output a bot could use).
- Platforms other than GitHub; non-Python projects; multi-repository runs.
- Inferring the hosted repository from `git remote` (rejected in DS-1, DR-11).
- Publishing docs to a website.
