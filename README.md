# Codex customizations

Reusable Codex skills and an optional automatic post-change review hook.

A **skill** is a set of instructions that teaches Codex how to run a particular
workflow. After installing a skill, invoke it by name in a prompt, for example
`Use $simplify to clean up the files changed in this branch.` Skills do not run
continuously in the background. The hook is separate executable automation
that can request a review after Codex changes code.

See the official Codex documentation for [skills](https://developers.openai.com/codex/skills)
and [hooks](https://developers.openai.com/codex/hooks).

## Included workflows

| Workflow | Use it when | What it does |
| --- | --- | --- |
| [`feature-planning`](skills/feature-planning/) | You have a feature idea that needs discussion and a repository-grounded plan before implementation. | Assesses the idea, resolves material design choices with you, creates a plan only when requested, and reviews the plan for correctness and scope. |
| [`peer-review`](skills/peer-review/) | A document or code change warrants a bounded adversarial review. | Coordinates Codex with Claude, Gemini, or another Codex agent until both sides agree on the reviewed version or report a clear blocker. |
| [`simplify`](skills/simplify/) | Recently changed code works but could be clearer. | Applies conservative, local simplifications while preserving observable behavior and then runs proportionate validation. |
| [`staged-agent-implementation`](skills/staged-agent-implementation/) | A substantial, already-planned feature should be implemented in user-approved stages. | Delegates one bounded implementation step at a time, verifies it, records concise progress, and pauses before the next stage. |
| [`auto-review.py`](hooks/auto-review.py) | You want an automatic correctness and simplification pass after Codex changes source code. | Compares repository state at the start and end of a turn and requests one review pass when source or executable code definitely changed. |

## Install the skills

Clone the repository, then copy the skill directories into a user-level skill
location:

```sh
git clone https://github.com/arviphd/codex-customizations.git
cd codex-customizations
mkdir -p "$HOME/.agents/skills"
cp -R skills/* "$HOME/.agents/skills/"
```

Codex detects skill changes automatically. Restart Codex if newly installed
skills are not discovered immediately. For repository-scoped installation,
copy the selected directories into `<repo>/.agents/skills/` instead.

To install only one skill, copy only its directory. For example:

```sh
cp -R skills/simplify "$HOME/.agents/skills/"
```

## Usage examples

Invoke a skill explicitly with `$skill-name`, and include the target and any
important constraints in the same prompt.

### Plan a feature before writing code

Start with assessment rather than immediately asking for a plan:

```text
Use $feature-planning to assess adding offline sync to this application.
Discuss the design and trade-offs with me first; do not create the plan yet.
```

After resolving the important choices:

```text
Create the implementation plan now and save it under docs/plans/.
```

The skill preserves the accepted decisions across the discussion, creates one
canonical plan, and subjects it to focused correctness and scope reviews.

### Run an adversarial peer review

Use a Codex subagent as the independent reviewer:

```text
Use $peer-review Codex to review docs/offline-sync-plan.md. Fix confirmed
material findings, but do not broaden the accepted feature scope.
```

You can select Claude or Gemini instead:

```text
Use $peer-review Claude to review the current branch diff.
```

Claude and Gemini workflows require the corresponding environment to be
available. The skill uses a hash-bound Markdown ledger when the reviewer cannot
coordinate directly with Codex, preserving findings and approvals across
turns.

### Simplify changed code without redesigning it

```text
Use $simplify on src/parser.py and its changed tests. Preserve public APIs,
error behavior, output formatting, and performance characteristics.
```

The skill intentionally prefers a no-op over speculative cleanup. It does not
authorize feature work, bug fixes, broad refactors, or unrelated edits.

### Implement an approved plan in stages

```text
Use $staged-agent-implementation to implement docs/offline-sync-plan.md one
substantive step at a time. Show me the stage breakdown and wait for approval
before Step 1.
```

After reviewing a completed stage, authorize the next one explicitly:

```text
Proceed with Step 2.
```

This workflow is intended for substantial features with an already-reviewed
design. Small, self-contained changes generally do not need its ceremony.

## Install the optional automatic review hook

The hook is designed for macOS and requires Python 3, Git, and a Codex version
that supports lifecycle hooks. It is fail-open: unexpected hook errors do not
prevent Codex from finishing a turn.

```sh
mkdir -p "$HOME/.codex/hooks"
cp hooks/auto-review.py "$HOME/.codex/hooks/auto-review.py"
cp hooks/hooks.json "$HOME/.codex/hooks.json"
```

`hooks.json` is a complete example configuration. The last command replaces an
existing user hook configuration, so merge its `UserPromptSubmit` and `Stop`
entries manually if you already use other hooks.

After installing or changing the hook, open `/hooks` in Codex to review and
trust the exact hook definition. Codex skips untrusted user hooks.

At the start of a turn, the hook records a bounded snapshot of relevant Git
state. At the end, it requests one correctness-first review followed by
`$simplify` only when it can establish that source or executable code changed.
It ignores documentation-only changes and avoids claiming uncertain files as
definite code changes.

Optional environment variables control its limits:

| Variable | Default | Purpose |
| --- | ---: | --- |
| `CODEX_AUTO_REVIEW_MAX_CANDIDATES` | `50` | Maximum candidate paths considered in one turn. |
| `CODEX_AUTO_REVIEW_MAX_FILE_BYTES` | `26214400` | Maximum size of a worktree file inspected by the hook. |
| `CODEX_AUTO_REVIEW_MAX_DISPLAY_PATHS` | `40` | Maximum changed paths included in the generated review request. |
| `CODEX_AUTO_REVIEW_STATE_TTL_DAYS` | `7` | Age after which stale hook state is removed. |
| `CODEX_AUTO_REVIEW_STATE_DIR` | `~/Library/Caches/CodexAutoReview/state` | Override for temporary hook state. |

## Choosing a workflow

- Still deciding what to build: use `feature-planning`.
- Plan accepted and implementation is substantial: use
  `staged-agent-implementation`.
- Need an independent challenge to code or a document: use `peer-review`.
- Working code only needs conservative cleanup: use `simplify`.
- Want review to be requested automatically after coding turns: install the
  hook.

These workflows can be combined deliberately. For example, plan with
`feature-planning`, implement the accepted plan with
`staged-agent-implementation`, and use `peer-review` for a high-risk milestone.
Avoid invoking several overlapping workflows in one prompt unless their order
and scope are clear.

Generated review ledgers, local backups, caches, credentials, sessions, and
other machine-local Codex state are intentionally excluded from this
repository.

## Maintenance

Keep reusable source files here. Do not add generated handoff ledgers or
machine-local Codex state.
