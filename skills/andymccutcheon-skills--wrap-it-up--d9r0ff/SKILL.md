---
name: wrap-it-up
description: Session conclusion and handoff generator. This skill should be used when the user signals the end of a working session — phrases like "wrap it up", "create a handoff", "summarize this session", "document what we did", "generate session summary", or any request to finalize and persist the session's work. Produces a structured session log, updated codebase documentation, and an agent-specific handoff document so the next session can onboard with minimal friction.
---

# Wrap It Up

## Overview

Decompose the current working session into three structured, persistent outputs designed for maximum continuity and minimum onboarding cost across sessions. Every output is written to a standard location under `.goose/session/` so the companion `boot-up` skill can discover them automatically.

## Output Locations

All outputs are written relative to the project root. When no project root is discernible, use the current working directory.

| Output | Path | Purpose |
|---|---|---|
| Session Log | `.goose/session/SESSION_LOG.md` | Chronological, factual record of the session |
| Handoff Doc | `.goose/session/HANDOFF.md` | Agent-optimized brief for the next session |
| Codebase Docs | Project-appropriate locations (README.md, CHANGELOG.md, `docs/`, etc.) | Updated living documentation |

The `assets/` directory in this skill contains template files that can be referenced for structure. The `references/` directory contains detailed format specifications.

## Workflow

### Phase 1: Gather Session Intelligence

Before writing any output, collect the following data. Use scripts and shell commands to avoid consuming context window on raw data.

**Git intelligence** — Run `scripts/gather-session-stats.sh` from the project root. This script captures:
- Files changed (staged and unstaged)
- Commit range (first and last commit touched this session)
- Diff summary (files touched, insertions, deletions)
- Branch and remote status

If the script is not available or the project is not a git repo, gather equivalent information manually with individual commands.

**Conversation intelligence** — Extract from the current conversation:
- The user's original request(s) and how they evolved
- Every explicit decision made and the rationale
- Every error encountered and how it was resolved
- Files created, modified, or deleted (cross-reference with git)
- Patterns or conventions established
- Work that was explicitly deferred or left incomplete

**Codebase intelligence** — Note the current state of:
- Project structure (run `tree` at depth 2-3 if not already known)
- Key configuration files and their purpose
- Active dependencies and versions
- Environment-specific setup (API keys needed, services running, ports in use)

### Phase 2: Produce the Session Log

Write `.goose/session/SESSION_LOG.md`. This is the authoritative factual record.

Load `references/session-log-format.md` for the complete format specification. At minimum, the log must include:

1. **Header**: Session date, duration (approximate), project/repo
2. **Objectives**: What the user set out to accomplish
3. **Chronological Log**: Time-ordered entries covering every significant action
   - Each entry: timestamp (approximate), action taken, files affected, outcome
   - Mark decisions with `[DECISION]` and errors with `[ERROR]` / `[RESOLVED]`
4. **Key Decisions**: Extracted list of every decision with rationale
5. **Errors & Resolutions**: Every error encountered and its fix
6. **File Manifest**: Every file created or modified, with a one-line summary of changes
7. **Deferred Work**: Explicit list of items mentioned but not completed
8. **Environment Notes**: Services running, ports used, credentials needed, gotchas

### Phase 3: Update Codebase Documentation

Identify all living documentation that should reflect the session's changes. Load `references/doc-update-guide.md` for detailed heuristics.

**Read-before-write rule**: Always read the current state of a documentation file before editing it. Never overwrite documentation blindly.

**Files to consider updating**:
- `README.md` — If new features, setup steps, or usage patterns were added
- `CHANGELOG.md` — If this project maintains one, append an entry
- `docs/` directory — Any architecture docs, API docs, or guides affected by changes
- `CLAUDE.md` — If project conventions or instructions for AI agents changed
- `.cursor/rules/` or similar — If project-specific AI rules were modified

**Update principles**:
- Additive by default — append or insert; never remove existing content unless it is factually incorrect
- Minimal and precise — add only what a future reader needs; do not narrate
- Cross-reference the session log for full context (`see .goose/session/SESSION_LOG.md`)

### Phase 4: Produce the Handoff Document

Write `.goose/session/HANDOFF.md`. This is the most critical output — it is the primary document the `boot-up` skill reads to onboard the next session.

Load `references/handoff-format.md` for the complete format specification.

The handoff is **agent-optimized**: it prioritizes information density and structural predictability over narrative prose. Every section follows a consistent schema that the `boot-up` skill can parse efficiently.

Required sections:

1. **Continuity Key** — 1-2 sentence summary of where things stand, formatted for immediate comprehension
2. **Active Context** — The 3-5 most important facts the next agent must know
3. **Current Task Stack** — What was being worked on, in priority order, with completion status
4. **Technical Surface** — Key files, their roles, and what needs attention in each
5. **Decisions Register** — Every decision made, with rationale, in compact form
6. **Blockers & Open Questions** — What is blocking progress, what needs answers
7. **Next Actions** — Concrete, ordered list of the first 3-5 things to do in the next session
8. **Environment Quick-Start** — The exact commands to get running again
9. **Reference Index** — Links to SESSION_LOG.md, updated docs, and any other relevant files

### Phase 5: Verify and Summarize

After writing all outputs:

1. Verify every file path referenced actually exists
2. Confirm git status aligns with the file manifest
3. Present a terse summary to the user covering:
   - Files produced (paths)
   - Documentation updated (paths)
   - Top 3 next actions from the handoff
   - One-line session summary

## Quick Reference

- **Session log format**: `references/session-log-format.md`
- **Handoff format**: `references/handoff-format.md`
- **Doc update heuristics**: `references/doc-update-guide.md`
- **Stats gathering script**: `scripts/gather-session-stats.sh`

## Keywords

wrap up, handoff, session summary, document session, end session, finalize, session log, HANDOFF.md, SESSION_LOG.md, summarize work, next session, onboarding
