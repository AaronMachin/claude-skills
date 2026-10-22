---
name: planning-workstreams
description: Use when a piece of work is too large for one agent session and has to be split up for several agents, running at the same time or one after another.
---

# Planning workstreams

Split a large piece of work into workstreams and write the prompt that starts each one.

Each prompt hands its agent a piece of the problem to work out, not a list of tasks to do: it runs its own brainstorm with the user, writes its own spec and its own plan for its slice. You're doing that one level up, for the split itself.

Requires the superpowers plugin. Use `superpowers:brainstorming` to work the split out with the user. The agents you write prompts for use superpowers too, so `common.md` names the skills each one follows and when.

You're planning. You're not building anything and you're not writing specs.

## Before you write anything

One question at a time, and only about what the user hasn't already told you. Settle at least:

- **Where each agent stops.** Brainstorm and spec, spec and plan, or built and merged.
- **How the workstreams run.** Whether each agent gets its own git worktree so several can run at once, or they share one checkout and go one after another. A branch per workstream off the main branch, branches stacked where one depends on another, or commits straight onto the main branch. A pull request each, one at the end, or none. How branches are named. And if several run at once they share more than files: settle how each one gets its own test database, ports and container names, and write the command into `common.md`. Where running the tests takes a wrapper, write one runner script per workstream into the shared folder, plus a template the next one copies, rather than leaving every agent to work the invocation out again.
- **Tickets.** Whether tickets already exist for this work or the agents create their own, and separately whether there's already an epic or parent ticket covering the whole project or the user wants one created from it. If the workstream tickets exist, get the key for each one and for the parent, and check whether any ticket has no workstream covering it. If they don't, get the tracker and project to create them in, the issue type, and whatever the team expects every ticket to carry: the summary shape, labels, components, estimate, acceptance criteria. If there's no tracker at all, say that in `common.md` so no agent invents one.
- **What's already decided** and must not be reopened, against what's still open for the agents to settle with the user.
- **Designs, specs or documents** the agents must read. Get the order that settles it when two of them disagree, as an ordered list, and write that list into `common.md`.
- **Where two workstreams meet**, settle the shape between them now rather than leaving it for later. A sketched endpoint or signature the user has approved lets both sides start today; "read the log for it when it lands" turns an agreement into a queue.
- **Which repositories and directories** are in scope. Where there's more than one, settle which repo hosts the shared folder, and get the main checkout path, the worktree naming and the test command for each, because `common.md` needs a row per repository.

Work out the "read first" list from the repository itself rather than assuming: its CLAUDE.md or AGENTS.md, any rules index those point at, how tests are run, how code is formatted, and anything CI enforces on new files. Name the measures it already has and the bar each one sets, because that's what the agents have to clear. Where something worth measuring isn't measured at all, say what you'd add and why it matters in one line, and offer it as its own workstream for the user to take or drop. Never bolt new tooling onto a workstream that's there to do something else.

Research runs while you brainstorm, through subagents rather than in your own context. One narrow question each (on Claude Code, Explore or general-purpose), and ask for a short answer with `file:line` references rather than file contents. It's how you find out what the workstreams are, so it can't wait until after the map. Hold the findings in your replies until the map is approved, then write them to `background.md` so no agent reads the same code twice.

Before you show the map, walk the source material item by item - the designs, the tickets, the document the work comes from - and name the workstream that covers each one. Anything with no workstream against it is either scope you're dropping, which you say so in the README, or a workstream you haven't written yet. Then go the other way: anything in a workstream that traces back to nothing is scope someone invented.

Then draw the dependency grid: workstreams down the side, the same list across the top, a mark in each cell where the row needs something from the column: `D:` where the row needs a shape or a decision agreed before it can specify its own work, `B:` where it needs merged code before it can build. Then a word saying which shape, which decision, which code. A row with no marks can start now. A pair marked both ways isn't two workstreams, it's one you've cut down the middle.

If a cell holds something nobody knows the answer to, and the plan can't be settled without it, that's a spike rather than a workstream. Give it a prompt of its own named `spike-<name>.md` with no phase number, run it before the rest, and have its findings land in `decisions.md` rather than the log, because everything after it gets built on them.

Show the grid and the phase table together and stop for the user's approval. Write no files until they've approved it. The phase table has two order columns, not one: when a workstream can start being specified, and when it can start being built. They're usually different, and one column can't hold both. Say which kind each dependency is, because a design dependency is satisfied by an agreed shape and a build dependency only by merged code.

If the user wants an epic or parent ticket created, create it yourself once they've approved the map, holding the project description and the list of workstreams, so every prompt can name its parent. Show them the summary and description before you create anything.

## Sizing

A workstream is one agent's job from start to finish: one spec, one branch, one review.

Prefer a workstream that cuts through every layer of one capability to one that owns a layer across several. A backend workstream and a console workstream for the same feature means neither can be shown working, each waits on the other's shape, and the join lands at the end. Split by layer only where the layer ships on its own, like a migration or a CI job.

