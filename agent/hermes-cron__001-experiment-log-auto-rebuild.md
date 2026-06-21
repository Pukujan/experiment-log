hermes-cron__001-experiment-log-auto-rebuild
tags: hermes, cron, experiment-log, search, automation
updated: 2026-06-21

cron job runs every 1h, checks if experiment log entries newer than search.db, rebuilds if needed

hermes cron create --name experiment-log-index-rebuild --schedule "every 1h" --skill experiment-log --workdir "<project>/collective-doc-library" --prompt "Check if experiment log entries are newer than search.db, rebuild if needed"

write entries when discovered, index rebuilt automatically, no user action needed
