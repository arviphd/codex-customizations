---
name: peer-review
description: Coordinate a bounded adversarial review in which Codex reviews and makes focused fixes while a chosen Claude, Gemini, or Codex subagent performs review-only checks over a scoped document or repository diff, using direct subagent coordination or a hash-bound Markdown ledger with explicit convergence.
---

# Peer Review

Run one bounded review over a precisely scoped target. Codex reviews and fixes;
one selected Claude, Gemini, or Codex subagent reviews only. Use direct agent
coordination for a Codex subagent and one retained Markdown ledger for an
external peer.

## Roles and invariants

Choose the peer before starting. Use the user's choice; otherwise ask, “Should
Claude, Gemini, or a Codex subagent be the peer reviewer?” The invocation
`$peer-review Codex` selects the Codex subagent mode. Changing the peer during
an active review requires user approval and invalidates approvals.
Treat an unqualified Claude selection as native Claude Code unless the user
explicitly names Antigravity. Both Claude environments use `CLAUDE` in ledger
routing fields.

For native Claude Code, assume by default that it can read the canonical target
and append to the normally placed workspace ledger. Do not ask the user to
confirm access. Stop only if the user reports an access limitation or an actual
read or append failure occurs.

For Gemini or Claude hosted in Antigravity, place the ledger and bytes under
review where both sessions can read them and the peer can append to the ledger.
Use known workspace boundaries without asking the user to confirm access. If a
canonical document target is outside that workspace, create a byte-identical
review snapshot beside the ledger, record both paths, and verify that its
fingerprint matches the canonical target. The peer reviews the snapshot; Codex
continues to own the canonical target. An actual access failure stops that run.

- Codex independently reviews the target, validates peer findings, makes only
  focused in-scope fixes, runs validation, and owns canonical state and
  completion.
- The peer searches for material issues but never edits the target or canonical
  state. An external peer may only append under `Peer submissions`; a Codex
  subagent is read-only.
- In external-peer mode, only one live peer session may hold append authority.
  Start it with the one kickoff prompt, then coordinate every later turn,
  correction, pause, and resume through the ledger. Never ask the user to relay
  follow-up text between Codex and that peer session.

In external-peer mode, “invalidate approvals” means set `Approvals` to `NONE`
and log why. In Codex subagent mode, discard both in-session approvals and
require fresh approvals on the current hash. Invalidate after any target edit
or hash mismatch, finding addition or disposition change, or approved scope or
reviewer change.

## Scope and fingerprint

1. Read repository instructions.
2. Record whether the target is `DOCUMENT` or `CODE_DIFF`, its paths or path
   filters, Git base when applicable, review lenses (such as correctness,
   feasibility, security, tests, performance, and scope control), required
   validation, and explicit non-goals. Ask only about a material unresolved
   choice; never broaden scope without user approval.
3. For a document, read the target and only the evidence needed to verify it.
   For a code diff, default “since last commit” to `HEAD`; inspect status and
   the diff, include relevant untracked files, and preserve unrelated changes.
4. Compute and record a reproducible SHA-256 fingerprint. Hash one document's
   raw bytes; for multiple files or a code diff, hash an unambiguously framed,
   sorted inventory of the Git base and each in-scope path's status, type, mode,
   and content or deletion/symlink target, including in-scope untracked files.
   Exclude the ledger and out-of-scope paths. Both approvals bind to this hash.
   Changing the fingerprint method or scope requires user approval; editing an
   already scoped path does not.
   When using a peer-readable document snapshot, hash the canonical document
   and snapshot raw bytes at creation and before every ingestion. Regenerate
   the snapshot after a canonical target edit; never edit it as a substitute
   for the canonical target.

## Codex subagent mode

When the user selects Codex, use one fresh Codex subagent as the review-only
peer. Follow this section instead of the ledger, bootstrap, marker, polling, and
external-peer turn protocol below. Do not create a handoff document or review
snapshot.

