---
name: review-loop
description: Review and repair a pull request with an independent delegated reviewer until no findings remain. Use when the user requests a PR review loop or a workflow requires one.
---

# PR Review Loop

Drive review directly through a delegated agent. GitHub review bots are not a
prerequisite. Repository instructions own scope, review criteria, testing,
repair procedures, and delivery conventions; read the applicable guidance
before starting. Do not import another project's rules.

## Establish scope

Resolve the PR from the request or current branch. Record its URL, checkout,
base/head SHAs, intended scope, and verification evidence. Check
`git status --short`; preserve unrelated work and ensure the review covers the
actual PR head. Register the PR with the host when supported. Creating a PR
belongs to the calling workflow or a separate user request.

## Delegate review

Use the reviewer/model specified by the user or repository. Otherwise use
the latest available GPT Sol at medium reasoning effort. In T3 Code, call
`orchestrator_capabilities` once to resolve live provider/model IDs and
supported settings. Report an unavailable requested model rather than silently
substituting one.

Start each round as a new `delegate_task` with `role: "review"`,
`mode: "async"`, `isolation: "shared"`, the branch-owning checkout, and a new
`clientRequestId`. Put supported model settings in `target.options`. Leave
`runtimeMode` unset; constrain the reviewer in the brief instead. Follow the
task's returned `next` instructions and end the turn when it will wake the
parent on completion. Do not poll, start a watcher for the delegate, or use
`t3_thread_send` to start a re-review.

Outside T3 Code, use a native independent reviewer with the selected model
and a fresh session for each round. If no supported delegation route exists,
report the blocker; self-review is not independent approval.

Give the reviewer a self-contained brief:

- Checkout, PR URL, base/head SHAs, user intent and scope, applicable repository
  instructions and review skills, and observed verification results or gaps.
- Review only: no edits, tests or app launches, git writes, PR/ticket changes,
  watchers, or subagents. Read source and existing evidence independently.
- First round: review the full PR, discover findings independently, then
  reconcile any known unresolved findings. Earlier findings are context,
  not the discovery checklist.
- Return actionable findings with stable IDs, severity, file/line, evidence,
  and consequence, plus a verdict tied to the reviewed head SHA. Use the
  repository's finding contract when present. Otherwise distinguish real bugs
  or requirement violations from optional suggestions.

Do not edit the scoped tree while review runs. A timeout, failed task, missing
evidence, or absent verdict is not approval. Cancel a task before replacing it.

## Claim-check and repair

Check each finding against the source, intended behavior, and repository
rules. Fix confirmed findings using the repository's repair procedure when
present. Keep the change within the user's goal; return disputed findings
with concrete evidence instead of silently dropping them or fixing them merely
to satisfy the reviewer. Optional suggestions do not expand authorized scope.

Maintain a short ledger of every finding and its disposition: repaired,
disputed with evidence, or explicitly deferred with the reason and authority.
Run the required focused verification, then commit and push repairs to the
PR branch under repository conventions. A review-loop request authorizes
these repair updates, not merging or unrelated external actions. Use
`leaving-pr-comments` if the user requested a GitHub reply; the loop does not
require posting reviewer exchanges to GitHub.

## Re-review and completion

Start a fresh delegated task with the original brief, latest pushed SHA,
prior findings and dispositions, repair commits, and verification evidence.
Require it to account for every prior finding and inspect affected changes
and their consequences in the full PR. A newly requested fresh loop starts
with independent full-PR discovery again.

Repeat until the reviewer reports no remaining findings on the current pushed
head and all prior findings are accounted for. Outstanding disputes, missing
evidence, and deferred findings must be reported explicitly; do not describe
them as a clean verdict. Stop for a required user decision or a dispute that
cannot be resolved with new evidence rather than looping indefinitely.

Report the PR, reviewed head, reviewer/model, rounds, repairs, verification,
and verdict. Do not merge. CI and required GitHub approvals remain separate
delivery gates for the calling workflow. Do not wait for a review bot to run.
