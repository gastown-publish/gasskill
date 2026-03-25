---
name: gascontext
description: Operate the Gasclaw gascontext stack for gastown-publish/context-hub — mayor tmux (hq-mayor), gateway :18797, gashub CLI, and repo rig (GT_RIG_URL).
metadata:
  openclaw:
    emoji: 📚
    os:
      - linux
    requires:
      bins:
        - docker
        - tmux
---

# Context Hub (gascontext) Operator

Use this skill when working on **gastown-publish/context-hub** from the **gasskill** Gasclaw instance or when coordinating with the dedicated **gascontext** container.

## Stack facts

| Item | Value |
|------|--------|
| Container name | `gascontext` |
| Gateway | `http://localhost:18797` (host and container) |
| Compose project | `name: gascontext` in `/home/gascontext/gasclaw/docker-compose.yml` |
| Rig repo | `gastown-publish/context-hub` (via `GT_RIG_URL` in container `.env`) |
| Mayor tmux | `hq-mayor` (same pattern as other Gasclaw HQs) |

**Important:** Docker Compose project names must differ per stack. Both `/home/gasskill/gasclaw` and `/home/gascontext/gasclaw` use a parent folder named `gasclaw`; each compose file sets `name: gasskill` or `name: gascontext` so `docker compose up` does not replace the wrong container.

## Quick checks

```bash
# Container up
docker ps --filter name=gascontext

# Gateway health
docker exec gascontext curl -sf http://localhost:18797/health

# Mayor session
docker exec gascontext tmux ls
docker exec -it gascontext tmux attach -t hq-mayor   # interactive
```

## gashub (install in container)

Follow [gasclaw-management adding-a-project](https://github.com/gastown-publish/gasclaw-management/blob/main/docs/adding-a-project.md) step 11 — clone `context-hub` to `/opt/gashub`, `npm install` in `cli/`, symlink `gashub` / `gashub-mcp` into `/usr/local/bin`, then `gashub update`.

## npm registry

`@gastown/gashub` may not be on the public npm registry; install from the git repo until a release is published.

## Management scripts (host)

On `gpu-workspace`, gateways and agents include `gascontext` in:

- `/home/nic/gasclaw-workspace/gasclaw-management/scripts/restart-gateways.sh`
- `/home/nic/gasclaw-workspace/gasclaw-management/scripts/activate-agents.sh`
- `/home/nic/gasclaw-workspace/gasclaw-management/scripts/watchdog.sh`
