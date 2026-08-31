---
name: staged-agent-implementation
description: Implement substantial repository features in user-gated steps using fresh implementation subagents that inherit the parent agent's model and reasoning settings, one concise progress record, focused root verification against an already-reviewed design, and proportionate validation. Use for step-by-step roadmap implementation, delegated coding, fresh implementation contexts, or durable implementation evidence. Exclude deep design critique, exhaustive edge-case analysis, and repeated independent-review cycles unless the user requests them or a critical implementation risk requires escalation. Skip this ceremony for small, self-contained changes.
---

# Staged Agent Implementation

Keep the root agent responsible for scope, acceptance, integration, reporting,
and user gates. Use fresh implementation agents for bounded steps.

## Set authority and plan

Before implementation:

1. Read applicable instructions, branch, `HEAD`, status, relevant design
   documents, and existing changes. Identify protected or concurrent work.
   Record fingerprints when comparability or accidental drift matters.
2. Treat the canonical design or plan as an already-reviewed implementation
   contract. Check only for contradictions with live code, missing information
   that blocks implementation, or a choice that would materially change the
   requested behavior. Do not perform a fresh deep design review by default.
3. Divide the feature into substantive, independently verifiable steps. Give
   each an outcome, file scope, validation, progress-document section, and
   exit gate. Attach purely mechanical setup to the next substantive step.
4. Distinguish roadmap approval from execution authority. Accepting the
   roadmap does not authorize implementation. A user instruction to proceed
   authorizes only the next substantive step.
5. Treat validation and measurements explicitly listed and budgeted in the
   step as authorized. Keep unplanned or expanded measurement, Git,
   publication, material deletion, and scope expansion as separate boundaries.
6. Present the plan and obtain authority for Step 1.

Order work to expose bugs early: wiring with little or no logic, the smallest
verifiable core, complexity-increasing layers, edge/output handling, then
expensive scale or production validation. If live evidence invalidates the
remaining plan, revise it and return to the user instead of following it
mechanically.

Use reasonable implementation assumptions for small ambiguities that do not
change the design's intent. Record them briefly instead of stopping. Escalate
only when the missing decision changes behavior, scope, compatibility, cost,
or another material trade-off.

## Keep design review separate

Implement the accepted design unless the user also asks for design review. Do
not reopen accepted choices, pursue theoretical completeness, expand scope for
speculative concerns, or block on rare unsupported scenarios. Record those
observations as deferred. Escalate only a concrete risk to current correctness,
data integrity, determinism, security, recoverability, or a material
implementation choice that cannot safely be inferred.

## Preserve durable state

For an effort that warrants this staged skill, maintain:

- one canonical design containing accepted decisions;
- one canonical progress document with a concise section per substantive step;
  and
- an invariant list only when the design already defines one or the feature has
  a small set of genuinely load-bearing rules that future steps could violate.

Use an existing repository convention or a path approved in the plan. Keep
the progress document uncommitted by default and distinguish temporary
roadmaps from permanent documentation. Use one document rather than a report
per stage. Append one `Draft` section per step, finalize it in place after root
acceptance, and update a compact status index. Treat accepted sections as
immutable except for noted factual corrections. Leave historical stage reports
intact unless the user requests consolidation; link to them once.

Use this compact stage-section shape unless the repository has a better
convention:

```markdown
## Step NN - Outcome
Status: Draft | Accepted

### Changes and decisions
### Validation and evidence
### Review and remediation
### Limitations and deferred checks
### Git state and next authority boundary
```

Record only material hazards with their symptom, mechanism, and resolution.
Avoid duplicate facts. Re-prime fresh agents from live code, the design,
applicable invariants, and the latest accepted stage section—not the raw chat.
Keep sections short and link to source, tests, logs, or evidence instead of
pasting outputs or repeating design decisions. Keep any added invariants in
the progress document.

Do not add hashes, exhaustive trace tables, or duplicate evidence unless the
design, repository rules, or user explicitly requires them. The progress
document is an implementation handoff, not a second design-review report.

## Brief and delegate one step

