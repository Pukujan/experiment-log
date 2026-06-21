# Docker backend image verified working

Date: 2026-06-21
Tags: docker, hermes, backend, verification, windows

## What I did
Built a custom Docker image `hermes-docker:latest` based on `nikolaik/python-nodejs:python3.11-nodejs20` with Docker CLI and jq added. Configured Hermes config.yaml with `backend: docker` and `//var/run/docker.sock` mount for MSYS compatibility.

Verified the image works by running inside the container:

- Python 3.11.15 present
- Node v20.20.2 present
- Docker CLI 26.1.5 present and can reach host socket
- jq 1.7 present

Docker socket mount requires `//var/run/docker.sock` (double slash) because MSYS translates bare `/var/run/...` to a Windows path that does not exist.

## What didn't work
Initial test command with `bash -c` piping got mangled by git bash on Windows. Simple single commands like `python3 --version` work fine. The Hermes terminal backend handles this internally so it should not be an issue once the session restarts.

## What's pending
Session needs restart (`/new` or close/reopen) for `backend: docker` to take effect. Currently still running locally (hostname: Teresa-Pujan). After restart, terminal commands will run inside the container.

## Key command
```bash
docker run --rm -v //var/run/docker.sock:/var/run/docker.sock hermes-docker:latest docker ps
```

## Lesson
The custom image works. The MSYS double-slash fix is essential on Windows. Session restart is the only remaining step.
