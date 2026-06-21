# Hermes Docker backend on Windows — setup guide

Date: 2026-06-21
Tags: hermes, docker, windows, git-bash, hermes-config, docker-socket

## What I tried

Setting Hermes to run terminal commands inside a Docker container on
Windows 10 with Docker Desktop.

Attempted approaches:
1. Set `backend: docker` in config.yaml and `TERMINAL_ENV=docker` in .env
2. Used nikolaik/python-nodejs:python3.11-nodejs20 base image directly
3. Built custom image extending the base with docker.io and docker-cli
4. Mounted Docker socket via //var/run/docker.sock (MSYS double slash)
5. Tried single /var/run/docker.sock — failed, MSYS rewrote it to C:\Program Files\Git\var\run\docker.sock

## What worked

Final working setup:

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
  docker_image: hermes-docker:latest
  docker_volumes:
    - "C:\\Users\\pujan\\OneDrive\\Desktop\\web dev\\webdev 2.0:/workspace"
    - "//var/run/docker.sock:/var/run/docker.sock"
```

**.env setting:**
```
TERMINAL_ENV=docker
```

## What didn't work

1. **Directly editing config.yaml with write_file or patch in Hermes**
   Config file is under C:\Users\pujan\AppData\Local\hermes\config.yaml,
   NOT in the project folder. The tool guard blocks writes outside the
   project directory. Use `hermes config set` CLI commands instead.

2. **Single slash /var/run/docker.sock from MSYS/git-bash**
   MSYS translates Unix paths starting with / to Windows paths using
   its root mapping. /var/run/docker.sock becomes
   C:\Program Files\Git\var\run\docker.sock which doesn't exist.
   Double slash //var/run/docker.sock bypasses this translation.

3. **Without docker.io installed in the container image**
   The nikolaik/python-nodejs image does not include Docker CLI.
   Even with the socket mounted, docker ps fails inside the container.

4. **Terminal tool still runs locally after config changes**
   The terminal backend type is read at Hermes process startup.
   Changes to config.yaml or .env require a restart (/new or close
   and reopen Hermes). There is no live reload for terminal backend.

## Verification commands

```bash
# Verify image is built
docker images hermes-docker:latest

# Verify tools inside container
docker run --rm -v //var/run/docker.sock:/var/run/docker.sock \
  hermes-docker:latest sh -c "python3 --version && node --version && docker ps"

# Check config is set
hermes config get terminal.backend
hermes config get terminal.docker_image
```

## Lesson

Hermes Docker backend on Windows needs: (1) a custom image with docker.io
installed, (2) double-slash //var/run/docker.sock for the socket mount,
(3) properly escaped backslashes in docker_volumes, and (4) a session
restart to take effect. The config.yaml tool guard blocks direct edits,
so use `hermes config set` for config changes.
