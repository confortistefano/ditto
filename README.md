# ditto

A documentation review skill for Codex, Claude Code, and other coding agents. Before committing, pushing, or publishing repository changes through an API, follow [SKILL.md](SKILL.md) to keep documentation consistent with the code being shipped.

## Contents

| File | Purpose |
| --- | --- |
| [SKILL.md](SKILL.md) | Portable review workflow and optional gate integration. |
| README.md | Usage and compatibility notes. |

## Use with your agent

Make the complete `SKILL.md` available through the host's supported skill mechanism, or explicitly ask the agent to read it. Consult your host's documentation for skill discovery and installation; this repository does not provide an installer.

| Environment | How to apply ditto | Enforcement |
| --- | --- | --- |
| Codex | Load it as a skill where supported, or reference it from project instructions the agent reads. | Agent follows the checklist; no blocking hook included. |
| Claude Code | Load it through the host's supported skill mechanism or project instructions. | Optional existing Claude hook; implementation not included. |
| Other coding agents | Supply the Markdown as a skill, project rule, or explicit task instruction supported by the host. | Depends on the host integration. |
| Chat/API without repository tools | Provide code diffs and relevant docs for review. | Review only; cannot verify or publish the remote without tools. |
| GitHub connector/API | Read the current files, review proposed changes, write within authorization, then fetch the result to verify. | Version checks depend on the tools available. |

A model does not install or enforce this workflow by itself. Its surrounding agent software must load the instructions and provide the necessary repository tools.

Example instruction for a project rule file:

```text
Before committing, pushing, or publishing repository changes, read and follow
<path-to-ditto>/SKILL.md. Keep documentation aligned with the changes.
Preserve unrelated edits and report anything you could not verify.
```

Replace the placeholder with the actual path and place this instruction in a file your host is configured to read.

Example prompts:

- "Use ditto before committing these changes."
- "Check whether the docs match this diff."
- "Sync the docs and push the authorized changes."

## Workflow

1. Review the exact changes being committed or published.
2. Inspect relevant project and agent documentation.
3. Correct inaccuracies supported by the changes.
4. Include documentation with the intended changes.
5. Complete an existing review gate if configured.
6. Verify the committed or remote files and report any limits.

The checklist supports staged changes, outgoing commits, first pushes, read-only reviews, and API writes. It avoids assuming a default branch, Claude-specific paths, or shell availability.

## Optional Claude hook

The original skill references a Claude PreToolUse hook at `~/.claude/hooks/ditto.py`.

**That hook is not included.** Neither this Markdown skill nor the README installs automatic command blocking. Only use the following command when that integration already exists:

```bash
python3 ~/.claude/hooks/ditto.py stamp --mode commit --repo /path/to/repository
python3 ~/.claude/hooks/ditto.py stamp --mode push --repo /path/to/repository
```

Codex and other hosts can follow the checklist without that script. Automatic enforcement requires a separate integration appropriate to the host.

## Requirements and limits

Local Git workflows require Git. The optional Claude stamp script requires Python 3. GitHub metadata can be checked using an available connector or GitHub CLI.

Instruction compatibility does not establish tested runtime integration with every agent. No hook, installer, or automatic enforcement implementation is supplied in this repository.
