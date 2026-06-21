hermes-docker__002-image-verified-working
tags: docker, hermes, backend, windows
updated: 2026-06-21

image: hermes-docker:latest from nikolaik/python-nodejs:python3.11-nodejs20
+ docker.io + jq

msys double-slash //var/run/docker.sock required for socket mount on windows
single /var/run/... gets rewritten to C:/Program Files/Git/var/run/...

session restart after config change: close session (/new) for docker backend to activate
