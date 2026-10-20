---
name: boot-up
description: Session onboarding accelerator. This skill should be used when the user starts a new session and needs to resume previous work — phrases like "boot up", "pick up where we left off", "get me up to speed", "what were we working on", "load the last session", "resume work", "continue from last session", or any request to re-establish context from prior sessions. Reads handoff documents, session logs, and codebase documentation in a token-efficient manner to onboard the agent with minimal context cost.
---

# Boot Up

## Overview

Efficiently re-establish working context from a prior session by reading structured outputs produced by the `wrap-it-up` skill. The primary onboarding document is `HANDOFF.md`; fall back to `SESSION_LOG.md` and codebase documentation only when the handoff is unavailable, stale, or insufficient.

The core principle is **progressive disclosure**: load the smallest document first (HANDOFF.md), then drill into larger documents only for specific information needed.

## Document Discovery

Session artifacts follow a standard location convention established by `wrap-it-up`:

```
.goose/session/HANDOFF.md       — Primary: agent-optimized handoff
.goose/session/SESSION_LOG.md   — Secondary: full chronological record
```

If the project uses a different convention, load `references/file-locations.md` for alternative layouts.

### When Standard Locations Are Empty

If `.goose/session/` does not exist or contains no relevant files, fall back to:

1. Search for any `HANDOFF.md` or `SESSION_LOG.md` in the project tree
2. Check `CLAUDE.md` or `CONTEXT.md` at the project root for project-level guidance
3. Read `README.md` for overall project orientation
4. Run `scripts/discover-sessions.sh` to scan for session artifacts in non-standard locations

## Loading Protocol

The loading order is designed to minimize token consumption while maximizing context recovery. Stop at any point when sufficient context is established to continue work.

### Step 1: Read HANDOFF.md (Primary)

Read `.goose/session/HANDOFF.md` **in full**. This document is designed to be compact and information-dense. From it, extract:

- **Continuity Key**: The 1-2 sentence summary of where things stand
- **Active Context**: The 3-5 critical facts for continuity
- **Current Task Stack**: What was in progress, ordered by priority
- **Next Actions**: The concrete first 3-5 things to do
- **Environment Quick-Start**: The exact commands to get running
- **Reference Index**: Links to any other files that may be needed

After reading the handoff, assess whether sufficient context is established to begin work. In most cases, the handoff alone is enough.

### Step 2: Read SESSION_LOG.md (Secondary — only if needed)

Load `.goose/session/SESSION_LOG.md` **only when**:
- HANDOFF.md is not found
- HANDOFF.md references a decision or error whose rationale is unclear
- The task requires understanding the full sequence of steps that led to the current state
- HANDOFF.md explicitly instructs reading the session log

When reading the session log, load `references/session-log-format.md` first to understand the structure, then read only the relevant sections:
- **Objectives** and **Key Decisions** sections provide 80% of the value
- Read **Errors & Resolutions** only if encountering similar errors
- Read **Chronological Log** entries only for specific time ranges of interest
- Skip **File Manifest** unless verifying a specific file's status

### Step 3: Load Updated Documentation (Tertiary — only if needed)

If the handoff or session log references specific documentation files (README.md, CHANGELOG.md, `docs/` entries), load only those files, and only the sections needed.

### Step 4: Verify Environment State

Before beginning work, verify that the environment described in the handoff matches reality:

1. Run the Environment Quick-Start commands from the handoff (if provided)
2. Check that the git branch matches the handoff's reported branch: `git branch --show-current`
3. Check for uncommitted changes: `git status --short`
4. Verify any services the handoff says should be running are actually running

Report any discrepancies to the user before proceeding.

## Presenting Context to the User

After loading, present a compact orientation:

```
## Session Resumed: [One-line summary from Continuity Key]

**Status**: [Current state from Active Context]
**Last worked on**: [Top item from Task Stack]
**Branch**: [Git branch]
**Next actions**:
1. [First next action]
2. [Second next action]
3. [Third next action]

**To get running**: [Key command from Environment Quick-Start]
```

Then proceed to the first next action **without waiting for confirmation**, unless the handoff indicates a blocker that requires user input.

## Handling Missing or Stale Handoffs

### No Handoff Found

If no HANDOFF.md exists anywhere in the project:
1. Check for SESSION_LOG.md — if present, treat it as the primary document (read Objectives, Key Decisions, and Deferred Work sections)
2. If neither exists, read README.md and CLAUDE.md for project orientation
3. Present what is known and ask the user for direction

### Handoff Appears Stale

If the handoff references files that no longer exist, a git branch that has diverged significantly, or dates that are more than a week old:
1. Note the discrepancies
2. Cross-reference with `git log --oneline -10` to understand what happened since
3. Present the handoff's content alongside the discrepancies
4. Ask the user which direction to pursue

## Quick Reference

- **File location conventions**: `references/file-locations.md`
- **Session log structure**: `references/session-log-format.md` (shared with wrap-it-up)
- **Session discovery script**: `scripts/discover-sessions.sh`

## Keywords

boot up, resume, pick up where we left off, get up to speed, continue, last session, handoff, HANDOFF.md, what were we working on, load session, previous session, onboarding, get me caught up
