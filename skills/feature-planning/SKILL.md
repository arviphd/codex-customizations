---
name: feature-planning
description: Assess and refine a proposed software feature interactively before creating a repository-grounded implementation plan, then review it through correctness, scope-control, preservation, and high-value-only sanity passes until it converges. Use when the user wants deliberate feature planning rather than immediate implementation.
---

# Feature Planning

Turn an evolving feature idea into an implementation-ready plan without drafting
too early or expanding the requested scope. Treat the conversation as one
continuous planning workflow and preserve every accepted user decision across
passes.

## Workflow state

Track the current phase from the conversation:

1. assessment only;
2. collaborative refinement;
3. plan creation;
4. substantive review and correction;
5. scope-control and preservation review;
6. high-value-only sanity review; or
7. converged.

Do not restart the workflow or discard earlier decisions when the user moves to
the next phase. A new or changed requirement may reopen an earlier phase when it
materially changes the design.

## 1. Assess before planning

When the user initially asks for thoughts, assessment, or discussion before a
plan:

- Do not create a plan document, roadmap, task breakdown, or implementation
  sequence.
- Evaluate the proposed behavior, user value, likely failure modes, important
  tradeoffs, and fit with the current product or architecture.
- Separate observations, recommendations, assumptions, and unresolved choices.
- Point out material contradictions or missing behavior, but do not invent
  requirements for completeness.
- Lead with the assessment and the few decisions worth discussing.

Inspect the repository only when it helps answer the assessment accurately.
Read applicable repository instructions before doing so. Assessment authorizes
read-only investigation, not implementation.

## 2. Refine interactively

As the user adds or changes requirements:

- Incorporate each decision into the working design and explain any meaningful
  consequence.
- Preserve the user's terminology unless a clearer term prevents a product or
  implementation ambiguity.
- Identify conflicts with earlier decisions explicitly; do not silently choose
  one.
- Ask only when a missing choice would materially alter the result and cannot
  be resolved from repository evidence or a safe assumption. When a question
  is necessary, ask one decision at a time.
- When genuinely different approaches remain viable, present a small set with
  concrete tradeoffs and recommend one. Do not manufacture alternatives when
  one approach clearly fits the accepted requirements.
- Continue discussing the feature rather than producing the plan until the user
  explicitly asks to create it.

Keep an internal decision set. Do not create a separate decision-log artifact
unless the user requests one.

## 3. Create the plan only on request

Create the plan document after an explicit instruction such as “create the
plan” or “work out the implementation details and create the plan.” Before
writing it:

- Read the applicable repository instructions and the minimum relevant code,
  architecture, capability, and validation documentation.
- Verify current file names, contracts, configuration shapes, and existing
  behavior instead of planning from memory.
- Distinguish current capability from proposed behavior.
- Resolve remaining implementation details that can be determined safely from
  the repository. Surface only genuinely material unresolved choices.
- If the request contains independent subsystems that do not form one coherent,
  testable vertical slice, recommend separate plans instead of hiding several
  projects inside one document.

Use the repository's existing documentation location and naming conventions.
Maintain one canonical plan document rather than creating competing versions or
sidecar checklists unless the user requests them. Make it usable by a fresh
implementer with the plan and repository, without requiring the raw chat, while
linking to authoritative repository material instead of duplicating it.

The plan should contain only sections useful to implementing and validating the
feature. Depending on the feature, useful content may include outcome and
boundaries, product behavior and state transitions, contracts and data flow,
failure behavior, implementation map, sequence, focused tests, non-goals, and
completion criteria. Do not include a section merely to satisfy a template.

The plan must retain all accepted requirements and map material work to actual
components. Order implementation by dependency and risk so each substantive
phase ends in an independently verifiable outcome. State completion criteria in
observable terms and connect them to focused validation without building an
exhaustive traceability matrix.

