---
name: orca-delegate
description: >
  Run one or more tasks (research, code, or review) in separate Orca worktrees, each with its own
  agent session started via `orca terminal send`, then report the terminal ids and monitor them on
  request. Works for a single task too.
  Triggers on: "/orca-delegate", "for each task create a worktree", "start separate claude
  session per task", "spawn a session per task", "do this in a new worktree",
  "start a claude session for this in orca", "check with orca".
allowed-tools: Bash Read
---

# orca-delegate: One Worktree + One Agent Session per Task

Load the `orca-cli` skill first (`orca skills get orca-cli`) and use `orca` as the executable
unless `ORCA_CLI_COMMAND` is set. Do not guess flags: run `orca <cmd> --help` when unsure.

## Step 1 -- Split the request

One task per numbered item. A single task is valid: run the same steps once, no table needed in Step 5,
just the terminal id. Give each a short kebab-case worktree name (e.g. `payment-api-research`).
Do not ask for confirmation unless the split is truly ambiguous.

## Step 2 -- Create worktrees (all in one call)

```
orca worktree create --name <name> --no-parent --agent claude --json
```

- Use `--no-parent` so the worktrees are independent of the current context.
- Use `--agent claude` unless the user names another agent id (run `orca worktree create --help`).
- Omit `--base-branch` to branch from the repo default. Pass `--base-branch <ref>` only if the user asks.
- Parse the JSON for the terminal handle (`result.agentTerminalHandle`, or `result.startupTerminal.handle` on older runtimes) and the worktree path.
- New worktrees contain committed files only. Uncommitted files in the source checkout are NOT present.
  If a task depends on one, name its path in the brief and say it is uncommitted in the source checkout.

## Step 3 -- Wait for each TUI

```
orca terminal wait --terminal <term_id> --for tui-idle --timeout-ms 90000 --json
```

Proceed only when the JSON shows `"satisfied": true`. On timeout, read the terminal (Step 6) before retrying.

## Step 4 -- Send the briefs

```
orca terminal send --terminal <term_id> --text "<brief>" --enter --wait-submit 10 --json
```

The JSON must show both `accepted` and `turn_started`. If missing, read the terminal (Step 6).

The session has no chat context, so every brief is self-contained. Pick the variant that fits.

**Common to all briefs**

1. `Task:` goal in one sentence, plus the concrete user problem.
2. Scope: files, areas, or questions to cover. Say "do not limit to X" when the user wants breadth.
3. `Verify claims against docs, code or repos; do not guess APIs, paths or flags.`
4. Project conventions: if the project's `CLAUDE.md`, or the user's request, states rules the session must follow (output locations, skills to use, filing steps), restate them in the brief. The new worktree inherits committed `CLAUDE.md` files but not rules from chat.

**Research brief, add**

- Per option: what it does, effort, risk, feasibility for the goal, source link.
- `End with a Sources line.`

**Code brief, add**

- Acceptance criteria and how to verify (test or build command).
- `Work on the current branch. Do not commit or push unless asked.` (or the opposite, if the user asked for commits)

## Step 5 -- Report

Short table: task, worktree/branch, terminal id. Note base commit and any uncommitted-file caveat.
Do not summarize the work; it has not happened yet.

## Step 6 -- Follow-ups (on request)

- Scope addition for a running session: `orca terminal send` to its terminal with `Scope addition from user: ...` plus the same flags as Step 4.
- "Check with orca": `orca terminal read --terminal <term_id> --json` for each terminal (last ~80 lines), then summarize status per task.
- Never send to a terminal whose id you did not create or read in this session.
