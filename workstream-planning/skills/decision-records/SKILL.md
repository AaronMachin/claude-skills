---
name: decision-records
description: Use when the user says something has been decided, when a choice between options gets settled in conversation, when someone is about to revisit, contradict or work around a decision already made, or when work is starting on a decision nobody wrote down. Covers ADRs, architecture decision records and design decisions kept as files in a repository.
---

# Decision records

One decision per record, with the reasoning that produced it, frozen once accepted. The point isn't the document, it's that six months later you can see what was decided and why it looked right at the time.

## What belongs here

Decisions about the code in this repository: the ones made while building, that otherwise live in a commit message, a pull request comment, or nobody's memory.

Not the organisation's architecture decisions. The test is what it takes to reverse one. If changing your mind means changing this repository and nothing else, it belongs here. If it means another team, a customer, a contract or a published commitment, it belongs in whatever process the organisation runs for those, and this skill's job is to say so rather than record it.

A record here can cite one of those larger decisions as its input, and never contradicts one. Work that needs something a larger decision rules out is an escalation to the user, not a record.

Where the organisation calls its decisions ADRs, don't call these that as well. Two things with the same name and different authority in one codebase get confused, and the confusion is always about which one is binding.

## States

| State | What it means | Who moves it |
|---|---|---|
| `proposed` | Written, under discussion. Revises freely | Whoever wrote it |
| `accepted` | Decided. Frozen from this point | The user, after a review |
| `rejected` | Considered and turned down. The record stays | The user |
| `superseded` | A later record replaced it. Its text stays as written | Set when the replacement is accepted |

## Frozen means frozen

An accepted record is never edited. Not to fix the reasoning, not to add what you learned afterwards, not to bring it in line with what got built. Changing a decision means writing a new record that supersedes the old one.

Two edits are allowed to an accepted record, and only these: its status line, and the line pointing at whatever superseded it. Nothing else, including typos. The details file freezes at the same moment and takes no edits at all, because reasoning you can rewrite afterwards isn't a record of why you decided something, it's a record of what you later wished you had.

Before acceptance it's a draft and rewrites as much as it needs to. If it carries a changes log, that log covers the proposal only and stops at acceptance.

Why it matters: a record edited after the fact tells you what someone later wished had been decided. The reason to keep it is to see the decision as it was made, including the parts that turned out wrong.

## Where they live

A record is two files, and most of the point is the split:

```
docs/decisions/
  README.md                         the index
  0007-orders-are-never-deleted.md    the decision: the frozen statement everything cites
  details/
    0007-orders-are-never-deleted.md  the working: optional, same number, same name
```

The decision file is half a page and it's what gets read, cited and loaded into a session. The details file holds what it took to get there, and gets opened only when someone needs the why. Keeping them apart is what stops an agent that needs one line paying for twelve sections of working, and `docs/decisions/*.md` globs to exactly the decisions without dragging the details in.

The index lists number, title, status, date, and what supersedes what. Create the directory and the index if they don't exist.

A proposed record has no number. It's `proposed-<short-title>.md`, with its details alongside at `details/proposed-<short-title>.md`. The number is allocated at acceptance, by the user, in the main checkout, from the index. Numbering is the one thing two agents can't both get right: each checks the directory in its own worktree, neither sees the other's uncommitted file, both take the next number, the filenames differ by slug, and git merges both without ever raising a conflict. Checking before you write doesn't fix that. One person accepting one record at a time does.

Accepting renames both files to `NNNN-short-title.md`, four digits, puts the number in the heading, sets the status and adds the row to the index. That's the same moment it freezes, so it isn't an edit to an accepted record. A rejected record is numbered too, when it's rejected, because the point of keeping it is that someone can cite it. Numbers are never reused.

Cite a proposed record by its filename, and fix the citation when it gets its number, which is allowed because your own record is still proposed too.

## The header

Every record opens with these four lines, with nothing above them:

