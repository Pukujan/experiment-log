# Experiment Log — Agent Protocol

All entries in agent/ are agent written and agent readable.
The human does not maintain this repo.

## Purpose

When a problem reappears or a tool needs configuring again, search here
before trying anything new. If a solution took 3 iterations to get right,
the third iteration is the one to use.

## Structure

```
agent/
  <prefix>__<nnn>-<description>.md
```

Prefixes: hermes-config, hermes-docker, hermes-paths, hermes-cron, hermes-<topic>

## Entry format (agent only)

```text
<file name without .md>
tags: comma, separated, tags

compact facts. executable commands.
one per line or short block.

exact commands with arguments.
prefer bash/powershell one liners.

key config values if applicable.
```

No prose. No sections. No markdown headings. No lessons.
Just tags, facts, commands.

Rules:
1. Tags: tool name, problem type, os (hermes, docker, windows, git-bash, etc.)
2. First line is always the filename without extension matching the entry
3. Include exact CLI commands that solved it
4. Update entries when new info supersedes old info
5. Do not repeat the date -- git history tracks that