1. Compute the scope and fingerprint above, then use `spawn_agent` with
   `fork_turns="none"` for one subagent. Give it the objective, exact target,
   target type, Git base when applicable, current hash, lenses, validation,
   non-goals, and review-only restriction. Do not include Codex's findings or
   conclusions in the initial assignment. Require the reviewer to read all
   applicable repository instructions before the target, and do not let it
   delegate further.
2. Require an independent search for substantiated material issues and exactly
   one result: `FINDINGS`, with severity, location, evidence, impact, and the
   smallest supported correction for each issue; or `APPROVE`, naming the exact
   target hash and any residual evidence gaps. The subagent must not edit files
   or mutate repository state.
3. While the subagent works, Codex independently reviews the same scope. Use
   `wait_agent` for its response; do not poll files. Recompute the fingerprint
   before using the response. If it changed, do not accept the approval and
   reassign the current hash; retain still-relevant findings only as hypotheses.
4. Validate every finding against the target. Codex alone makes the smallest
   permitted fixes and runs proportionate validation. Track each finding as
   `CONFIRMED`, `NARROWED`, or `REJECTED`, with reasons; confirmed or narrowed
   findings remain open until validation supports `RESOLVED`. Any edit
   invalidates both approvals. Recompute the hash, then use `followup_task` with
   the same subagent for a fresh review of that hash; require it to search
   beyond merely confirming the stated fix.
5. Complete only when Codex and the subagent approve the same freshly computed
   hash, validation is current, and no material finding remains open. Report the
   scope, approved hash, material findings and fixes, validation, and residual
   evidence gaps. Do not create a review coordination artifact.

Material issues affect correctness, feasibility, security, scope,
implementation clarity, or whether tests expose failure. Do not prolong the
review for style preferences, optional extensions, invented requirements, or
out-of-scope edge cases. If the same material disagreement repeats twice
without new evidence, another subagent response would exceed turn 10, or the
subagent fails, stop and report the incomplete review rather than substituting
a self-review or creating a handoff. A response with neither usable findings
nor a hash-bound approval gets one corrective follow-up; a second consecutive
invalid response is a subagent failure.

## Ledger

The remaining sections apply only when Claude or Gemini is the external peer.

For native Claude Code, place `<scope-slug>_Review_Handoff.md` next to a document
target. For a code diff, use the repository's document directory when present,
otherwise its root.

For Gemini or Claude hosted in Antigravity, place the ledger in a peer-readable,
peer-writable shared workspace. Prefer the document target's directory when it
is shared; otherwise use the workspace's document directory when present, then
its root. For a code diff, use the repository's document directory when
present, otherwise its root. Keep any required document snapshot beside the
ledger.

Always exclude the ledger and any snapshot from the target and validation.

Never discard or truncate ledger history or peer submissions. Resume `ACTIVE`
work with `Next: CODEX` normally. Resume an `ACTIVE` peer-owned turn only when
Codex still has the exact pre-handoff ledger bytes needed for the prefix check.
Otherwise do not ingest or complete. If canonical state is recoverable, set
`STOPPED` with `Next: NONE`. If it is not, preserve the file, stop monitoring,
report that it is no longer a trusted ledger, and ignore its routing fields.
Resume `NEEDS_USER` only after the user supplies the requested decision, then
log it, invalidate affected approvals, set `ACTIVE`, and name the next owner in
`Next`. The peer keeps monitoring while status is `NEEDS_USER` and discovers the
resume in the ledger; do not issue a resume prompt. `STOPPED` means the running
peer process must exit and cannot resume through the ledger. If `Next: CODEX`,
continue the Codex turn normally.
A `COMPLETE` ledger is immutable. Retain it and use a UTC-timestamped filename
for any later review.

Use this template and fill every field:

