# Hermes Docker path / Telegram media handoff

Date: 2026-06-21
Tags: hermes, docker, paths, gateway, telegram, write_file, windows, codx

## What I tried

Running Hermes tools in Docker with the project mounted at `/workspace`, while the Telegram gateway needs to send files from Docker paths to users.

## Root cause

Two filesystem contexts that don't align:

1. **Docker container** — terminal and code tools see `/workspace/...`
2. **Windows host** — Telegram gateway, media delivery, and host-side tools see `C:/Users/pujan/OneDrive/Desktop/web dev/webdev 2.0/...`

Before the fix, assistant responses containing paths like:
- `/workspace/foo.gif`
- `/workspace/report.md`

would fail because the Telegram gateway tried to check/send those paths directly on Windows. `/workspace/...` does not exist on Windows, so files and GIFs silently vanished or failed delivery.

Related: `write_file` had the same Docker/host mismatch. It could resolve `/workspace/...` into incorrect host paths like `C:\workspace\...`, or write somewhere the Docker container could not see.

## What was fixed

### 1. Gateway media delivery path translation

Patched: `hermes-agent/gateway/platforms/base.py`

New behavior: `/workspace/file.ext` is translated to the real Windows host path before media validation and upload.

`/workspace/foo.gif` becomes `C:/Users/pujan/OneDrive/Desktop/web dev/webdev 2.0/foo.gif`

### 2. Gateway bare file detection

The same path translation now runs on bare file references in assistant replies. So a response containing `/workspace/generated.gif` is detected as a file, converted to the Windows host path, and uploaded to Telegram.

### 3. write_file Docker workspace awareness

Patched: `hermes-agent/tools/file_tools.py`

When asked to write `/workspace/foo.txt`, write_file now writes through the Docker backend using `/workspace/foo.txt`, but reports the matching host path back to the caller.

### 4. Gateway media allowlist

Patched: `config.yaml`

Current allowed directories for media delivery:

```
gateway:
  strict: false
  media_delivery_allow_dirs:
  - C:/Users/pujan/OneDrive/Desktop/web dev/webdev 2.0
  - C:/Users/pujan/AppData/Local/hermes/gif-cache
  - C:/Users/pujan/AppData/Local/hermes/images
  - C:/Users/pujan/AppData/Local/hermes/cache/images
  trust_recent_files: true
  trust_recent_files_seconds: 600
```

## What worked in verification

- `/workspace/hermes_media_path_test.gif` translated to the real Windows project path
- Telegram/gateway media validator accepted the translated path
- `write_file('/workspace/hermes_write_file_path_test.txt', ...)` created the real mounted project file
- Docker could read that same file through `/workspace/...`
- Python compile checks passed for both patched files

## Important

After these patches, restart the gateway to pick up the new code/config:

```
hermes gateway stop
hermes gateway start
```

Also restart any open `hermes` CLI session when testing file tools there.

## Troubleshooting

If Telegram files/GIFs still fail:
- Check whether the assistant is returning `/workspace/...` or a Windows path
- `/workspace/...` should now be translated
- Files must actually exist under the mounted project folder or one of the allowlisted Hermes cache folders
- Old gateway logs may still contain pre-fix errors; check timestamps
