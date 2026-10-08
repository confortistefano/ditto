# ditto

A documentation synchronization skill for AI coding agents. Before a commit or push, it checks whether the documentation describes the code being shipped and updates any inaccurate or missing details.

## Repository contents

| File | Purpose |
| --- | --- |
| [SKILL.md](SKILL.md) | The documentation review checklist, commit/push workflow, and verification rules. |
| README.md | Overview, usage, and integration requirements. |

## Workflow

1. Scope the changes: staged and unstaged changes plus untracked files for a commit; outgoing commits and their diff for a push.
2. Inventory relevant documentation, including the README, project instructions, changelogs, setup examples, and CLI help.
3. Identify statements made inaccurate or incomplete by the changes.
4. Update documentation minimally, in English, based on actual behavior.
5. When the hook is installed, stamp the completed review after the last edit and retry the commit or push.
6. After a push, verify branch synchronization and confirm documentation matches the remote.

Documentation-only or cosmetic changes can be marked `no doc impact`; the hooked workflow still requires a stamp.

See [SKILL.md](SKILL.md) for the full procedure and exceptions.

## Usage

Make `SKILL.md` available to your agent through its supported skill mechanism, or explicitly ask it to follow the checklist before committing or pushing.

Example prompts:

- "Use ditto before committing these changes."
- "Are the docs up to date?"
- "Sync the docs before pushing."

## Hook integration and prerequisites

The skill describes a Claude PreToolUse hook at `~/.claude/hooks/ditto.py` that blocks `git commit` and `git push` until a documentation review is stamped.

**The hook implementation is not included in this repository.** The skill file alone does not install a hook, enforce blocking, or provide the stamp command. Without that separate integration, the checklist can be followed manually.

With the hook installed, the documented stamp commands are:

```bash
python3 ~/.claude/hooks/ditto.py stamp --mode commit --repo /path/to/repository
python3 ~/.claude/hooks/ditto.py stamp --mode push --repo /path/to/repository
```

The workflow uses Git, Python 3 for the hook, and GitHub CLI (`gh`) for checking GitHub repository metadata. Upstream comparison requires an upstream branch or the fallback described in the skill.

## Review rules

- Document only behavior supported by the changes.
- Stage changed documentation by explicit path so it ships with the relevant code.
- Never stage secrets.
- Do not bypass the hook unless the user explicitly requests it.
- Report contradictory GitHub repository metadata and ask before changing it.
