# Cron job for automatic experiment log index rebuild

Date: 2026-06-21
Tags: hermes, cron, experiment-log, search, automation

## What I set up
A cron job named `experiment-log-index-rebuild` that runs every hour, checks if any experiment log entry is newer than the search.db index, and rebuilds if needed. No user action required.

## Key config
```bash
hermes cron create \
  --name experiment-log-index-rebuild \
  --schedule "every 1h" \
  --skill experiment-log \
  --workdir "C:/Users/pujan/.../collective-doc-library" \
  --prompt "Check if experiment log entries are newer than search.db, rebuild if needed"
```

## Lesson
The experiment log is automated end to end. Write entries happen when the agent discovers something worth recording. Index rebuilds happen automatically via cron. The user interacts with neither.
