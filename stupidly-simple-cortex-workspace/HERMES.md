# Stupidly Simple Cortex

Before planning or answering a task, use Cortex first.

Required order:

1. Run the Cortex preflight:

   ```powershell
   python stupidly-simple-cortex\scripts\cortex.py before-task "<task>"
   ```

2. Search audit logs for prior solved fixes:

   ```powershell
   rg -i "<query>" audit-log-1\agent audit-log-2\agent
   ```

3. Search local docs before web lookup:

   ```powershell
   python stupidly-simple-cortex\scripts\cortex.py search "<query>"
   ```

4. Use `cortex-library` as the docs catalog and prefer `agent/` docs over `human/` docs.

5. Refresh docs when stale, then create a new `cortex-{n}` shard when a shard reaches 500 MB.

Do not put this workflow in `SOUL.md`; SOUL is for identity and tone. This is project workflow.

