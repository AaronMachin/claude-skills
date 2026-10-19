# claude-skills

Skills for Claude Code.

## planning-workstreams

Splits a large piece of work into workstreams and writes the prompt that starts each one, so several agents can run it at the same time or one after another.

It produces a shared folder holding a dependency grid, a phase table with separate orders for specifying and building, the decisions that are settled before anyone starts, an append-only log every agent writes to, and one prompt per workstream.

Each prompt hands its agent a piece of the problem to work out rather than a list of tasks, so the agent runs its own brainstorm, spec and plan for its slice.

### Installing

Copy the folder into your skills directory and start a new session:

```
cp -r planning-workstreams ~/.claude/skills/
```

### Requires

The superpowers plugin, from the official plugin marketplace. The skill uses `superpowers:brainstorming`, and the prompts it writes point their agents at `superpowers:writing-plans` and `superpowers:using-git-worktrees`.