Split it if it has more than about five open questions to settle with the user, if it delivers more than one thing a person using the product would notice separately, or if its parts could be reviewed independently. Join two if they would change the same file for the same reason. Check the line you wrote for it: if it needs an "and" or a comma list, that's the one to split.

The line says what someone can do afterwards that they couldn't before, and it comes with how you'd know it's true. "Permission check on every API route" is a layer. "A Site manager opening the Partners tab is refused, and a test fails on any route that names no permission" is a deliverable you can check.

## The shared folder

All of it goes in one folder every agent reaches by the same absolute path, whatever checkout or worktree it is in. If you're planning from a worktree, the repository root is the wrong place: `git rev-parse --path-format=absolute --git-common-dir` gives the main checkout's `.git`, so the folder goes beside that, at `.superpowers/<slug>/`. Write that absolute path into every file and into every prompt's first line. Add `.superpowers/` to `.gitignore` if it isn't there, and tell the user the folder is on this machine only, so a fresh clone won't have it.

| File | What's in it |
|---|---|
| `README.md` | What the project is, the outcome the workstreams exist to deliver written so you can tell whether it's been met, the dependency grid, the phase table, who owns each shared file, the merge order, and the log protocol |
| `common.md` | Everything every agent does the same way, as headings with answers under them rather than topics |
| `decisions.md` | What's settled before anyone starts. One writer, the person who planned this. Agents build on it and don't edit it |
| `background.md` | Your subagents' findings, with `file:line` references |
| `log.md` | Append-only. Every agent writes decisions, contracts, status and challenges here as they happen |
| `deviations.md` | Only where the work is finishing a design or a decision record someone else signed off: each departure as original, decided and why, for handing back to them |
| `wt-<name>.sh` | Where tests need a wrapper: one runner per workstream, each with its own database and worktree path, plus a template to copy |
| `prompts/<phase><letter>-<name>.md` | One per workstream |

`common.md` needs answers, not pointers: what to read first and in what order; a row per repository giving its main checkout, where the agent's worktree goes and the command that runs its tests; keep your own context light and do further reading through subagents; which superpowers skills it follows and when, across the whole job, which is `superpowers:brainstorming` before its spec, `superpowers:writing-plans` for the plan, whose task template reaches only the tasks in that plan, so say that tasks depending on another workstream are marked with its number and that the plan says which groups run in parallel; `superpowers:using-git-worktrees` where several run at once, and `superpowers:subagent-driven-development`, `superpowers:test-driven-development` and `superpowers:verification-before-completion` where it builds; what the agent produces, with the exact path and filename pattern for each so every workstream's output sits together; what the finished thing must contain, including tests and anything CI enforces on new files; which source wins when two disagree; branches, commits, reviews and pull requests; tickets, including that an agent creates its own only once the user has approved its spec, shows the summary and description first, checks for an existing ticket rather than making a second one, and logs the key; what to do about a decision it can't build as written; when and how to write to the log; and the writing style.

A workstream that hits a decision in `decisions.md` it can't build as written stops, logs a `challenge` entry naming the decision and what it hit, and asks the user. It doesn't work around it quietly.

`log.md` is what stops parallel agents contradicting each other, so `common.md` gives the format, not just the habit. Every entry starts `## <date> <time> · <workstream number and name> · <decision|contract|status|question|challenge>` and ends with `Affects:` and the workstream numbers, or `none`. Append with a shell heredoc, never an edit, so two agents writing at once don't lose each other's entries. A contract entry carries the shape itself and not a description of it: the route, signature, payload or schema, as a `diff` block, logged when it's agreed rather than when it's built. A contract another workstream has already built against doesn't change quietly - the new shape goes beside the old one, and the entry names every workstream that has to redo something.

The log gets long, so agents read it by filter rather than from the top: `grep -n 'Affects:.*<their number>' log.md`, and the tail in full. They read on start, before each design section, and before asking for any approval.

## Numbering

`<phase><letter>-<name>.md`. The number is the phase and it follows the order work can start, so the filenames stay readable as "what can I launch now". The letter marks prompts in the same phase that run at the same time, and only appears when a phase holds two or more. Letters don't restart each phase, so a workstream keeps its letter when its phase moves: `0A-user-pool.md`, `0B-dev-idps.md`, `1C-idp-api.md`, `2-integration.md`. If everything runs one after another in a single checkout, drop the letters: the phases are just an ordered list.

A new phase needs a real dependency, not tidiness. Push each workstream into the earliest phase it can sit in, and say in the README what each phase is actually waiting for.

## Merging

The phase table says what can start. The README also has to say what merges into what: a branch off the main branch, or stacked on another workstream's branch, and in what order. Settle that when you draw the map, not when the first pull request is ready.

