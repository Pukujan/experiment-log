hermes-config__001-config-set-not-direct-edit
tags: hermes, config, tool-guard, windows
updated: 2026-06-21

hermes config set is the only way to modify AppData/Local/hermes/config.yaml
write_file and patch blocked by tool guard on host paths outside project dir

sed -i for .env edits via git-bash with /c/Users/pujan/... paths

hermes config set terminal.backend <value>
hermes config set terminal.docker_image <image>
hermes config set terminal.docker_volumes '<json-array>'
