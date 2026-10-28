---
name: takeover
description: Take over existing work. Use when the user asks to take over a PR, branch, or agent thread, or requests a takeover.
---

# Takeover

Take responsibility for finishing the original goal while preserving the
original contributors' credit. Do not defend an implementation just because
another agent already built it.

Read the source PR, branch, or thread, including review findings and the user's
later corrections. Resolve the handoff to the actual code and current head,
and account for uncommitted work. Treat the handoff as context, not proof.

For a replacement PR, import the original commits onto a new branch and add
your fixes separately. Honor an explicitly requested in-place or stacked
continuation instead. Do not overwrite unrelated work or change the original
PR without authorization.

Preserve Git authorship and existing co-author trailers. Credit the original
contributors in the PR description and link the original PR when there is one.
If the work will be squashed, preserve human author/co-author credit in the
merge commit too, using verified identities rather than guessed emails. Model
attribution is not a substitute for contributor credit.

Use `modernize` to update the continuation against its intended base and
current architecture. Rework what is necessary to finish the same goal, not
unrelated features. Simplify the implementation without dropping intended
behavior or surfaces. Recheck the source PR too: it may have improved while
you were working.

Independently verify the original failure and realistic workflows, following
repository guidance and permissions. Green mocked tests do not prove the user
experience works. State what remains unverified.

For Unity Play Mode, limit runtime verification to simple observations and
interactions, such as entering Play Mode, checking the initial scene, or clicking
one control. Do not attempt complex gameplay or multi-step interactions just to
satisfy this rule. Use other suitable checks for those behaviors and report what
remains unverified.

When the requested finish includes a PR, use `file-pr` to file or update it,
then run `review-loop` to review and repair the current PR head. Check the
repository's required CI and approval gates before declaring it ready;
delegated review does not replace branch protection. If required gates are
pending and the requested delivery depends on them, follow only the
[delivery waiting procedure](../full-send/SKILL.md#waiting-for-delivery-gates);
this does not grant full-send's merge or testing permissions. Do not wait for
GitHub review bots. After CI repairs, conflict resolution, or any branch
update, verify the changes and rerun `review-loop` if the clean verdict no
longer covers the current head. Before declaring readiness, re-read the live
PR and require its head to match the clean delegated verdict as well as
satisfying the repository's delivery gates. Use `leaving-pr-comments` for
authorized replies. Merge or close either PR only when authorized.

Report what you inherited, what you changed, how you verified it, how credit
was preserved, and any remaining decision.
