# Pi Intercom Supervisor/Worker Skills

Two companion [Pi](https://github.com/badlogic/pi-mono) skills for coordinating a persistent supervisor and worker across local sessions.

The skills use structured work leases provided by the [`rcrohmana/pi-intercom`](https://github.com/rcrohmana/pi-intercom) fork. A lease gives delegated work an explicit lifecycle and lets the extension wake a worker that settles before reporting a terminal result. This is especially useful when GPT-5.6 Terra is selected as the worker model.

## What each skill does

### `supervisor`

The supervisor keeps the overall objective and delegates bounded tasks. It:

- defines the goal, scope, constraints, acceptance criteria, and verification;
- starts executable work with `delegate`;
- avoids routine acknowledgements and manual `continue` messages;
- reviews and independently verifies completion evidence;
- uses the same work ID with `resume` when rework is required;
- escalates important decisions to the human boss.

### `worker`

The worker executes continuously under the active lease. It:

- stays within the delegated scope;
- continues concrete work after meaningful progress reports;
- requests decisions only when safe execution genuinely depends on an answer;
- reports a real external blocker through `blocked`;
- runs fresh verification before using `complete`;
- never treats an empty response as a completion signal.

## Requirements

Install a version of [`rcrohmana/pi-intercom`](https://github.com/rcrohmana/pi-intercom) that includes structured work leases. Both Pi sessions must expose the `intercom` tool with these actions:

| Action | Purpose |
|---|---|
| `delegate` | Start a bounded work lease |
| `progress` | Report a meaningful milestone and keep working |
| `complete` | End the lease after verification |
| `blocked` | Pause for a genuine external blocker |
| `resume` | Reactivate the same work ID for reviewed rework |
| `cancel` | Stop delegated work |

Generic `send` is still appropriate for ordinary messages. Use blocking `ask` only when a decision must be answered before the worker can proceed safely.

## Installation

Clone this repository:

```bash
git clone https://github.com/rcrohmana/pi-intercom-supervisor-worker-skills.git
```

Then either copy the skill directories into Pi's global skill directory:

```text
~/.pi/agent/skills/supervisor/SKILL.md
~/.pi/agent/skills/worker/SKILL.md
```

or add both directories to the `skills` array in `~/.pi/agent/settings.json`:

```json
{
  "skills": [
    "/absolute/path/pi-intercom-supervisor-worker-skills/supervisor",
    "/absolute/path/pi-intercom-supervisor-worker-skills/worker"
  ]
}
```

Restart Pi or reload the affected sessions after installation.

## Session setup

Open two Pi sessions on the same machine:

```text
Terminal 1: /name supervisor  then /skill:supervisor
Terminal 2: /name worker      then /skill:worker
```

Check connectivity from both sessions:

```typescript
intercom({ action: "status" })
intercom({ action: "list" })
```

The examples below use `supervisor` and `worker` as the session names.

## Structured workflow

### 1. Delegate bounded work

From the supervisor session:

```typescript
intercom({
  action: "delegate",
  to: "worker",
  message: `Task: Add bounded retry handling
Goal: Prevent premature task termination without changing providers.
Mode: implement
Context: The worker may settle after a tool result.
Scope: src/retry.ts and its tests
Constraints: Do not add dependencies or change public APIs.
Acceptance criteria:
- Retry state is bounded.
- Terminal states stop retries.
Verification required:
- npm test
Report complete with summary, changed paths, exact verification output, risks, and questions.`
})
```

The tool result includes a `workId`. Keep it for rework or cancellation. The supervisor should not send routine acknowledgements when the worker reports progress.

### 2. Report progress without stopping

From the worker session:

```typescript
intercom({
  action: "progress",
  message: "Focused regression is green. Continuing with the full suite and diff review."
})
```

After this call, the worker continues with the next concrete non-intercom tool action.

### 3. Request a blocking decision

Use `ask` only when no safe path remains without supervisor input:

```typescript
intercom({
  action: "ask",
  to: "supervisor",
  message: `Decision needed: retry storage
Context: The existing API supports memory or disk persistence.
Options:
A) Keep memory-only state and preserve restart behavior.
B) Add disk persistence and a migration path.
Recommendation: A, because persistence is outside the delegated scope.
Blocking: Choosing either option changes the implementation boundary.`
})
```

The supervisor answers with `reply`. Once answered, the worker resumes concrete work rather than sending an acknowledgement.

### 4. Complete with evidence

From the worker session:

```typescript
intercom({
  action: "complete",
  message: `Task complete: bounded retry handling
Summary: Added an in-memory bounded retry controller.
Files changed: src/retry.ts, src/retry.test.ts
Verification:
- npm test: 42 passed, 0 failed
Risks/notes: Retry state intentionally resets after process restart.
Ready for supervisor review.`
})
```

If delivery fails, the lease remains active. The supervisor independently checks the files and reruns required verification before accepting the result.

### 5. Resume reviewed rework

If verification finds a gap, the supervisor reuses the original work ID:

```typescript
intercom({
  action: "resume",
  to: "worker",
  workId: "<original-work-id>",
  message: `Rework required:
Issue: The continuation cap is not covered by a boundary test.
Expected: Add a test for the exact maximum and the first rejected continuation.
Keep: Existing public behavior.
Verification: npm test`
})
```

A distinct task should receive a new `delegate` call rather than being appended to an old lease.

### 6. Report a real blocker

From the worker session:

```typescript
intercom({
  action: "blocked",
  message: `Blocked: repository permission
What I tried: git push origin master
Evidence: remote rejected the authenticated user.
Needed from supervisor: repository access or an approved alternate remote.`
})
```

Unfinished implementation, an expected failing test, formatting work, or needing more time are not blockers.

### 7. Cancel work

From the supervisor session:

```typescript
intercom({
  action: "cancel",
  to: "worker",
  workId: "<work-id>",
  message: "Stop this task; the requirements changed."
})
```

## Runtime safety

The companion pi-intercom extension owns lifecycle continuation:

- a settled active lease triggers an automatic continuation turn;
- two consecutive settled runs without concrete non-intercom tool progress stall safely;
- fifty automatic continuations also stall safely;
- `complete`, `blocked`, `cancel`, shutdown, or session replacement stops continuation;
- leases are held in memory and do not automatically survive a Pi process restart.

These guards complement the skills. They do not replace clear task boundaries, independent verification, or human approval for important decisions.

## Acknowledgements

Thanks to the [Pi / pi-mono](https://github.com/badlogic/pi-mono) project for the coding-agent platform and to Nico Bailon for the original [`pi-intercom`](https://github.com/nicobailon/pi-intercom) extension.

## License

MIT License. See [LICENSE](LICENSE).