Make the step brief self-contained:

- objective, starting branch and `HEAD`;
- relevant source documents, code, tests, and latest accepted state;
- allowed and forbidden changes, including protected paths;
- behavioral invariants and compatibility boundaries;
- acceptance checks and the main implementation risk to verify;
- a reasonably cheap forced-case or exact reference fixture only when normal
  execution could otherwise avoid the new branch;
- budgets for expensive runs and explicitly deferred validation;
- canonical progress-document path and new stage heading; and
- Git and publication policy, normally no stage, commit, push, merge, or
  publication.

For controlled comparisons, change one material factor at a time and record
input, configuration, and source fingerprints.

For each authorized substantive step, spawn a fresh implementation subagent.
Do not set `model` or `reasoning_effort`; inherit both from the parent agent.
Keep the implementation context fresh:

```text
fork_turns: none
```

Tell it to inspect live code, preserve unrelated work, implement only this
step, review its own substantive diff before any expensive run, validate
proportionately, append one clearly marked draft stage section to the canonical
progress document, and stop. Do not let it create a separate permanent stage
report. Reuse it only for tightly related remediation while its context
remains small.

The draft contains only current facts: outcome and provisional verdict;
starting and ending worktree state; files and decisions; commands and exact
results; applicable runtime evidence; and limitations or deferred checks.

## Run focused root acceptance

Use this sequence:

```text
implement and append draft section → focused root verification
→ one in-scope remediation pass if needed
→ second pass only for an unresolved blocker → targeted validation
→ evaluate any lifecycle stop-hook activation condition
→ stop-hook handoff only when that condition is definitely satisfied
→ hook remediation and smallest affected validation
→ finalize stage section → final user summary
```

1. Inspect the live diff, status, ownership, and current draft stage section.
   Leave unrelated or concurrent hunks untouched.
2. Verify that the implementation matches the accepted design and step scope.
   Review the changed logic, realistic error paths, configuration wiring,
   directly affected callers, and relevant mirrored implementations. Stay
   within changed paths, planned checks, and the relevant regression surface;
   expand only when a failure or concrete code path shows a real interaction.
   Check the load-bearing crux only as far as needed to trust the
   implementation; do not re-review the design or search exhaustively.
3. Run the smallest meaningful targeted tests plus a relevant regression.
   Add a reasonably cheap forced-case test when normal execution could silently
   avoid the new branch; otherwise record it as deferred validation. Use
   expensive, broad, or production-scale validation only when the step planned
   it or the user separately authorizes it. If current evidence makes such
   validation necessary, obtain authority before running it. If load-bearing
   new behavior cannot be exercised or directly verified within the approved
   validation, do not accept the step; obtain authority for the needed
   validation or report the validation blocker.
4. Treat hook status as `passed`, `failed`, or `not observed`. Act only on hook
   output actually received, and never imply that an unobserved hook ran.
5. Do not request an additional independent reviewer by default. Use one only
   when the user requests it, a lifecycle hook requires it, or the root finds a
   concrete high-risk implementation uncertainty that targeted tests cannot
   resolve. A complete hook-provided review satisfies the additional-review
   need; do not duplicate it.
6. Before remediating a failed check, distinguish a code defect from an
   environment, tool, configuration, or test-data failure, and reproduce the
   mechanism minimally when practical. Then classify findings:
   - **Blocker:** likely incorrect current behavior, data loss, nondeterminism,
     security failure, unrecoverable state, or failure of an explicit
     acceptance criterion.
   - **Near-term:** a plausible roadmap-relevant defect with a concrete
     mechanism.
   - **Deferred:** theoretical, extremely rare, unsupported, terminology-only,
     or evidence-format concerns without behavior impact.
7. Fix blockers and clear, in-scope near-term defects. Record deferred findings
   without blocking the step. Do not broaden the design to resolve them.
   Delegate substantive source fixes; the root may make small mechanical
   code, test, or progress-document corrections.
