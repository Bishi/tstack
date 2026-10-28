# tstack — agent-driven review fork

Reusable agent skills for Claude Code and Codex, forked from
[t3dotgg/tstack](https://github.com/t3dotgg/tstack). Some use
[T3 Code](https://github.com/pingdotgg/t3code) features when available and native
agent tools elsewhere, subject to the required model being available.

## Fork differences and upstream updates

This fork uses independent delegated PR review instead of GitHub review-bot
babysitting. It works in repositories with no GitHub AI review integration.
Preserve these differences when incorporating upstream changes:

For every fork-specific change, including small skill edits, update this
section in the same commit with a short note naming the affected skill or file,
what differs from upstream, and why. Keep the notes current as differences
change or are removed; commit messages alone are not sufficient.

- `review-loop` delegates review, claim-checks findings, repairs confirmed
  issues, verifies and pushes repairs, then starts a fresh reviewer task each
  round until the current head has no remaining findings. It reports unresolved
  disputes or blockers without claiming approval and never merges itself.
- `full-send`, `full-stop`, and `takeover` use that loop after PR creation or
  update. The upstream `babysit-pr` skill and its `pr-wait` script are removed.
  `sol-loop` remains an optional standalone local review workflow.
- No workflow requires a GitHub review bot to run. Required CI checks and
  GitHub approvals still apply; delegated review never bypasses protection.
  Host PR notifications may wait for these delivery gates, separately from
  delegated reviewer completion notifications. Outside T3 Code, hosts without
  PR notifications use GitHub CLI's built-in required-checks wait; no custom
  bot watcher is needed. Final readiness must cover the current PR head,
  including repairs made after the first review.
- Each repository supplies its own scope, review criteria, verification,
  repair rules, and delivery conventions. These skills carry no Factory
  Survival dependency or project-specific Unity, ticket, or test requirements.
- `full-send`, `full-stop`, and `takeover` limit Unity Play Mode verification
  to simple observations and interactions, rather than requiring complex
  gameplay to satisfy runtime proof. Use other suitable checks and report
  remaining gaps so verification does not turn into lengthy Play Mode actions.
- Keep the original workflow permissions: `full-send` permits merging after
  its gate passes; `full-stop` and `review-loop` do not. A takeover does not
  grant permission to merge or close a PR.
- `council` accepts two different providers instead of requiring three, so it
  works with a two-provider setup. It still defaults to three when available
  and reports unavailable named members rather than silently replacing them.
- This README adds PowerShell installation and update commands so Windows
  users can install without a Bash shell.

Keep `origin` pointing to this fork and `upstream` to `t3dotgg/tstack`. Fetch
upstream and integrate changes on a separate branch. Inspect changes to skill
dependencies, merge/readiness gates, model selection, and waiting behavior;
do not restore the bot requirement or overwrite this section wholesale.
Check all skill references and frontmatter after resolving the update.
Retain upstream attribution and the MIT license.

## Skills

| Skill | Say | What it does |
|---|---|---|
| `full-send` | "full send" | Proves a change, files its PR, runs delegated review and repairs, checks delivery gates, and merges. |
| `full-stop` | "full stop" | Same as `full-send`, but stops when the PR is ready for you to merge. |
| `council` | "council" | Gets review from models across at least two providers, and loops until they agree. |
| `sol-loop` | "sol loop" | Reviews a change with GPT Sol in a loop until both agents agree. |
| `file-pr` | "file a PR" | Files a short PR that leads with the problem, not an inventory of changes. |
| `takeover` | "take over" | Finishes someone else's PR, branch, or agent thread, and keeps their credit. |
| `review-loop` | "review loop" | Delegates independent PR review and repeats verified repairs and fresh reviews until no findings remain. |
| `modernize` | "modernize" | Rebases an old PR and decides if it should still exist. |
| `leaving-pr-comments` | (automatic) | Formats comments that an agent posts on your behalf. |

Some skills call others. Install the ones they need too:

- `full-send` uses `file-pr` and `review-loop`.
- `full-stop` uses `full-send`.
- `takeover` uses `modernize`, `file-pr`, `review-loop`, and `leaving-pr-comments`.
  It also shares `full-send`'s delivery waiting procedure, without its permissions.
- `file-pr`, `review-loop`, and `modernize` use `leaving-pr-comments` when posting authorized comments.

## Install

macOS/Linux (Bash):

```bash
git clone https://github.com/Bishi/tstack
cp -R tstack/skills/* ~/.claude/skills/   # Claude Code
cp -R tstack/skills/* ~/.codex/skills/    # Codex
```

Windows (PowerShell), using the shared user skills folder loaded by this
T3 Codex setup:

```powershell
git clone https://github.com/Bishi/tstack "$env:USERPROFILE\tstack"
if ($LASTEXITCODE -ne 0) { throw 'Clone failed.' }
New-Item -ItemType Directory -Force "$env:USERPROFILE\.agents\skills" | Out-Null
Copy-Item "$env:USERPROFILE\tstack\skills\*" "$env:USERPROFILE\.agents\skills" -Recurse -Force
```

To update that PowerShell installation:

```powershell
git -C "$env:USERPROFILE\tstack" pull --ff-only
if ($LASTEXITCODE -ne 0) { throw 'Update failed; installed skills were not changed.' }
Copy-Item "$env:USERPROFILE\tstack\skills\*" "$env:USERPROFILE\.agents\skills" -Recurse -Force
```

If your clone is elsewhere, replace `$env:USERPROFILE\tstack` with its path.
For Claude Code, use `$env:USERPROFILE\.claude\skills` as the destination in
both the directory creation and copy commands. Copying overwrites matching
skill files but preserves unrelated skills; it does not remove skills deleted
from the repository. Updated skills are available on the next turn.

If updating an installation of upstream tstack, remove its installed
`babysit-pr` folder too; copying the fork's skills does not delete old skills.

## Requirements

- An authenticated [GitHub CLI](https://cli.github.com) (`gh`).
- Independent agent delegation for `review-loop`, `council`, and `sol-loop`.
  T3 Code uses `delegate_task`; elsewhere the harness must support the selected
  reviewer models. `review-loop` follows user/repository model preferences and
  otherwise uses the latest available GPT Sol at medium effort. `council`
  requires models from at least two different providers.

## Make them yours

These are starting points. Edit them. Put personal settings, like which models
sit on your council, in your `CLAUDE.md` or `AGENTS.md`, so you can update the
skills without losing them.

## License

[MIT](LICENSE)
