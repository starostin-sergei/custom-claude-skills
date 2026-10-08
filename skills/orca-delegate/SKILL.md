---
name: orca-delegate
description: >
  Run one or more research or work tasks in separate Orca worktrees (branched from main), each with
  its own Claude session started via `orca terminal send`, then report the terminal ids and monitor
  them on request. Works for a single task too.
  Triggers on: "/orca-delegate", "for each task create a worktree", "start separate claude
  session per task", "spawn a session per task", "do this in a new worktree",
  "start a claude session for this in orca", "check with orca".
allowed-tools: Bash Read Skill
---

# orca-delegate: One Worktree + One Claude Session per Task

Load the `orca-cli` skill first (`orca skills get orca-cli`) and use `orca` as the executable
unless `ORCA_CLI_COMMAND` is set. Do not guess flags: run `orca <cmd> --help` when unsure.

## Step 1 -- Split the request

One task per numbered item. A single task is valid: run the same steps once, no table needed in Step 5,
just the terminal id. Give each a short kebab-case worktree name (e.g. `steam-overlay-research`).
Do not ask for confirmation unless the split is truly ambiguous.

## Step 2 -- Create worktrees (all in one call)

```
orca worktree create --name <name> --no-parent --agent claude --json
```

- Always `--no-parent` and `--agent claude`.
- Parse the JSON for the terminal id (`term_...`) and worktree path.
- Worktrees branch from `main`. Uncommitted files in the main checkout are NOT present there.
  If a task depends on one, name its path in the brief and say it is uncommitted in the main checkout.

## Step 3 -- Wait for each TUI

```
orca terminal wait --terminal <term_id> --for tui-idle --timeout-ms 90000 --json | grep -E '"satisfied"'
```

Proceed only when `satisfied` is true.

## Step 4 -- Send the briefs

```
orca terminal send --terminal <term_id> --text "<brief>" --enter --wait-submit 10 --json | grep -E 'accepted|turn_started'
```

Brief template (self-contained, the session has no chat context):

1. `Task:` goal in one sentence, plus the concrete user problem.
2. Coverage list: concrete areas to examine, "do not limit to X" when the user wants breadth.
3. Per option: what it does, effort, risk, feasibility for the goal, source link.
4. `Verify claims against docs or repos; do not guess APIs, paths or flags.`
5. Research tasks: `Follow the wiki-research skill: check the vault first, present findings, wait for my confirmation, and only then file via wiki-ingest.`
6. `Any HTML concept mockups go in G:\Projects\ClaudeWikiHub\Concepts\<topic>\ per CLAUDE.md.`
7. `End with a Sources line.`

Both `accepted` and `turn_started` must appear per terminal. If missing, read the terminal (Step 6).

## Step 5 -- Report

Short table: task, worktree/branch, terminal id. Note base commit and any uncommitted-file caveat.
Do not summarize the research; it has not happened yet.

## Step 6 -- Follow-ups (on request)

- Scope addition for a running session: `orca terminal send` to its terminal with `Scope addition from user: ...` plus the same flags as Step 4.
- "Check with orca": `orca terminal read --terminal <term_id> --json | tail -80` for each terminal, then summarize status per task.
- Never send to a terminal whose id you did not create or read in this session.
