---
name: run-workstream
description: Use when picking up, resuming or checking on a project that has been split into workstreams, to see where each one stands, what is blocking it, and how to start the next one.
---

# Run a workstream

Read a workstream plan and tell the user where it stands and what to start next. This skill reads and reports. It doesn't start agents, create worktrees, create tickets or change the plan.

## Find the plan

Look for folders under `<repo root>/.superpowers/` that hold a `prompts/` directory and a `common.md`. If you're in a worktree, `git rev-parse --path-format=absolute --git-common-dir` gives the main checkout, which is where the folder lives.

One match, use it. Several, list them with the date of each one's last log entry and ask which. None, say so plainly: this reads a plan that the `planning-workstreams` skill produced, and this repository hasn't got one.

## Work out where everything stands

Don't ask the user any of this. It's on disk or one command away. Per workstream, cheapest source first:

- **What the agents reported.** `grep -n '· status ·' log.md`, which the log protocol requires them to write at spec approval, at ticket creation and at merge.
- **Its branch.** Does it exist, is it merged, how far ahead. `git branch -a --contains`, `git log origin/main..<branch> --oneline | wc -l`.
- **Its pull request**, where the host's CLI is available and `common.md` names the host. Read the state and the review state, not just whether one exists.
- **Its ticket**, where `common.md` says tickets exist. Its status, not its description.
- **Whether it can start.** Every prompt's dependencies section carries its own check commands, written as literal `grep` and `git` invocations. Run them. That is the difference between "nobody has picked this up" and "nobody can pick this up yet", and it's the thing the user can't see by looking.

Where two sources disagree, say so rather than picking one. A merged branch with no `status · merged` entry means somebody skipped the log, and the next agent reading the log will get it wrong too.

## What to show

A table, a row per workstream, in phase order: number and name, what it delivers in a few words, state, what it's waiting on, and the links.

| State | Read from |
|---|---|
| Not started | No status entries, no branch |
| Blocked | A readiness check in its prompt fails. Name the check and what it wanted |
| Ready | Readiness checks pass, nothing started |
| Specifying | A spec exists at the path `common.md` gives, no approval entry |
| Building | Spec approved in the log, branch exists, not merged |
| In review | Pull request open |
| Merged | Merged, and say whether the log agrees |

Then three lines underneath: what can start right now, what's waiting and on what, and anything the plan expected that hasn't happened, such as a phase boundary nobody reconciled or a contract a prompt says it consumes that no log entry has produced.

## Starting one

The user picks. Give them one copyable block and nothing else in it: the worktree command from `common.md` with this workstream's values filled in, the absolute path of its prompt, and the line to paste into the new session.

If a readiness check failed, say that before the block, not after. Giving someone the command and mentioning underneath that it can't run yet is how they end up halfway in.

Don't create the worktree, don't start the session, don't touch the plan. Those are the user's to do, and the agent that picks the prompt up does the rest.

## When the plan has moved on

If the README names a reconciler and a phase has finished since the last reconciliation entry in the log, say so. That check is the cheapest moment to catch two workstreams that decided something differently, and it's the one that gets skipped, because nothing prompts for it.