```markdown
# Review Handoff

## Current state

- Status: `ACTIVE`
- Reviewer: `<CLAUDE | GEMINI>`
- Next: `<CODEX | CLAUDE | GEMINI | NONE>`
- Turn: `1`
- Current target SHA-256: `<64 lowercase hex characters>`
- Approvals: `NONE`

## Review scope

- Type: `<DOCUMENT | CODE_DIFF>`
- Target: `<canonical paths or filters; peer-readable snapshot path if used>`
- Git base: `<N/A | revision>`
- Fingerprint method: `<exact command or versioned algorithm>`
- Coordination file: `<this path; excluded from review>`
- Review lenses: `<binding lenses>`
- Validation: `<required checks>`
- Non-goals: `<explicit exclusions>`

## Role contract

- Codex: review, make focused fixes, validate, and own canonical state.
- Peer: adversarial review only; never edit the target or canonical state.

## Current peer assignment

<reviewer-specific assignment>

## Finding ledger

`NONE`

## Review log

### Turn 0 — Codex setup

- Recorded scope, roles, target hash, and reviewer. Peer reviews first.

## Peer submissions

Append submissions below this line. Do not rewrite earlier content.
```

`Approvals` has one canonical scalar form: `NONE`; one
`<CODEX|CLAUDE|GEMINI>:<full hash>` entry; or the selected peer entry followed
by the Codex entry, separated by `, `. Keep at most one entry per owner and
require every entry to name the current hash. Approval log lines are audit
history, not additional entries in this field.

Valid states are `ACTIVE` with `Next` set to `CODEX` or the selected peer;
`NEEDS_USER`, `COMPLETE`, or `STOPPED` with `Next: NONE`. The peer keeps polling
through `NEEDS_USER` and exits on `COMPLETE` or `STOPPED`. Use `STOPPED` only
when the user stops, either active session fails, or Codex cannot safely resume
a peer-owned turn. The non-selected peer never appears in `Next`. Normalize
other combinations when reliable state can be recovered; otherwise set
`NEEDS_USER`. A new review starts `ACTIVE` with `Next` set to the selected peer.

## Peer assignment

Before every peer turn, refresh `Current peer assignment` with the current hash,
scope, lenses, and review-only restriction. For Gemini, keep this pre-read
assignment neutral: do not summarize or quote Codex findings, dispositions,
corrections, conclusions, or test interpretations. Gemini reads those only
after producing its independent risk map and initial attack traces. Require the
submission to name the hash, append only under `Peer submissions`, and leave
canonical state unchanged. Give that turn one exact completion marker and
require the peer to append it as the final non-whitespace line after its
submission. When a hosted-environment review snapshot is used, direct the peer
to it as the exact bytes under review while retaining the canonical path and
canonical hash in the scope; do not ask the user to transfer target content.
Use this marker:

```text
<!-- PEER_SUBMISSION_COMPLETE reviewer=<CLAUDE|GEMINI> turn=<turn> target_sha256=<64 lowercase hex> -->
```

The marker is a write-completion boundary, not an approval.
The peer must not include any other completion-marker line in that turn's
append. When discussing marker attacks, use placeholders such as
`<STALE_MARKER_TURN_1>`; never reproduce a raw line matching the completion
marker schema anywhere except the required final boundary.

For Claude, request an independent search for substantiated material issues and
either findings or approval of the exact hash.

For Gemini, include this directive substantively on every turn:

```text
ADVERSARIAL RED-TEAM / STRESS-TEST REVIEW. Independently try to falsify the
current target; do not merely validate, summarize, or endorse Codex's work. Use
the recorded lenses to generate candidate risks, not as headings for a coverage
checklist. Do not produce one superficial verification per lens.

Use a staged reading order. On turn 1, use the bootstrap's neutral pre-read
packet and do not open the handoff. On later turns, first read only the handoff
prefix ending immediately before the first exact `## Finding ledger` heading
while omitting the `Approvals` line; do not open or search the remainder. For a
local Markdown handoff, use the equivalent of
`awk '/^## Finding ledger$/{exit} !/^- Approvals:/' '<handoff>'`. This exposes
only routing fields, `Review scope`, `Role contract`, and `Current peer
assignment`, not findings, logs, approvals, or prior submissions. Inspect the
target and complete the RISK MAP and initial ATTACK TRACES before reading the
remainder. Preserve that work as the submission's first sections; then read the
full handoff and separately challenge the prior work.

