# Stupidly Simple Cortex Workspace

Local workspace for the new Cortex repos.

## Exact Problem It Solves

Stupidly Simple Cortex is not a replacement for Obsidian, Honcho, Holographic,
Mnemosyne, or long-term chat memory. Those systems are useful memory layers.

Cortex solves the missing operational layer around them: before an agent plans,
it needs a deterministic place to check local docs, prior working commands,
versioned changes, and repo-backed audit trails. Cortex makes that lookup
explicit, scriptable, refreshable, and easy to push to GitHub.

In short:

- Obsidian is great for human notes; Cortex is a pre-task agent lookup path.
- Honcho-style memory is great for user modeling; Cortex is repo-backed docs and fix history.
- Holographic/Mnemosyne-style memory can help recall; Cortex preserves exact commands, files, diffs, and source paths.
- Generic vector memory retrieves similarities; Cortex also keeps canonical markdown, manifests, shards, and audit logs.

```text
stupidly-simple-cortex/            stripped package spine
stupidly-simple-cortex-migration/  old-to-new migration recipe
cortex-library/                    docs catalog and search
cortex-1/                          docs shard 1
cortex-2/                          docs shard 2
audit-log-library/                 audit log catalog
audit-log-1/                       audit log shard 1
audit-log-2/                       audit log shard 2
```

Use:

```powershell
python stupidly-simple-cortex\scripts\cortex.py before-task "your task"
python stupidly-simple-cortex\scripts\cortex.py search "Hermes Docker volume"
python stupidly-simple-cortex\scripts\cortex.py shard-check
```

For Hermes CLI, start from this folder so `HERMES.md` is loaded:

```powershell
cd stupidly-simple-cortex-workspace
hermes
```

For the stricter preflight wrapper:

```powershell
.\stupidly-simple-cortex\bin\hermes-cortex.ps1 "your task"
```

## Install Into Hermes

From GitHub:

```powershell
hermes plugins install Pukujan/stupidly-simple-cortex --enable
```

Then point the plugin at this workspace:

```powershell
[Environment]::SetEnvironmentVariable("CORTEX_WORKSPACE", (Get-Location).Path, "User")
```

Restart Hermes after installing or changing `CORTEX_WORKSPACE`.
