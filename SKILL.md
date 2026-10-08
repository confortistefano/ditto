---
name: ditto
description: Use before every git commit or git push — the ditto PreToolUse hook blocks both until this skill has run. Makes sure all documentation, local and on the remote, matches the code being committed or pushed. Also use when the user says "ditto", "are the docs up to date?", "sync the docs".
---

# ditto — docs always match the code

Its only job: at every commit and push, the documentation (local and remote) describes the code as it actually is.

The hook (`~/.claude/hooks/ditto.py`) denies `git commit` / `git push` and tells you `mode` and `repo`. Run this checklist, write the stamp, retry the exact same command.

## 1. Scope the change

| mode | What to diff |
|------|--------------|
| `commit` | `git -C <repo> status --short` + `git -C <repo> diff HEAD` + untracked files. That is what will be committed. |
| `push` | `git -C <repo> log --oneline @{u}..HEAD` + `git diff @{u}...HEAD`. No upstream → diff against `origin/<default branch>`; no remote → treat as commit mode on the last commit. |
| `commit` + "command also pushes" | Both of the above: the commit-mode diff plus any commits already waiting to be pushed. |

If the diff touches only docs, or is purely cosmetic (formatting, typo, comment), say "no doc impact", then go to step 5.

## 2. Inventory the docs

- In the repo: `README.md`, `CLAUDE.md`, `CHANGELOG*`, `docs/**`, any other top-level `*.md`, `.env.example`, usage/help text in the CLI, module docstrings for entry points.
- Outside the repo: `~/projects/CLAUDE.md` → "Project index" table, **only** if a project was added, renamed, moved or its purpose changed.
- Not docs: auto-memory files and `~/.claude/**`. Don't touch them here.

## 3. Find what the change made wrong or missing

For each change, check what the docs claim:
- new/renamed/removed script, command, flag, endpoint, env var, config key, file path → `grep -rn` the old and new names across the docs
- changed setup, deploy, schedule, dependencies, auth or data source
- a new feature with no mention anywhere
- statements that are now false (numbers, defaults, step order)

## 4. Update

- Keep edits minimal and match the existing style and language. Write in English.
- Never invent behaviour: document only what the diff shows.
- Repo has no README and the change is material → propose one and ask before creating it.
- Tell the user in a short table which files you changed and why (or "docs already accurate").

**Commit mode:** add the doc files you changed to the `git add` of the retried command. Name them explicitly and never use `git add -A`, so the docs ship in the **same** commit.
**Push mode:** if docs changed, commit them (explicit paths, message `docs: sync documentation with <short range>`, standard co-author trailer) so they ride in the same push.

## 5. Stamp, then retry

Stamp only after the **last** file edit. Any later edit invalidates the stamp and the hook will block again:

```bash
python3 ~/.claude/hooks/ditto.py stamp --mode <commit|push> --repo <repo>
```

Push mode after a docs commit: that commit moved HEAD, so stamp **after** committing.

Then retry the original command (plus the doc paths in commit mode).

## 6. Verify the remote (push mode, or commit+push)

After the push succeeds:
- `git -C <repo> status -sb`: the branch must not be `ahead`
- `git -C <repo> diff @{u} -- <doc files>`: must be empty
- `git -C <repo> status --short -- <doc files>`: no uncommitted doc changes left behind
- GitHub remote: `gh repo view --json description,homepageUrl`. If the description contradicts the README, **report it and ask** before running `gh repo edit` (shared state).

End with a one-line verdict: `ditto: docs in sync (local + <remote>)` or what is still pending.

## Rules

- Never bypass the hook on your own. `DITTO_SKIP=1` only when the user explicitly asks (e.g. WIP commits).
- Never stage secrets (`.env`, `credentials.json`, `token.json`, `hubspot.config.yml`).
- A "no doc impact" result is fine. Still stamp, and say so.
