# tstack — agent-driven review fork

**Instruction sync:** Keep the shared instructions in `AGENTS.md` and `CLAUDE.md` identical. When changing either file, apply the same shared changes to the other file in the same task. Only the final `# Claude Instructions` section in `CLAUDE.md` is exempt; keep Claude-specific rules there and do not copy that section into `AGENTS.md`.

Reusable agent skills for Claude Code and Codex, forked from
[t3dotgg/tstack](https://github.com/t3dotgg/tstack). Some use
[T3 Code](https://github.com/pingdotgg/t3code) features when available and native
agent tools elsewhere, subject to the required model being available.

## Fork differences and upstream updates

This fork uses delegated `sol-loop` review instead of GitHub review-bot
babysitting. It works in repositories with no GitHub AI review integration.
Preserve these differences when incorporating upstream changes:

For every fork-specific change, including small skill edits, update this
section in the same commit with a short note naming the affected skill or file,
what differs from upstream, and why. Keep the notes current as differences
change or are removed; commit messages alone are not sufficient.

- `full-send`, `full-stop`, and `takeover` rely on `sol-loop` for review, as
  upstream `full-send` does before filing, and rerun it when a later CI fix,
  rebase, or conflict resolution changes behavior. The upstream `babysit-pr`
  skill and its `pr-wait` script are removed because the user's repositories
  have no review bots; an earlier fork-only `review-loop` was retired in favor
  of `sol-loop`'s bounded rounds and smallest-change bias. `file-pr` opens a
  ready-for-review PR rather than one meant to trigger bots, and hands off to
  `sol-loop` instead of `babysit-pr` when a review was requested.
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
  its gate passes; `full-stop` does not. A takeover does not
  grant permission to merge or close a PR.
- `council` accepts two different providers instead of requiring three, so it
  works with a two-provider setup. It still defaults to three when available
  and reports unavailable named members rather than silently replacing them.
- `AGENTS.md` and `CLAUDE.md` include PowerShell installation and update commands so Windows
  users can install without a Bash shell. Both files explicitly require installs
  and updates to cover Codex and Claude Code so neither installation is missed.

- `AGENTS.md` and `CLAUDE.md` contain the former README guidance and the shared
  instruction sync rule so both agents load it directly; `README.md` is removed.

Keep `origin` pointing to this fork and `upstream` to `t3dotgg/tstack`. Fetch
upstream and integrate changes on a separate branch. Inspect changes to skill
dependencies, merge/readiness gates, model selection, and waiting behavior;
do not restore the bot requirement or overwrite this section wholesale.
Check all skill references and frontmatter after resolving the update.
Retain upstream attribution and the MIT license.

## Skills

| Skill | Say | What it does |
|---|---|---|
| `full-send` | "full send" | Proves a change, reviews it with `sol-loop`, files its PR, checks delivery gates, and merges. |
| `full-stop` | "full stop" | Same as `full-send`, but stops when the PR is ready for you to merge. |
| `council` | "council" | Gets review from models across at least two providers, and loops until they agree. |
| `sol-loop` | "sol loop" | Reviews a change with GPT Sol in a loop until both agents agree. |
| `file-pr` | "file a PR" | Files a short PR that leads with the problem, not an inventory of changes. |
| `takeover` | "take over" | Finishes someone else's PR, branch, or agent thread, and keeps their credit. |
| `modernize` | "modernize" | Rebases an old PR and decides if it should still exist. |
| `leaving-pr-comments` | (automatic) | Formats comments that an agent posts on your behalf. |

Some skills call others. Install the ones they need too:

- `full-send` uses `sol-loop` and `file-pr`.
- `full-stop` uses `full-send`.
- `takeover` uses `modernize`, `sol-loop`, `file-pr`, and `leaving-pr-comments`.
  It also shares `full-send`'s delivery waiting procedure, without its permissions.
- `file-pr` and `modernize` use `leaving-pr-comments` when posting authorized comments.

## Install

"Install" and "update" mean both Codex and Claude Code unless the user explicitly
limits the request to one. Run the commands for both destinations.

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
Installations of earlier fork versions should likewise remove `review-loop`.

## Requirements

- An authenticated [GitHub CLI](https://cli.github.com) (`gh`).
- Independent agent delegation for `council` and `sol-loop`. T3 Code uses
  `delegate_task`; elsewhere the harness must support the selected reviewer
  models. `council` requires models from at least two different providers.

## Make them yours

These are starting points. Edit them. Put personal settings, like which models
sit on your council, in your `CLAUDE.md` or `AGENTS.md`, so you can update the
skills without losing them.

## License

[MIT](LICENSE)
