hermes-docker__001-docker-backend-setup
tags: hermes, docker, windows, git-bash, json, escape
updated: 2026-06-21

TERMINAL_DOCKER_VOLUMES must use forward slashes C:/Users/... not C:\Users\...
backslashes are invalid JSON escapes -- crashes json.loads silently

do not mount docker socket, not needed for normal terminal use

docker cwd must be container path /workspace, not Windows host path

.env overrides config.yaml -- keep both consistent

after changing config or .env, fully restart Hermes (kill process, not just close tab)
