---
name: decision-records
description: Use when the user answers a question that settles a choice, says something is decided, or changes their mind about something decided earlier; when brainstorming or design is finishing and planning is about to start; when someone is about to revisit, contradict or work around a decision already made; or when work is starting on a decision nobody wrote down. Covers ADRs, architecture decision records and design decisions kept as files in a repository.
---

# Decision records

One decision per record, with the reasoning that produced it, frozen the moment it's made. The point isn't the document, it's that six months later you can see what was decided, in what order, and why it looked right at the time, including the decisions that were later changed.

## What belongs here

Decisions about the code in this repository: the ones made while building, that otherwise live in a commit message, a pull request comment, or nobody's memory.

Not the organisation's architecture decisions. The test is what it takes to reverse one. If changing your mind means changing this repository and nothing else, it belongs here. If it means another team, a customer, a contract or a published commitment, it belongs in whatever process the organisation runs for those, and this skill's job is to say so rather than record it.

A record here can cite one of those larger decisions as its input, and never contradicts one. Work that needs something a larger decision rules out is an escalation to the user, not a record.

## Only decisions worth keeping

Most decisions don't need a record. A record is for when the why is the valuable part: real options were weighed, the one we turned down still looks tempting, and the reasoning can't be read back from the code. Without the record, someone would undo the choice or bring the other option back. If the code or a line in the agent docs already tells a later reader enough, the decision lives there instead.

Each settled answer goes to one of three places:

| Where | When | Example |
|---|---|---|
| The code | The code shows what was decided clearly enough that nobody would undo it by accident: a name, a regex, a validation rule, a test that pins the behaviour | Emails are stored lowercase. The regex that normalises them, and its test, are enough |
| The agent docs (CLAUDE.md, AGENTS.md, `.ai/rules`, whatever this repository uses) | An agent would break the rule without being told, but the rule is enough on its own and any reasoning fits in a clause | "Never log the identity provider's client secret, Telescope included" |
| A decision record | The why is the point, and it isn't visible in the code | Identity provider rows are soft-deleted because the activity log needs their history, even though a hard delete looks simpler |

When it goes in the code or the agent docs, the reasoning still gets written down once: say why we chose it in the pull request description. A reviewer reads it there now, and `git blame` leads anyone who asks later back to it.

None of these get a record either. They go in the pull request description, the spec or the commit message:

