# Structured Work-Lease Skills Design

**Date:** 2026-07-17
**Status:** Approved

## Goal

Update the standalone supervisor and worker skills to use pi-intercom's structured work-lease lifecycle so a persistent GPT-5.6 Terra worker can continue through premature settled turns without manual `continue` messages.

## Scope

Update:

- `supervisor/SKILL.md`
- `worker/SKILL.md`
- `README.md`

The repository keeps its existing `master` default branch. No branch migration, dependency, executable code, or model/provider change is included.

## Protocol

Executable work uses these structured pi-intercom actions:

- `delegate`: start a bounded worker lease;
- `progress`: report a meaningful milestone without ending the lease;
- `complete`: terminate after fresh verification;
- `blocked`: pause only for a genuine external blocker;
- `resume`: reactivate the same work ID for reviewed rework;
- `cancel`: stop work when the supervisor changes direction.

Generic `send` remains suitable for ordinary messages, and blocking `ask` remains suitable for decisions that must be answered before safe execution. Neither replaces structured delegation or terminal reporting.

## Supervisor Contract

The supervisor:

1. delegates one coherent task with explicit goal, scope, constraints, acceptance criteria, and verification;
2. does not acknowledge routine progress or manually send `continue` after transient idle status;
3. independently validates structured completion evidence;
4. uses the same work ID with `resume` for rework;
5. uses `cancel` when work must stop;
6. escalates product, architecture, dependency, destructive, git/release, security, and verification-bypass decisions to the human boss.

## Worker Contract

The worker:

1. treats an active work lease as executable until `complete`, `blocked`, or `cancel`;
2. never uses an empty response as a task boundary;
3. continues with a concrete non-intercom tool call after `progress` or rework instructions;
4. uses `ask` only for a genuinely blocking decision;
5. uses `blocked` only for an external blocker, not unfinished implementation;
6. reports exact verification evidence through `complete`.

The selected worker remains GPT-5.6 Terra.

## README

The README will:

- identify the structured-work-lease-capable pi-intercom fork as the required implementation;
- explain installation of both skill directories;
- document terminal setup and the six work actions;
- replace legacy `send` plus prefix-parsing examples;
- state the in-memory lease and watchdog safety limits;
- retain acknowledgements and MIT licensing information.

## Verification

Before push:

1. validate frontmatter names and descriptions;
2. verify all six structured actions appear in both skills and the README;
3. ensure legacy completion-through-`ask` guidance is absent;
4. scan for placeholders and contradictory lifecycle guidance;
5. run `git diff --check`;
6. verify the commit and remote branch after push.
