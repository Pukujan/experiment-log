hermes-paths__001-docker-host-path-translation
tags: hermes, docker, paths, gateway, telegram, windows
updated: 2026-06-21

two filesystems: /workspace/ in docker, C:/Users/pujan/... on host
gateway cannot send /workspace/ paths -- must translate to host

patches applied:
- gateway/platforms/base.py: /workspace/ -> real Windows host path for media delivery
- tools/file_tools.py: write_file /workspace/ routes through docker backend
- config.yaml: media_delivery_allow_dirs whitelist for project + hermes cache dirs

after patching: hermes gateway stop && hermes gateway start to pick up changes
