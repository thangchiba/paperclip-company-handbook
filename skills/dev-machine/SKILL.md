---
name: dev-machine
description: "Where and how code runs at Hoang LLC: on ThangChiba-Desktop (WSL2), where agent runs already execute. Use when cloning a project, building, testing, running scripts/verify.sh or Docker, or when a command might land on another host."
---

# Dev Machine (ThangChiba-Desktop)

Clone, build, test, Docker and `scripts/verify.sh` run here and only here (`security-baseline` §2).

## Machine facts

- Hostname `ThangChiba-Desktop` (192.168.1.111): Windows 11 with WSL2 Ubuntu. Your run is a Linux shell inside WSL.
- Docker: Docker Desktop's engine (amd64), used from WSL through Docker Desktop's WSL integration for the Ubuntu distro.

## Steps

1. First command in a run that will run code: `hostname`. Anything but `ThangChiba-Desktop`: stop, report on the task, run nothing further. There is no fallback host.
2. Work under `~/workspace/<project>` on the WSL filesystem, not `/mnt/c`. If the repo is there, `git fetch && git pull` instead of cloning again. Touch only the task's project folder.
3. Docker: use the local engine; never set `DOCKER_HOST` to an `ssh://` host. If `docker compose version` fails ("could not be found in this WSL 2 distro" means the WSL integration is off), report it on the task as an environment problem.

## Images for the Mac

Odeku prod is built on MacbookServer by its deploy webhook after a merge. Agents never push images there.

## Disk

A full C: drive stalls WSL and Docker for every agent on this machine.

- Before heavy builds: `docker system df` and `df -h /mnt/c`.
- Low on space: `docker builder prune`, then report it on the task.
- Never delete other projects' files or other agents' run folders.

## Never

- Open Docker TCP port 2375, or weaken the SSH config of this machine.