Before presenting the first draft, make one lightweight self-review for omitted
accepted requirements, internal contradictions, unresolved placeholders or
`TBD`s, ambiguous ownership, and completion criteria that cannot be tested.
Correct only confirmed issues. This check does not replace later user-requested
review passes. Do not implement the feature unless the user separately asks for
implementation.

## 4. Make substantive review passes

A plain request to review is read-only: inspect the complete current plan and
relevant repository evidence, then report findings without editing. Edit the
plan only when the user asks to fix, update, revise, apply findings, or otherwise
authorizes a change pass. In either mode, search for high-value issues in this
order:

1. mismatch with an explicit user decision;
2. incorrect or infeasible assumptions about current code or architecture;
3. missing state transitions, ownership, data flow, failure behavior, or
   compatibility needed for demonstrated behavior;
4. security, privacy, mutation, or authority-boundary errors;
5. implementation ambiguity likely to cause divergent code;
6. missing validation that could allow the feature to appear complete while its
   required behavior fails; and
7. unnecessary scope, abstraction, machinery, or operational burden.

When changes are authorized, apply the smallest focused correction for confirmed
issues. Preserve unrelated content and user-owned repository changes. Do not
rewrite the document merely to make it sound different.

If the user explicitly invokes another review skill, use it as requested and
integrate only validated findings. A peer review does not replace Codex's own
review or expand the plan's scope.

## 5. Enforce POC scope deliberately

Treat POC scope as binding when the user states it. During the POC pass, remove
or narrow items whose main value is production hardening, generalized reuse, or
hypothetical completeness rather than the demonstrated vertical slice.

Typical low-value POC scope includes:

- generic frameworks, registries, plugin systems, or speculative abstraction;
- automatic discovery, fallback, orchestration, or configurability not needed
  for the demonstrated path;
- multi-user, distributed, background, durable-recovery, administration, or
  observability machinery without an observed need;
- exhaustive edge-case matrices and broad refactors; and
- prose, repeated rationale, or process ceremony that does not guide an
  implementation decision.

When planning exposes useful work outside the demonstrated slice, mark it as a
non-goal or deferred follow-up instead of absorbing it into the POC. Propose a
separate plan only when that work is independently worth pursuing.

Do not remove small safeguards required for correctness, security, privacy,
data integrity, failure isolation, or a user-visible requirement merely because
they sound production-oriented. Prefer the simplest fail-closed behavior that
keeps the POC honest.

## 6. Run a preservation pass after pruning

After scope reduction, make one deliberate pass to ensure useful content was
not lost. Check that the plan still contains:

- every accepted user-visible behavior;
- the minimum end-to-end implementation path;
- essential state invalidation and failure semantics;
- authoritative data and mutation boundaries;
- focused validation of the demonstrated behavior; and
- enough file and contract detail for implementation without rediscovery.

Restore only material content. Do not use this pass to reintroduce generalized
or production-only scope.

## 7. Converge with high-value-only sanity passes

Once the user asks for a sanity pass or says changes should be high-value only:

- Review the plan and, when that request authorizes changes, edit only for a
  confirmed high-value issue affecting requirements, correctness, feasibility,
  safety, POC scope, or meaningful testability.
- Do not make wording-only, stylistic, organizational, speculative, or
  “nice-to-have” changes.
- Do not feel obligated to change the document. “No high-value issues found” is
  a successful result.
- After any high-value correction, run proportionate checks and allow a later
  sanity pass to test the new state.
- Declare convergence when a complete sanity pass finds no high-value issue.
  Stop manufacturing additional passes or churn. If the user requests another
  sanity pass, perform one fresh check under the same threshold.

## Review and handoff discipline

- State what materially changed in each pass, or state clearly that no change
  was warranted.
- Keep validation proportionate to a plan document: structural/document checks
  plus repository-required documentation gates. Do not run expensive execution
  suites unless the plan change or repository instructions require them.
- Never claim implementation or tested runtime behavior when only the plan was
  changed.
- At convergence, summarize the agreed feature, POC boundary when applicable,
  remaining explicit assumptions, plan location, and validation performed.
