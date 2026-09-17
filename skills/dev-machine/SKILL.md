---
name: dev-machine
description: Use the Hoang LLC dev machine (192.168.1.111) as the preferred environment for development work. Trigger when cloning a project, setting up a dev environment, running heavy builds/tests, or building Docker images. The machine is powerful (28 CPUs) but not always on — always check availability first and fall back to local if it is offline.
---

# Dev Machine (192.168.1.111)

Hoang LLC has a dedicated dev machine. **Prefer it over your local workspace** for cloning projects, development, heavy builds/tests, and Docker image builds.

## Machine facts

- Host: `192.168.1.111`, hostname `ThangChiba-Desktop`
- OS: Windows 11 with WSL2 Ubuntu; SSH user: `thang` (key-based auth already set up, no password)
- Docker: v29.x, Linux engine — 28 CPUs, ~15.5 GB RAM available to Docker
- Docker TCP port 2375 is intentionally closed (security). Use SSH-based access only.
- **Not always powered on.** Availability check is mandatory before relying on it.

## Step 1 — Availability check (always do this first)

```bash
ping -c 1 -W 2 192.168.1.111 && ssh -o ConnectTimeout=5 -o BatchMode=yes thang@192.168.1.111 "hostname"
```

- Both succeed → use the dev machine.
- Either fails → the machine is off/asleep. **Do not block on it.** Fall back to local execution, note in your task comment that the dev machine was offline, and continue.

## Step 2 — Where to work

SSH lands in a Windows shell. For dev work, always enter WSL2 Ubuntu:

```bash
ssh thang@192.168.1.111        # interactive
ssh thang@192.168.1.111 "wsl -e bash -lc '<command>'"   # one-shot Linux command
```

Clone and develop projects **inside WSL2** (Linux filesystem, e.g. `~/workspace/<project>`), not on the Windows `C:` mount — WSL filesystem I/O is much faster.

```bash
ssh thang@192.168.1.111 "wsl -e bash -lc 'mkdir -p ~/workspace && cd ~/workspace && git clone <repo-url>'"
```

If the repo already exists there, `git fetch && git pull` instead of re-cloning.

## Step 3 — Docker builds

Prefer the dev machine's Docker engine for image builds:

```bash
# From your local machine, over SSH (no TCP exposure needed):
docker -H ssh://thang@192.168.1.111 build -t <image:tag> .

# Or set it for a whole session:
export DOCKER_HOST=ssh://thang@192.168.1.111
```

Alternatively run the build entirely on the machine inside WSL after cloning there.

## Rules

- Never open TCP 2375 or weaken SSH config on this machine.
- Do not store secrets in files on the dev machine; use env vars scoped to the session or Paperclip secrets.
- Clean up large temporary artifacts (dangling images, build caches) if a build fails repeatedly: `docker system df` before `docker builder prune`.
- If the machine is offline and the task is urgent, proceed locally and mention the fallback in the issue comment. Do not ask a human to power it on unless the task explicitly requires that machine.