Rank plausible material failures of existing requirements by impact and
plausibility. For a non-trivial target, pursue the two highest independent risks
deeply; pursue fewer only when no others are genuinely plausible. At least one
must start from a concrete adverse outcome and work backward to an unstated
assumption, component interaction, ordering or recovery gap, or negative path.
For a procedure, simulate an end-to-end execution and state transitions; for
code, trace a concrete input and state to the observable result. An explicit
safeguard is not an independent hypothesis unless the attack attempts a bypass
or interaction it does not address. Never invent a risk to meet a count.

For each hypothesis, execute a concrete test artifact using the target type:

- For `DOCUMENT`, if the document specifies a parser, protocol, state machine,
  ordering rule, calculation, or format, run a minimal scratch simulation
  against at least two adversarial payloads or event sequences. For other
  claims, perform a direct comparison against authoritative evidence; do not
  write a script that merely restates the claim.
- For `CODE_DIFF`, prefer a focused existing test; otherwise use a temporary
  harness that invokes the changed behavior with at least two adversarial cases
  derived from its actual contract and callers. Do not add generic null,
  concurrency, malformed-type, or boundary cases unless the code makes them
  relevant. If safe execution is unavailable, trace exact inputs, branches,
  state changes, and outputs and report the execution gap.

Use only operations without external side effects, keep scratch artifacts
outside the target and repository, and do not mutate either. Base conclusions
on observed output or direct source evidence.

Every ATTACK TRACE must use this mechanical execution schema:

1. `Targeted invariant`: quote the exact existing rule or transition at risk.
2. `Synthetic payload / event sequence`: give the literal candidate input,
   command, evidence pair, or interleaved states and values. Represent any
   completion marker with a placeholder such as `<STALE_MARKER_TURN_1>`.
3. `Executed check`: include the exact command or minimal script logic, or name
   the direct evidence comparison, and include its observed result.
4. `Step-by-step rule evaluation`: for each relevant check, show the exact
   value or comparison and its `PASS` or `FAIL` result.
5. `Vulnerability result`: if the bypass succeeds, report a `FINDING` with
   severity and smallest supported fix. If blocked, name the exact comparison,
   token, transition, or evidence that rejected it and any residual gap.

Treat Codex's conclusions as unverified. On later turns, search for an
independent high-risk failure before testing the stated fix and its regressions.
If no independent risk plausibly remains, show how prior attacks covered the
risk classes rather than inventing one. A turn that only confirms the fix is
incomplete.

Stay strictly within the recorded target, lenses, and existing requirements.
Do not invent requirements, optional enhancements, or out-of-scope edge cases
to create findings. Material outside the target may be inspected only as
evidence about the scoped target; do not report an unrelated defect there.
Candidate risks are not findings until supported by evidence.
```

Require exactly one of:

- `FINDINGS`: each material finding includes severity, location, evidence,
  impact, and the smallest correction supported by the evidence. Report a
  material issue even when the correction is uncertain; or
- `APPROVE`: the exact hash, residual evidence gaps, and a concise explanation
  of why none is a material issue. Gemini must also include its completed risk
  map and attack traces.

A Gemini submission is not convergence if it lacks the independent risk map and
mechanically evaluated attack traces, mainly summarizes Codex's work, or on a
later turn only checks the stated fix. A trace that restates a safeguard or
asserts compliance is invalid even when it uses the required headings. When
safe execution is available, approval requires observed output from at least
two concrete adversarial cases. Otherwise require the exact static trace and
record the execution gap as residual evidence. A blocked attack supports
approval only when a literal payload or event sequence is rejected at a named
comparison or transition; a safeguard's text or a green test alone is not
proof. Apply the equivalent direct-evidence standard to non-executable claims.
Handle an invalid submission through the generic ingestion rule below and never
ask the user to relay a correction. Do not require a manufactured finding when
the attacks support approval.

These adversarial-evidence rules gate approval, not useful defect evidence.
Always ingest a substantiated material finding even if the rest of the
submission is not sufficient for convergence, then hand it to Codex. Request a
corrected peer turn only when no finding can be ingested and no approval can be
accepted.

For Gemini, give the user this ready-to-copy bootstrap prompt, filling the
pre-read packet verbatim from the neutral ledger sections, then begin polling
without another confirmation:

```text
Work with Codex as the review-only peer on <target>.