```
# NNNN. Short title
Status: proposed | accepted | rejected | superseded by NNNN
Date: the date of that status
Goal: the outcome this decision serves, in one line
```

While it's proposed the heading carries no number and the status is `proposed`. The only edit allowed on an accepted record is its `Status:` line. Not the date, not the goal, not a typo.

## Findable from the work

Accepting a record also makes it reachable from the code it governs. Wherever this repository tells agents what applies to a path, whether that's a rules index, a CLAUDE.md or a README beside the code, add a line naming the number, what it decided and the paths it covers. A record nothing points at gets contradicted by the next agent, which is the thing it exists to stop.

## The shape

The decision file, after its header, is three sections and nothing else:

- **Summary.** What forced a decision. The constraint, the thing that broke, the fork in the road.
- **Decision.** What we're doing, in the present tense. It can carry a schema, a signature or a diff where that says it more exactly than prose can. Never call this section "recommendation": a record is binding, and the heading can't be fixed later.
- **Consequences.** What this makes easy, what it makes hard, and what it rules out. Spell out the workaround for each hard thing rather than leaving it implied.

The details file, where there's working worth keeping, carries whatever that is: where we are today, each option considered with a plain verdict on why it lost, how it's stored, the research. No header, no status, no open questions. It's reference, and the decision file is what's binding.

Write a details file when the decision rejected real alternatives, when it changes a data model, or when it coins vocabulary someone will have to look up. Most decisions won't need one, and one that has nothing to show doesn't get an empty file.


## The review

Before the user accepts anything, a fresh agent reviews it. Fresh because the writer can't see what they assumed.

Give it both files, the index, every accepted decision this one cites or contradicts, and the record's own Goal line. Most of what it's checking is in the details. Ask it four things:

- Does the reasoning hold, or does the decision rest on something unstated or unchecked?
- Does it contradict an accepted record? Name the number and the line.
- Does it serve the goals, or has the work drifted into something nobody asked for?
- What does it cost, that the record doesn't mention?

It reports. It does not accept, reject, edit or renumber anything. The user decides, and a review that says "looks good" with no specifics hasn't been done.

## Superseding

A new record, in its own file with its own number, saying what changed, why the earlier reasoning no longer holds, and what has to be undone or migrated. It opens by naming what it supersedes.

The superseded record keeps its text exactly, details and all. Its status becomes `superseded` and it gains one line pointing at the replacement.

Don't supersede a record because the implementation drifted from it. Either the drift is wrong and gets fixed, or the decision changed and that's a new record saying so.

## Rejecting

A rejected record keeps its number and its text, with the reason added. It's what stops the next person doing the work again.

## When a decision can't be built

Stop that line of work, not the session. Do the rest of the task and say in your next message which record blocks the part you left. Don't work around it quietly, and don't edit the record.

Write a proposed superseding record if what you hit changes what we'd decide. Raise it with the user instead if the record still says the right thing and only a detail under it is wrong, like a column name or an endpoint that doesn't exist. If you can't tell which, write the proposal: rejecting it costs the user a line.

Drift and can't-be-built aren't the same thing. Drift is code that went its own way where the decision was buildable, and that gets fixed in the code. Can't-be-built is the record being wrong about the world, and that's a new record.

Don't build against your own proposal before it's accepted. Build what doesn't depend on it and say what's waiting.

A workaround that never becomes a record is a decision nobody made.

## Working alongside planning-workstreams

The planner reads the accepted records before drawing the map. `decisions.md` then names the record and quotes nothing: "0007 - orders are cancelled, never deleted." When a record is superseded mid-project the planner updates that line and logs a `decision` entry naming both numbers, because no agent may edit `decisions.md`.

A `challenge` entry is the trigger, not a second procedure. The agent logs it and asks the user as `common.md` already says, and whoever writes the record afterwards names the log entry in it.

At hand-back, the person taking the work reads `deviations.md` and writes a record for every row that changed a decision rather than finished one. That folder is machine-local and gets thrown away, so it happens then or not at all.