Give each shared file one owning workstream, named in the README. Anyone else who needs a change in it asks the owner in the log and lets them make it.

Give the last phase an integration workstream: rebase every branch onto the merged trunk, run the whole suite once on the combined result, and check by hand anything a workstream only ever proved against its own mocks. A clean merge isn't evidence that two workstreams agree, only that they changed different lines.

Say in the README who reconciles. When a phase finishes, someone reads the log's new entries against `decisions.md` and the phase table, names anything two workstreams have decided differently, checks what's been built so far against the outcome in the README, and settles both with the user before the next phase starts. A contradiction found at merge time costs a rewrite; found at the phase boundary it costs a paragraph.

## What goes in each prompt

A fresh agent reads only this file and `common.md`, so each prompt stands on its own. Same seven parts every time:

1. **Opening line.** Which workstream this is, what the project is, the repository path, "start by reading `<absolute path>/common.md` and follow it throughout", the short name to use for its branch, files and log entries, and the ticket it's working to or the instruction to create its own once its spec is approved.
2. **What's decided**, in a few bullets, each with the reason and a `file:line` or document reference.
3. **The points to settle with the user**, as questions rather than answers. Where you think you know the answer, say what you currently think it is and that it's one option to test against the others, not the answer. Mark anything you haven't verified UNCONFIRMED and tell the agent to check it.
4. **The research the agent must do** with its own subagents before proposing anything, named as specific questions.
5. **Dependencies, by the stage each one bites, and the check that it's met.** Not one list but four: what it needs before it can spec and plan, what it needs before it can build, what it needs before it can deploy or merge, and what other workstreams are waiting on from it. Name the other workstream and the exact thing, and check that workstream's prompt says it produces it, because a dependency only one side knows about is the one that bites. Write each check as the command that settles it, the `grep` of the log for the contract or the check that a branch is merged, and make the first of them the agent's first action, so it stops and says so rather than starting and guessing.
6. **What it may change.** The files and directories this workstream owns or may edit, and the shared ones it must ask about first. If the work turns out to need a change outside that list, the agent logs it as a request and asks the user rather than making it.
7. **Done means.** What proves this one finished, not the checks `common.md` already sets for everybody: the command to run, what its output has to say, and the evidence that goes in the log as a status entry. Where the agent builds, it has a fresh subagent review its diff against its own approved plan before calling it done, asking for gaps against the plan rather than style. Where the agent stops at a spec or a plan, the approval is the finish line, so say what that document has to cover to earn it.

Gate only on checks whose failure is specific and visible: a failing test, a type error, a lint rule, an accessibility violation. Numbers like complexity or a maintainability index are a reason to look, never a bar to clear - the moment one is a gate, hitting it becomes the job, and an agent will split a function in two badly-named halves to move it. Where a prompt names one, it says the number gets reported and explained in the log, and read by the reviewing subagent rather than by the agent that wrote the code.

The last thing each prompt asks is whether what got built still serves the one line this workstream was created for, and the outcome in the README. If it doesn't, the agent says so rather than closing it.

## Review the set before you hand it over

Reading each prompt on its own won't find the errors that live between them. Do both passes and fix what they turn up.

**The structural pass, in your own context**, because it needs the whole set and the conversation that produced it. Build one table, a row per workstream: what it delivers, what it waits for, what waits on it, the files it changes, the contracts it logs, the ticket. Then check:

- Every dependency points backwards. If A waits on B, B is in an earlier phase. Two that wait on each other are one workstream, or the shared part splits out into its own earlier one.
- Every "waits for" names a thing, not a workstream. "Waits for 03" can't be checked. "Waits for 03's permission keys to be in the log" can.
- Nothing is delivered by two prompts. Shared context gets repeated wherever both agents need it; shared work doesn't.
- Nothing is delivered by nobody. Every contract a prompt says it will use is one another prompt says it will log.
- The order survives the slowest workstream. If one in an early phase is several times the size of the rest, say so and ask the user whether to split it or let the others start against its logged contracts.

**The fresh-reader pass, with subagents**, because you can't un-know the conversation. Dispatch one agent per prompt, in parallel, each given only that prompt and `common.md` and nothing else. Ask each what it would do first, what it would have to guess, and what in there only makes sense if you'd been in the planning conversation. Any answer that guesses, or asks a question the prompt should have answered, goes back into the prompt.

Then say what you wrote and what order to run them in. Don't start any of the workstreams.

If you're adding a workstream to a plan that already exists, don't renumber: give it the phase it can run in, add it to the grid and the README table, and log it as a status entry so the agents already running know it exists.

If the user changes something after the prompts are written, apply it to the files rather than writing a list of changes beside them. A prompt that's out of date is worse than one that's missing, because its agent has no way to know it's reading the old answer. If there's too much to apply in one pass, say in the README which files are stale and which one wins until it's done.

Write everything, the prompts included, in the style the repository's CLAUDE.md or AGENTS.md sets.