Do not open <handoff> yet. First read the repository instructions and use only
this neutral pre-read packet:

<verbatim Status, Reviewer, Next, Turn, and Current target SHA-256 fields; Review
scope; Role contract; and Current peer assignment; omit approval fields, Finding
ledger, Review log, and Peer submissions>

The scope and role contract are binding. Inspect the full target and needed
evidence, then draft the independent risk map and candidate attack payloads.
Before completing the attack traces or drafting the submission, perform the
target-type execution or direct evidence checks required by `Current peer
assignment`. Create no scratch artifact in the target or repository. Complete
the initial attack traces, then open and read the complete handoff, challenge
the prior work, and append the required submission. Before appending, verify
that the handoff still names <reviewer> in Next and matches the packet's
assignment and target hash. If it does not, do not append or ask the user to
relay the mismatch; return to routing-field polling and follow the refreshed
handoff.

Keep this session active. For later turns, check only `Status`, `Next`, `Turn`,
and `Current target SHA-256` every 30 seconds. When `Next` names you, re-read the
handoff only through the end of `Current peer assignment`, stopping before the
first exact `## Finding ledger` heading and omitting the `Approvals` line; do
not reuse the kickoff packet or read the remainder yet. Then repeat the staged
review with that refreshed assignment. While status is NEEDS_USER or ACTIVE
with another owner in Next, do not review or append; keep polling. Stop on
COMPLETE or STOPPED.
```

For native Claude Code, use this direct bootstrap, filled with the canonical
paths and current assignment:

```text
Work with Codex as the review-only peer on <target>.

Read the repository instructions, full target, and complete handoff at
<handoff>. The scope and role contract are binding. Start because Next names
CLAUDE. Follow the Current peer assignment exactly.

Keep this session active and check the handoff routing fields every 30 seconds
for later turns, including through NEEDS_USER. Stop on COMPLETE or STOPPED.

