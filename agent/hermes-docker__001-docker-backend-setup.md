# Hermes Docker backend on Windows — full fix

Date: 2026-06-21
Updated: 2026-06-21 — corrected backslash JSON escape bug and socket mount
Tags: hermes, docker, windows, git-bash, hermes-config, json, escape

## What I tried

Setting Hermes to run terminal commands inside a Docker container on
Windows 10 with Docker Desktop.

Attempts:
1. Set `backend: docker` in config.yaml and `TERMINAL_ENV=docker` in .env
2. Used nikolaik/python-nodejs:python3.11-nodejs20 base image directly
3. Built custom image extending the base with docker.io and docker-cli
4. Mounted Docker socket via //var/run/docker.sock (MSYS double slash)
5. Tried single /var/run/docker.sock — failed, MSYS rewrote it to
   C:\Program Files\Git\var\run\docker.sock

## Root cause of terminal/code tool failure

Terminal and execute_code tools both failed silently. The actual error was:

```
ValueError: Invalid value for TERMINAL_DOCKER_VOLUMES
json.decoder.JSONDecodeError: Invalid \escape
```

`TERMINAL_DOCKER_VOLUMES` was written with Windows backslash paths inside
a JSON string:

```
["C:\Users\pujan\...:/workspace"]
```

In JSON, backslashes start escape sequences. `\U`, `\O`, `\D`, etc. are
invalid JSON escapes. Hermes parses `TERMINAL_DOCKER_VOLUMES` with
`json.loads()`, so the entire env config setup failed before Docker
commands could run. This crashed both `terminal` and `execute_code` tools
since both call `_get_env_config()`.

## What worked

Final working config. Use forward slashes in JSON paths, not backslashes.
Do not mount the Docker socket. Set cwd to a container path.

**Custom Dockerfile** at `.hermes/Dockerfile`:
```dockerfile
FROM nikolaik/python-nodejs:python3.11-nodejs20
USER root
RUN apt-get update && apt-get install -y --no-install-recommends \
    docker.io \
    jq \
    && rm -rf /var/lib/apt/lists/*
CMD ["/bin/bash"]
```

**Build command:**
```bash
docker build -t hermes-docker:latest .hermes/
```

**Config.yaml settings:**
```yaml
terminal:
  backend: docker
  cwd: /workspace
  docker_image: hermes-docker:latest
  docker_volumes: '["C:/Users/pujan/OneDrive/Desktop/web dev/webdev 2.0:/workspace"]'
  docker_mount_cwd_to_workspace: false
```

**.env settings:**
```
TERMINAL_ENV=docker
TERMINAL_CWD=/workspace
TERMINAL_DOCKER_IMAGE=hermes-docker:latest
TERMINAL_DOCKER_VOLUMES=["C:/Users/pujan/OneDrive/Desktop/web dev/webdev 2.0:/workspace"]
```

Key rules:
- Use forward slashes `C:/Users/...` not `C:\Users\...` in JSON values
- Do not mount the Docker socket — not needed for normal terminal commands
  and adds Windows/Docker Desktop compatibility issues
- Docker cwd must be `/workspace`, not a Windows host path
- Keep .env and config.yaml consistent (env overrides config)
- After changing, fully quit Hermes and restart from fresh PowerShell

## What didn't work

1. **Backslash paths in JSON** — `\U`, `\O`, `\D` are invalid JSON escapes
   that crash json.loads. Hermes silently fails terminal/code tools.

2. **Windows host path as Docker cwd** — `docker run -w` needs a container
   path, not a Windows one.

3. **Docker socket mount** — unnecessary for basic terminal execution.
   MSYS translation problems add no value here.

4. **Direct config.yaml editing via write_file/patch** — blocked by Hermes
   tool guard. Use `hermes config set` for config, sed for .env.

5. **Partial restart** — just closing the tab is not enough. A running
   Hermes process keeps old env/config state in memory.

6. **Old errors.log entries** — pre-fix errors remain in the log. Ignore
   if timestamp is before the restart.

## Verification

```bash
# Direct Docker mount test
docker run --rm -v "C:/Users/pujan/OneDrive/Desktop/web dev/webdev 2.0:/workspace" -w /workspace hermes-docker:latest python -c "import os; print(os.getcwd()); print(os.path.exists('/workspace'))"
# Output: /workspace  True

# Docker version proves Docker itself works
docker version  # Client/Server 29.4.1

# Hermes terminal tool output after fix showed:
#   env_type = docker
#   cwd = /workspace
#   docker_volumes = ["C:/Users/pujan/.../webdev 2.0:/workspace"]

# Hermes execute_code tool also worked after the fix
```

## Separate unrelated issue

Discord gateway failed with privileged intents errors. That is unrelated
to Docker terminal commands. Ignore it unless working on Discord.

## Lesson

`TERMINAL_DOCKER_VOLUMES` MUST use forward slashes on Windows. Backslashes
are invalid JSON escapes and silently crash the Hermes env config parser.
Keep the config simple — no socket mount, no Windows paths as cwd. Verify
config and .env are consistent, then restart Hermes fully.
