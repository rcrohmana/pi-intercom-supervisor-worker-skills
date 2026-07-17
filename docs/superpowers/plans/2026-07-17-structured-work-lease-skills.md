# Structured Work-Lease Skills Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Publish supervisor and worker skills whose operating contract matches pi-intercom's structured work-lease lifecycle.

**Execution status:** Completed on 2026-07-17.

**Architecture:** Keep the two existing standalone Pi skills as the authoritative role prompts and make the repository README the installation and protocol guide. Validation is documentation-contract based because this repository contains no executable code.

**Tech Stack:** Markdown, YAML frontmatter, Git, ripgrep.

## Global Constraints

- Keep the repository's existing `master` default branch.
- Do not add dependencies or executable code.
- Preserve GPT-5.6 Terra as the selected worker model.
- Structured lifecycle actions are exactly `delegate`, `progress`, `complete`, `blocked`, `resume`, and `cancel`.
- Generic `ask` is only for a decision that blocks safe execution.
- Work leases remain memory-only and do not survive Pi process restart automatically.

---

### Task 1: Validate the role skill contracts

**Files:**
- Modify: `supervisor/SKILL.md`
- Modify: `worker/SKILL.md`

**Interfaces:**
- Consumes: structured work actions exposed by the companion pi-intercom fork.
- Produces: two Pi skills with `name: supervisor` and `name: worker` frontmatter and complementary lifecycle rules.

- [ ] **Step 1: Validate frontmatter and action coverage**

Run:

```bash
node - <<'NODE'
const fs = require('fs');
for (const [file, name] of [['supervisor/SKILL.md', 'supervisor'], ['worker/SKILL.md', 'worker']]) {
  const text = fs.readFileSync(file, 'utf8');
  if (!text.startsWith(`---\nname: ${name}\n`)) throw new Error(`${file}: invalid frontmatter`);
  for (const action of ['delegate', 'progress', 'complete', 'blocked', 'resume', 'cancel']) {
    if (!text.includes(action)) throw new Error(`${file}: missing ${action}`);
  }
}
NODE
```

Expected: exit code 0.

- [ ] **Step 2: Reject legacy lifecycle guidance**

Run:

```bash
if rg -n 'final action must be intercom ask|delegate using `send`|Do not produce a normal assistant final response' supervisor/SKILL.md worker/SKILL.md; then exit 1; fi
```

Expected: exit code 0 with no matches.

- [ ] **Step 3: Review the role boundary**

Confirm `supervisor/SKILL.md` independently verifies completion and protects boss-approval decisions, while `worker/SKILL.md` performs continuous concrete work and reports exact verification evidence.

### Task 2: Replace the legacy README protocol

**Files:**
- Modify: `README.md`

**Interfaces:**
- Consumes: the role contracts from Task 1 and the structured pi-intercom fork.
- Produces: installation and usage documentation with no prefix-parsing completion contract.

- [ ] **Step 1: Prove the current README is stale**

Run:

```bash
rg -n 'delegates using `send`|final action must be intercom ask|prompt/protocol skills only' README.md
```

Expected: at least one match before replacement.

- [ ] **Step 2: Rewrite README sections**

Replace the README with these sections in order:

1. project purpose and dependency on `rcrohmana/pi-intercom` structured work leases;
2. supervisor and worker responsibilities;
3. installation paths and two-terminal setup;
4. action table for `delegate`, `progress`, `complete`, `blocked`, `resume`, and `cancel`;
5. complete delegation, progress, decision, completion, rework, and cancellation examples;
6. watchdog limits and memory-only lease behavior;
7. acknowledgements and MIT license.

The examples must use structured actions directly and must not parse message prefixes.

- [ ] **Step 3: Validate README lifecycle coverage**

Run:

```bash
node - <<'NODE'
const fs = require('fs');
const text = fs.readFileSync('README.md', 'utf8');
for (const action of ['delegate', 'progress', 'complete', 'blocked', 'resume', 'cancel']) {
  if (!text.includes(action)) throw new Error(`README missing ${action}`);
}
for (const legacy of ['delegates using `send`', 'final action must be intercom ask']) {
  if (text.includes(legacy)) throw new Error(`README retains legacy guidance: ${legacy}`);
}
NODE
```

Expected: exit code 0.

### Task 3: Verify, commit, and push

**Files:**
- Modify: `README.md`
- Modify: `supervisor/SKILL.md`
- Modify: `worker/SKILL.md`
- Create: `docs/superpowers/plans/2026-07-17-structured-work-lease-skills.md`

**Interfaces:**
- Consumes: all documentation produced by Tasks 1 and 2.
- Produces: a clean `master` branch published to `origin/master`.

- [ ] **Step 1: Scan for placeholders and whitespace errors**

Run:

```bash
if rg -n 'T[B]D|T[O]DO|implement l[a]ter|fill i[n]' README.md supervisor/SKILL.md worker/SKILL.md docs/superpowers; then exit 1; fi
git diff --check
```

Expected: both commands exit 0.

- [ ] **Step 2: Run complete contract verification**

Run the frontmatter, action coverage, legacy-guidance, and README checks from Tasks 1 and 2 together.

Expected: all checks exit 0.

- [ ] **Step 3: Commit the implementation**

```bash
git add README.md supervisor/SKILL.md worker/SKILL.md docs/superpowers/plans/2026-07-17-structured-work-lease-skills.md
git commit -m "feat: adopt structured work lease workflow"
```

Expected: one implementation commit containing the three user-facing documents and this plan.

- [ ] **Step 4: Push without force**

```bash
git fetch origin master
git merge-base --is-ancestor origin/master HEAD
git push origin master
```

Expected: `origin/master` advances to the local `HEAD` without force-push.

- [ ] **Step 5: Verify remote state**

```bash
REMOTE_SHA=$(git ls-remote origin refs/heads/master | cut -f1)
test "$REMOTE_SHA" = "$(git rev-parse HEAD)"
git status --short --branch
```

Expected: remote SHA equals local `HEAD`; working tree is clean.
