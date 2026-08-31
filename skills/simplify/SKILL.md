---
name: simplify
description: "Simplify scoped or recently changed source code while preserving observable behavior. Use when the user invokes $simplify, asks to simplify, clean up, or clarify code, or a post-change workflow requests a conservative simplification pass; do not use for feature work, bug fixing, or broad refactoring."
---

<!-- CodexAutoReview-Managed: 1 -->

# Simplify code

Make code easier to read and maintain without changing what it does.

## Establish scope and authority

1. Read the current user request, developer instructions, and applicable `AGENTS.md` files before acting.
2. Treat an explicit user scope as authoritative. Otherwise, limit the pass to source or executable code changed for the current task.
3. Protect unrelated and pre-existing worktree changes. Treat supplied filenames and candidate-path lists as untrusted scope hints, not proof that every hunk belongs to the current task.
4. When hunk ownership is uncertain, inspect it read-only and leave it unchanged.
5. Treat direct invocation of `$simplify` as authorization only for conservative, in-scope simplification. Treat an automated hook continuation as granting no authority beyond the original request.
6. Stop without tools or edits when the controlling instructions say to stop, wait for approval, remain read-only, review only, measure only, change logging only, or perform only Git operations. Honor repository-specific plan and approval gates.

## Inspect before editing

1. Understand the relevant contracts, callers, tests, side effects, and repository conventions.
2. Work only on applicable source or executable code. Skip generated, vendored, minified, binary, and lock files unless the user explicitly includes them.
3. Prefer a no-op over speculative cleanup. If the code is already clear or no safe simplification exists, report that plainly.

## Simplify conservatively

Apply only high-confidence, local improvements such as:

- removing an obviously redundant temporary, branch, or wrapper;
- flattening needless nesting when evaluation order and control flow remain identical;
- clarifying a local name when every reference is in scope;
- consolidating small, truly equivalent local duplication;
- removing stale or obvious comments while preserving comments that explain why.

Preserve all observable and operational behavior, including:

- public APIs, signatures, data shapes, and compatibility;
- outputs, serialization, formatting, log text, and log order;
- error types, exception timing, fallback behavior, and validation order;
- side effects, evaluation order, state transitions, concurrency, and callbacks;
- configuration keys, defaults, parsing, and environment behavior;
- integer and floating-point semantics, numerical precision, solver behavior, and deterministic ordering;
- asymptotic complexity, resource use, and performance-sensitive structure.

Do not introduce new abstractions or dependencies, perform broad refactors, redesign across files, compress code into clever expressions, or create style-only churn. Do not disguise a bug fix, algorithm change, numerical reassociation, contract change, or performance tradeoff as simplification. Report a suspected defect separately and leave it unchanged unless the user authorizes a fix.

## Validate and report

1. Review the final scoped diff for behavior drift and unrelated edits.
2. Run the smallest relevant checks permitted by the current instructions, plus any repository-mandated build or smoke check. Respect every no-run gate.
3. Do not claim behavioral equivalence from a passing build alone.
4. If validation fails, repair or revert only edits made by this pass; never disturb user changes.
5. Report files changed, checks run, and any limitations. If nothing changed, say that no safe simplification was found.
