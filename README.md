# custom-claude-skills

Custom skills and output styles for Claude Code.

| Skill | What it does |
|---|---|
| [orca-delegate](skills/orca-delegate/SKILL.md) | Runs each task in its own Orca worktree with its own Claude session. Needs the Orca CLI (`orca`). |

| Output style | What it does |
|---|---|
| [ADHD Focus](output-styles/adhd-focus.md) | Action first, short numbered steps, visible progress, one next action. |

## Install

Skills: copy a folder from `skills/` into `~/.claude/skills/` or a project's `.claude/skills/`.

Output styles: copy a file from `output-styles/` into `~/.claude/output-styles/` (or a project's `.claude/output-styles/`), then pick it with `/config`.

## Status

Draft. Not yet reviewed for publication.
