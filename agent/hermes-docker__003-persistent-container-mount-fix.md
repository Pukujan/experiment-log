hermes-docker__003-persistent-container-mount-fix
tags: docker, hermes, persistent-container, mounts, windows
updated: 2026-06-21

fix: docker rm -f <old-container> then hermes gateway restart
config change alone insufficient when container_persistent=true
volumes cached at container creation time

bulk: docker ps -aq --filter "ancestor=hermes-docker:latest" | ForEach-Object { docker rm -f $_ } ; hermes gateway restart

verification: /hermes-config present with config.yaml and skills/ dir