- A decision that leaves nothing in the code. "No change", "leave it as it is", or a reply to a review finding that ends in not acting on it. There's nothing for a later reader to undo.
- Anything that only holds until something else happens: what gets deployed where until a migration, which environment we test against for now, a workaround until another piece of work lands. By the time someone reads the record it describes a situation that no longer exists.
- How the work is being done rather than what the code does: deploy steps, which account or cell to use, who runs what, what the README covers.
- Anything about a thing that lives outside the repository: a test environment, someone's machine, local state, where secrets are kept. Deciding to keep it out of the repository ("nothing about the test pool is committed") is still a decision about that thing, not about the code.
- A correction to the spec or the plan. Fix the spec.
- A value that only fills in a detail (a port number, a variable's name, a timeout). It goes in the spec.

For example, a review finds the template can still deploy to two staging cells that shouldn't run it yet. The user answers "it's only staging, we're only using eu-west-2 until we migrate". That settles something, but no code changes, it stops being true the day the migration happens, and it's about where we deploy rather than what the code does. It goes in the pull request description, not a record.

Another: the user agrees a temporary test pool is applied from their own machine with local state, nothing about it committed, and deleted once staging moves. It's temporary, it lives outside the repository, and it's about who runs what. A line in the pull request description covers it.

When it's unclear, don't write a record. Put it in the code or the agent docs, with the why in the pull request description. A record that was missed can be written later from the pull request; one that shouldn't exist can't be deleted, because records are frozen.

Where the organisation calls its decisions ADRs, don't call these that as well. Two things with the same name and different authority in one codebase get confused, and the confusion is always about which one is binding.

## When a record is written

The user's answer is the acceptance. When the user answers a question that settles a choice whose why is worth a record (see above), or says something like that is decided, write the record in that same turn, before asking the next question. When the answer belongs in the code or the agent docs instead, make that change, and keep the why for the pull request description. It's accepted from that moment. There is no proposed state and no separate step where the user approves the record.

One answer that settles two things is two records. An answer that isn't clear enough to write down is a question back to the user, not a record.

A record says whose each reason is. What the user said is quoted or attributed to them; anything you add yourself (a consequence you worked out, why an option lost when they didn't say) is marked as yours, for example "(my reasoning, not checked with the user)". So you never need to correct a record for putting words in the user's mouth.

## States

| State | What it means |
|---|---|
| `accepted` | Decided. Frozen from the moment it's written |
| `superseded by <file>` | A later record replaced it. Its text stays as written |

## Frozen means frozen

An accepted record is never edited and never deleted. Not to fix the reasoning, not to add what you learned afterwards, not to bring it in line with what got built, not because the user changed their mind ten minutes later. Changing a decision means writing a new record that supersedes the old one.

Two edits are allowed to an accepted record, and only these: its status line, and the line pointing at whatever superseded it. Nothing else, including typos, and including amending or squashing the commit that added it. The details file takes no edits at all. If you find a mistake in a record you've just written, it stays, and a superseding record says what was wrong.

Why it matters: a record edited after the fact tells you what someone later wished had been decided. The reason to keep it is to see the decision as it was made, including the parts that turned out wrong.

## Where they live

A record is two files, and most of the point is the split:

```
docs/decisions/
  README.md                                          the index
  2026-10-27-2140-own-ngrok-url-per-developer.md       the decision: the frozen statement everything cites
  details/
    2026-10-27-2140-own-ngrok-url-per-developer.md     the working: optional, same name
```

The name starts with the date and the time (24-hour, local) of the user's answer, then a short title. Nothing needs allocating, so two agents in two worktrees can't collide, and the records sort in the order things were decided. Cite a record by its file name, or as "the 27 Oct 21:40 record". Numbered records from before this scheme keep their numbers.

The decision file is half a page and it's what gets read, cited and loaded into a session. The details file holds what it took to get there, and gets opened only when someone needs the why. Keeping them apart is what stops an agent that needs one line paying for twelve sections of working, and `docs/decisions/*.md` globs to exactly the decisions without dragging the details in.

The index lists each record's file, title, status, date and what supersedes what. Create the directory and the index if they don't exist.

Commit the record on the branch being worked on, in the same turn it's written. If there's no branch yet, write it in a git-ignored working folder (for example `.superpowers/decisions/`) and make it the first commit on the branch when there is one. A record is never in git before the user has decided it.

## The header

Every record opens with these four lines, with nothing above them:

```
# Short title
Status: accepted | superseded by <file>
Date: YYYY-MM-DD HH:MM
Goal: the outcome this decision serves, in one line
```

## Findable from the work

Writing a record also makes it reachable from the code it governs. Wherever this repository tells agents what applies to a path, whether that's a rules index, a CLAUDE.md or a README beside the code, add a line naming the record, what it decided and the paths it covers. A record nothing points at gets contradicted by the next agent, which is the thing it exists to stop.

## The shape

The decision file, after its header, is three sections and nothing else:

- **Summary.** What forced a decision. The constraint, the thing that broke, the fork in the road. End it with an `Input:` line naming what it came from: the conversation and the user's words, a log entry, an earlier record.
- **Decision.** What we're doing, in the present tense. It can carry a schema, a signature or a diff where that says it more exactly than prose can. Never call this section "recommendation": a record is binding, and the heading can't be fixed later.
- **Consequences.** What this makes easy, what it makes hard, and what it rules out. Spell out the workaround for each hard thing rather than leaving it implied.

The details file, where there's working worth keeping, carries whatever that is: where we are today, each option considered with a plain verdict on why it lost, the research. No header, no status, no open questions.

Write a details file when the decision rejected real alternatives, when it changes a data model, or when it coins vocabulary someone will have to look up. Most decisions won't need one, and one that has nothing to show doesn't get an empty file.

## Superseding

When the user changes their mind, in the same session or months later, write a new record straight away, in the same turn. It opens by naming what it supersedes, and says what changed, why the earlier reasoning no longer holds, and what has to be undone or migrated. Then set the earlier record's status to `superseded by <new file>`. Its text stays exactly as written, details and all.

Don't supersede a record because the implementation drifted from it. Either the drift is wrong and gets fixed, or the decision changed and that's a new record saying so.

## Before planning starts: the review

When brainstorming or design is finishing and the next step is a plan, review the decisions made in this session before planning begins.

1. Dispatch a fresh agent. Give it the records this session wrote, the index, and every earlier accepted record they cite or touch. Ask it four things, with file and line for each answer:
   - Does each record's reasoning hold, or rest on something unstated or unchecked?
   - Does any contradict an earlier accepted record, or another of this session's?
   - Does each serve its Goal line?
   - What does each cost that it doesn't mention?
   It reports. It doesn't edit anything.
2. Show the user a short list: one line per key decision this session made, in order, with superseded ones marked, then what the reviewer found, each with the record it concerns.
3. Anything the user changes is a superseding record, written then. Then planning starts. The review doesn't hold planning up beyond the user's answer.

## When a decision can't be built

Stop that line of work, not the session. Do the rest of the task and tell the user which record blocks the part you left, with what you hit. Their answer is the decision: if it changes what we'd decide, write the superseding record then. If the record still says the right thing and only a detail under it is wrong, like a column name or an endpoint that doesn't exist, fix the detail and say so; no record.

Drift and can't-be-built aren't the same thing. Drift is code that went its own way where the decision was buildable, and that gets fixed in the code. Can't-be-built is the record being wrong about the world, and that's a new record.

A workaround that never becomes a record is a decision nobody made.

## Working alongside planning-workstreams

The planner reads the accepted records before drawing the map. `decisions.md` then names the record and quotes nothing: "2026-10-27-2140-own-ngrok-url-per-developer - each developer's Keycloak is on their own ngrok URL." When a record is superseded mid-project the planner updates that line and logs a `decision` entry naming both files, because no agent may edit `decisions.md`.

A workstream agent writing a record when the user answers also logs the `decision` entry the log protocol asks for, naming the record's file, so parallel agents see it.

A `challenge` entry is the trigger, not a second procedure. The agent logs it and asks the user as `common.md` already says, and the record written from the user's answer names the log entry.

At hand-back, the person taking the work reads `deviations.md` and writes a record for every row that changed a decision worth keeping rather than finished one. That folder is machine-local and gets thrown away, so it happens then or not at all.