<current reviewer-specific assignment>
```

Do not add Gemini's staged-reading or attack-trace language to this Claude Code
bootstrap.

For Claude hosted in Antigravity, use the same short Claude directive and
30-second polling, but explain that `CLAUDE` is the ledger routing label rather
than an identity claim and point only to the host-readable target or snapshot
and handoff paths. If the host refuses polling or cannot retain the session,
stop the run rather than asking the user to relay later turns. These hosted
additions do not apply to native Claude Code.

The kickoff paste is the only manual relay. After the peer begins, write every
later assignment, rejection reason, user decision, pause, and resume into the
handoff and let both sessions discover it by polling. Do not display a follow-up
prompt or ask the user to carry text to the peer. If the peer session actually
terminates or loses ledger access, set `STOPPED` with `Next: NONE` and report
that the process cannot continue. Starting a replacement peer is a new kickoff,
not a handoff step, and occurs only if the user explicitly requests it.

## Alternate turns

While `Next` names the peer, poll every 30 seconds for up to 30 minutes. Accept
an append only when the prior ledger is an exact prefix, the appended suffix
contains exactly one completion-marker line, that line is the expected marker
and the final non-whitespace line, and target hashes computed before and after
the ledger read equal the handed-off hash. When a review snapshot is used, its
hash must also equal the handed-off hash at both checks. An unexpected or
additional completion marker makes the completed suffix invalid; retain it and
apply the generic invalid-submission rule rather than waiting for another
append to make the suffix appear valid. Otherwise keep waiting for a partial
append, or treat a changed prefix as a rewrite. Recheck the complete candidate
and target hash immediately before ingestion. Do not infer completion from
headings or verdict words.

On timeout, leave the turn `ACTIVE` and ask only whether to keep waiting or
stop; never ask the user to relay text. Continuing starts another 30-minute
window with the same turn, assignment, and hash. Set `STOPPED` with `Next: NONE`
if the user stops or the peer session has failed.

If the target changes while the peer owns the turn, preserve it, log the
handed-off and observed hashes, invalidate approvals, and set `NEEDS_USER` with
`Next: NONE`. Ask the user how to proceed; do not ingest a submission, edit, or
revert the target.

If the ledger disappears, stop and report it; absence is never completion.

Peer-written canonical fields such as `Status`, `Next`, hashes, or approvals are
never authoritative. On a peer submission:

1. Verify the retained-prefix and completion-marker boundary above, recompute
   the target hash, and apply the target-drift rule before ingestion. Preserve
   the marker with the raw submission in the audit history.
2. Accept substantiated, unambiguous findings in alternate Markdown formats and
   normalize them into stable finding IDs while preserving the raw submission.
   If the peer rewrote the ledger, restore canonical structure from the retained
   state and log the normalization. If recovery is unreliable, preserve the
   file and set `NEEDS_USER` with `Next: NONE`.
3. Accept approval only when it explicitly names the current hash and supplies
   the required review evidence. Record
   `Approval: <peer> — target sha256: <full hash>` in the Review log and
   `Approvals` field.
   If a completed submission has neither, log why and increment `Turn` exactly
   once. After the first consecutive invalid submission for the same reviewer
   and target hash, refresh the assignment and keep `ACTIVE` with `Next` on that
   peer if the applicable turn cap permits. After any later consecutive invalid
   submission, or if the cap would be exceeded, set `NEEDS_USER` with
   `Next: NONE`. Resume only with user approval recorded in the log; every
   further invalid submission returns to `NEEDS_USER`. Reset the count after a
   valid submission or reviewer or target-hash change.
4. Findings invalidate approvals. After successful ingestion, set
   `Next: CODEX`, increment the turn, and atomically update canonical state and
   the log without changing the peer's substantive claims.

When the ledger is `ACTIVE` and `Next` is `CODEX`:

1. Re-read the instructions, scope, ledger, target, and—for code—the current
   status, diff, and relevant untracked files.
2. Independently search for material issues and validate peer findings against
   source evidence. Add Codex findings to the ledger. Record peer findings as
   `CONFIRMED`, `NARROWED`, or `REJECTED`, with reasons.
3. Make the smallest permitted correction for confirmed material findings and
   run proportionate validation. A `CONFIRMED` or `NARROWED` finding stays open
   until Codex records a validation-backed `RESOLVED`; `REJECTED` is closed.
4. Recompute the hash after edits, invalidate approvals when required, and log
   the work. If no material finding remains open, record
   `Approval: CODEX — target sha256: <full hash>` in the log and `Approvals`.
5. If matching peer approval is present, apply the convergence rules below. If
   absent, increment the turn, create a fresh peer assignment, and set `Next`
   to that peer. Update the ledger atomically before handoff; its hash, log, and
   assignment must all describe the same target.

Material issues affect correctness, feasibility, security, scope,
implementation clarity, or whether tests expose failure. Do not prolong review
for style preferences or optional extensions. If the same material disagreement
repeats twice without new evidence, or another peer turn would exceed turn 10,
set `NEEDS_USER` with `Next: NONE`. Resume only with a user-approved bound
recorded in the Review log.

## Convergence and cleanup

Complete only when Codex and the peer approve the same freshly computed hash,
no material finding remains open, finding dispositions are unchanged, and
required validation is current after the last edit. Codex then sets `Approvals`
to the canonical two-entry form without duplicating approval log lines and sets
`COMPLETE` with `Next: NONE`. Peer approval never authorizes edits or completion.

Report the scope, reviewer, approved hash, material findings and fixes, and
validation. Retain the completed handoff as the immutable audit record and
report its path. Use a UTC-timestamped filename for any later review of the same
scope.
