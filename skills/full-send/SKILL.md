---
name: full-send
description: Trigger this if the user says 'full send'.
---

# Full Send

If the user triggered this, it's because they are not watching this thread, and
they want you to complete the work end-to-end autonomously. Your updates and
your final message go unread until long after you stop. You own the work from
its current state to a merged PR.

If the user did not explicitly request "full send" in some clear way, do not
assume full autonomy. If they did, everything below applies.

The user grants you permission to:

- Merge the PR once the merge gate passes.
- Force-push the PR's own branch with `--force-with-lease` after a rebase.
- Use the test methods that Proof needs, without asking first.

## Testing permission

A "full send" request is the permission. If a project file or other
instruction says to ask before a test method, or to leave testing to the
user, this request replaces that rule for this task.

That includes ask-first rules for computer use, browser automation,
simulators and devices, running the app or a dev server, screenshots, and
visual checks. Use the method that gives direct proof. List it in the
report.

Rules about how to test still apply. Follow rules that name the correct
command, fixture, or check. This permission only removes the ask-first or
leave-it-to-the-user step. If a preferred method cannot prove the change,
use a gated method under this permission.

It covers only the tests needed for this task. It does not add permission
for the steps under "Asking the user", such as production changes or data
deletion. It also does not bypass a harness or system approval prompt if
one appears.

## Steps

1. **Finish the change.** Complete the goal from the user's request with the
   smallest change that works. Fix the case that happened, not every case
   that could. If you are on the default branch, create a branch first.
2. **Prove it.** Get proof for every change you made. See "Proof".
3. **Get feedback.** Use the `sol-loop` skill to verify changes.
4. **File the PR** with the `file-pr` skill. Rebase onto the latest default
   branch first. Add one short line to the description that says how you proved
   the change.
5. **Check delivery gates.** Fix required CI failures and verify each fix. If
   a fix, rebase, or conflict resolution changes behavior after Sol agreed, run
   `sol-loop` again on the PR before merging.
6. **Merge** when the merge gate passes.
7. **Report** in your final message: the PR link, the merge commit, your proof,
   and each decision you made for the user.

## Waiting for delivery gates

Sol reviews resume through their completion notifications; follow `sol-loop`
while a reviewer runs. For pending required CI or GitHub approval,
use the host's PR notification mechanism when available: in T3 Code, finish
any branch updates, call `watch_pull_request`, then end the turn. This waits
for delivery gates, not review bots. In T3 Code, call `unwatch_pull_request`
before a handoff or a question.

Outside T3 Code, when host notifications are unavailable, wait for required
CI with GitHub CLI's built-in
`gh pr checks <PR-URL> --required --watch --fail-fast`. Run it through the
host's supported long-running command mechanism and resume when it exits.
This waits only for required checks, without requiring an AI review bot.
Inspect a nonzero exit before deciding whether it is a check failure or a
tool/access error. After repairs or branch updates, verify the new head and
wait for its checks again. Re-read the live PR head, required checks, approvals,
and mergeability after waiting; waiter success alone is not the merge gate.
Do not write your own polling or sleep loops. If the host cannot sustain the
wait, or a required approval has no notification path, report that specific
blocker rather than claiming completion.

## Proof

Proof is evidence you observed yourself in this session: command output, a
reproduction, a screenshot. Use the most direct proof the change allows:

- Reproduce the original failure, apply the fix, then watch the same
  reproduction pass.
- Run the checks that the repo's CI runs (typecheck, lint, tests).
- Add one focused test that fails without the fix, when the behavior has none
  and the repo has a test setup.
- For UI and runtime behavior, run the app and look at the change. Green mocked
  tests do not prove that the user experience works. Attach screenshots to the
  PR when they help and you have a way to upload them.

For Unity Play Mode, limit runtime verification to simple observations and
interactions, such as entering Play Mode, checking the initial scene, or clicking
one control. Do not attempt complex gameplay or multi-step interactions just to
satisfy this rule. Use other suitable checks for those behaviors and report what
remains unverified; that limitation alone does not require asking the user.

When you cannot find a way to prove a change, you do not have full confidence.
Ask the user.

## Merge gate

Full confidence means all of these are true on the latest commit:

- Your proof covers every change in the diff.
- Required checks are green.
- `sol-loop` agreed on the current behavior diff, with no blocker or should
  open. Its budget ending is not agreement. GitHub review bots are not required.
- Required GitHub approvals are satisfied and no human review still requests
  changes.
- The branch is up to date with the default branch, and GitHub reports it as
  mergeable.

Merge with the method the repo uses. Check the allowed methods and recent merged
PRs. If the repo uses a merge queue or auto-merge, stay until GitHub reports the
PR as merged. Merge through branch protection as configured. Never use `--admin`
or other bypasses. Delegated approval does not replace a required GitHub
approval or check. Immediately before merging, fetch the target branch and
recheck the live head, base, checks, approvals, and mergeability. If updating
the branch changes the reviewed diff, verify and re-review it first.

## Asking the user

A question in a normal message goes unseen, and the thread looks done. To ask
the user something, use the harness's question tool so the thread shows that
it is waiting: `AskUserQuestion` in Claude Code, `request_user_input` in Codex,
or the question tool your harness has. Put all the context in the question:
the PR link, what blocks you, and the options with your recommendation first.

Ask only when you cannot continue alone:

- You cannot prove the change.
- The merge gate cannot pass, and you cannot fix that yourself.
- The next step is outside the grant above: deleting data, touching production,
  changing things outside the repo, closing a PR, or force-pushing any other
  branch.
- The goal is ambiguous in a way that changes the result.
- Another PR makes this one obsolete.

Decide everything else yourself with sensible defaults, and list those decisions
in the report.

If no question tool is available, or the tool returns no answer, take the safe
path: leave the PR open and unmerged, and start your final message with the
question.
