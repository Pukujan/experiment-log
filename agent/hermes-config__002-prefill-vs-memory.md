# Memory gets pruned. prefill_messages_file does not.

Date: 2026-06-21
Tags: hermes, config, memory, prefill, system-prompt

## What I tried
I set the local-first search rule in memory. Then realized memory gets compacted and pruned by the curator. It is not reliable for durable rules.

## What worked
The config field `prefill_messages_file` at line 396 of config.yaml points to a file on disk. That file is read fresh every session and injected into the system prompt. It never gets pruned or compacted.

## Key config
```yaml
prefill_messages_file: 'C:/Users/pujan/AppData/Local/hermes/prefill.md'
```

## Lesson
For durable instructions that must survive pruning, use `prefill_messages_file`, not memory.
