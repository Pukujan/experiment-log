# Stupidly Simple Cortex Workspace - Agent Protocol

This folder groups the new Cortex repos.

Search order:

1. `audit-log-*` for solved local fixes.
2. `cortex-library` for docs search.
3. `cortex-*` for returned source files.
4. Web only when local Cortex misses or current upstream docs are required.

Use:

```powershell
python stupidly-simple-cortex\scripts\cortex.py before-task "task"
python stupidly-simple-cortex\scripts\cortex.py search "query"
```

Do not modify the old `collective-*` or `experiment-*` repos unless the user asks for legacy migration work.