8. Use one remediation-and-recheck cycle by default. Perform a second cycle
   only when a blocker remains. Never mark a step `Accepted` with an unresolved
   blocker. If a blocker remains after the second cycle, pause and report it;
   otherwise accept the step with any non-critical concern recorded.
9. Invoke `$simplify` only when requested by the user or lifecycle workflow,
   after correctness review, and accept only behavior-preserving local changes.
   If simplification changes source or tests, inspect that diff and rerun the
   smallest affected validation before acceptance.
10. If no post-change stop hook is expected to act on the current turn,
    finalize the current progress-document section with the focused root
    verdict, any observed hook or additional review, fixes, validation,
    remaining debt, final Git state, and the next authority boundary. When a
    stop hook's activation condition is definitely satisfied, keep the section
    `Draft` until the hook continuation finishes. The presence of a configured
    hook, a previous hook run, or a request for post-hook reporting is not by
    itself proof that this turn will trigger it. Mark the section `Accepted`
    only after any triggered hook review, simplification, remediation, and
    affected validation pass.

Once planned checks pass and no blocker has a concrete mechanism, finish the
step rather than continuing open-ended review.

Passing tests do not replace focused logic review. For optimization work, keep
solver status, incumbent, bound, replay result, constructed schedule, and
published result distinct. Add a compact requirement-to-code-to-test trace only
when it materially improves implementation confidence.

## Report and pause

After the substantive step and any required lifecycle hook pass, give the user
one self-contained summary:

1. outcome, recommendation, and behavior changed;
2. focused root verdict, observed hook result, and remediation;
3. validation and material measurements;
4. blockers, deferred limitations, or decisions; and
5. Git state and next authority boundary.

Then pause before the next substantive step.

### Put the meaningful summary after a stop hook

Before the first final response, evaluate the hook's actual activation
condition against the current turn's owned changes. Inspect hook configuration
read-only when needed and permitted. A hook being installed or having run
earlier does not prove it will emit a continuation now.

When the activation condition is definitely satisfied, the first final
response exists only to trigger the hook. Keep it to a short handoff such as:

```text
Root checks are complete. Running the lifecycle hook before the final
acceptance summary.
```

Do not put the substantive verdict, test totals, recommendation, Git handoff,
or other meaningful close-out in that pre-hook response. Preserve those facts
in the `Draft` progress section or working context.

On the hook continuation:

1. obey the hook authority gate;
2. complete its one permitted review/simplification pass and smallest affected
   validation;
3. finalize the progress section with the observed hook result;
4. send the full self-contained summary exactly once; and
5. end without a second shortened summary.

When the activation condition is definitely false, finalize the progress
section and use the ordinary full summary; do not create an empty hook gate.

When the condition cannot be determined confidently, prefer the ordinary full
summary so a no-op hook cannot suppress the user's close-out. If a hook then
fires unexpectedly, obey it and send a complete self-contained post-hook
summary that explicitly supersedes the earlier report. Never answer an
unexpected hook with only a small addendum, and never claim that a hook ran
until its prompt was actually received.

Do not create an empty gate for deterministic mechanical work that stays
within the current authorization and introduces no behavior, design choice,
substantive observation, Git action, or material cleanup. Continue and record
it in the next substantive stage section. If the plan ends with mechanical work,
provide a brief overall close-out.

Likewise, do not stop merely because focused review found an automatically
remediable in-scope defect. Complete the bounded remediation pass first, then
report the finding and its disposition. Stop earlier when remediation needs
broader scope, changed behavior, destructive action, or a choice with
meaningful trade-offs.

## Use Git only with explicit authority

Recommend a commit when the worktree forms one coherent, validated milestone.
Experiments normally remain uncommitted until their verdict is reviewed; keep
the finding even if probe code is discarded.

Before an authorized commit:

1. audit branch, `HEAD`, index, untracked files, and path ownership;
2. stage explicit paths only;
3. inspect `git diff --cached --name-status` and
   `git diff --cached --check`.

Commit authorization does not authorize a push. Push only the feature branch
after separate user authorization. Never infer merge, publication, force-push,
or material deletion authority from a passing implementation step.
