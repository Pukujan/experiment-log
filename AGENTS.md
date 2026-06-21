# Experiment Log — Agent Protocol

This directory records everything this agent has tried on Pujan's local
machine. All entries in experiments/ and fixes/ are agent written and
agent readable. The human does not maintain this repo.

## Purpose

When a problem reappears or a tool needs configuring again, search here
before trying anything new. If a solution took 3 iterations to get right,
the third iteration is the one to use.

## Structure

```
experiments/<topic>/
  _index.md             TL;DR — one line goal, one line verdict, date last touched
  001-resolution.md     What happened, in entry order
  ...

fixes/<tool-or-project>/
  _index.md
  001-fix-name.md
```

Use experiments/ when you set something up for the first time or
explored unknown territory. Use fixes/ when something was broken and
you un broke it.

## Entry format

```markdown
# Short descriptive title

Date: <YYYY-MM-DD>
Tags: comma, separated, tags

## What I tried
Context and commands attempted.

## What worked
The actual solution. Commands that succeeded, config that stuck.

## What didn't work
Dead ends and why. Save future sessions from repeating them.

## Key commands
bash
  the command that actually fixed it

## Lesson
One sentence. If you take nothing else from this entry, take this.
```

## Rules

1. Always create an AGENTS.md entry when a solution took more than
   3 iterations or a non obvious combination of config settings.
2. Entries are written as markdown under the agent/ directory so the
   collective-doc-library search.py picks them up via its standard
   agent/ convention.
3. The _index.md is the TL;DR. It has no more than 5 lines.
4. Tag consistently. Tags are: tool name, problem type, os
   (hermes, docker, hermes-config, windows, git-bash, networking, etc.)
