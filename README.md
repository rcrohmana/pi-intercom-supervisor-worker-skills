# Pi Intercom Supervisor/Worker Skills

Two companion skills for running a safe supervisor/worker workflow across two local [Pi](https://github.com/badlogic/pi-mono) sessions using **pi-intercom**.

These skills are designed for [Pi](https://github.com/badlogic/pi-mono) and are meant to be used together with **Nico Bailon's original [`pi-intercom`](https://github.com/nicobailon/pi-intercom) extension/package**. They do not replace `pi-intercom`; they provide the operating protocol that makes intercom-based delegation safer and more reliable.

## Skills

### `supervisor`

Use in the planner/supervisor Pi session.

Responsibilities:

- Keep the main objective, scope, constraints, and acceptance criteria clear.
- Delegate bounded tasks to a worker through `pi-intercom`.
- Require completion/blocker reports through intercom, not local-only final text.
- Validate worker evidence before accepting results.
- Escalate important product, architecture, dependency, git, destructive, security, or verification-bypass decisions to the human boss.
- Recover safely if the worker stops without a completion/blocker report.

### `worker`

Use in the worker Pi session.

Responsibilities:

- Receive tasks from the supervisor through `pi-intercom`.
- Stay within the assigned mode and scope.
- Ask the supervisor when requirements are unclear or decisions are needed.
- Report progress for long-running work.
- Verify before claiming completion.
- End delegated work through intercom, not with normal local final text.

## Required dependency

Install and enable Nico Bailon's original [`pi-intercom`](https://github.com/nicobailon/pi-intercom) package first.

Example Pi settings package entry:

```json
{
  "packages": [
    "npm:pi-intercom"
  ]
}
```

Both sessions should have access to the `intercom` tool.

## Recommended terminal setup

Open two Pi sessions in the same machine:

```text
Terminal 1: /name supervisor  then /skill:supervisor
Terminal 2: /name worker      then /skill:worker
```

Both sessions should check connectivity:

```typescript
intercom({ action: "status" })
intercom({ action: "list" })
```

Expected default targets:

- `supervisor` sends tasks to `worker`
- `worker` reports back to `supervisor`

## Core protocol

The supervisor delegates using `send`, but every task must include a completion contract.

The worker must not stop with ordinary final text for delegated work. It must use one of these intercom actions:

| Situation | Required worker action |
|---|---|
| Complete | `intercom({ action: "ask", to: "supervisor", message: "Task complete: ..." })` |
| Blocked / unclear | `intercom({ action: "ask", to: "supervisor", message: "Blocked: ..." })` |
| Decision needed | `intercom({ action: "ask", to: "supervisor", message: "Decision needed: ..." })` |
| Still working | `intercom({ action: "send", to: "supervisor", message: "Progress: ..." })`, then continue working |

This avoids silent or local-only worker stops during long delegated tasks.

## Safe delegation template

```typescript
intercom({
  action: "send",
  to: "worker",
  message: `Task: <short title>
Goal: <desired outcome>
Mode: <investigate-only | implement | verify | review>
Context: <important background>
Scope: <files/areas allowed>
Constraints: <what not to change>
Acceptance criteria:
- <criterion 1>
- <criterion 2>
Verification required:
- <commands or checks>
Checkpoint guidance: <when to send progress>
Completion contract:
- If complete, your final action must be intercom ask to supervisor with prefix "Task complete:" and evidence.
- If blocked, unclear, or a decision is needed, your final action must be intercom ask to supervisor with prefix "Blocked:" or "Decision needed:".
- If still working on a long task, send progress to supervisor with prefix "Progress:", then continue working.
- Do not stop with a normal final message or method notes only.
Report back with: summary, files changed/inspected, verification output, risks, questions.`
})
```

## Installation

Copy the skill directories into your Pi user skills directory:

```text
~/.pi/agent/skills/supervisor/SKILL.md
~/.pi/agent/skills/worker/SKILL.md
```

Then start two Pi sessions and activate the relevant skill in each session.

## Acknowledgements

Thanks to the [Pi / pi-mono](https://github.com/badlogic/pi-mono) project for the coding-agent platform and to Nico Bailon for the original [`pi-intercom`](https://github.com/nicobailon/pi-intercom) extension that enables same-machine session coordination.

## License

MIT License. See [LICENSE](LICENSE).

## Notes

- These skills are prompt/protocol skills only.
- They do not modify `pi-intercom` internals.
- They are intended for same-machine Pi sessions connected through Nico Bailon's [`pi-intercom`](https://github.com/nicobailon/pi-intercom).
- For long or ambiguous work, keep tasks bounded and use progress checkpoints.
