---
name: gasclaw-context
description: Operate the gasclaw-context stack for gastown-publish/context-hub — mayor tmux (hq-mayor), gateway :18797, gashub CLI, and repo rig (GT_RIG_URL).
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

# Context Hub (`gasclaw-context`) operator

Use this skill when working on **gastown-publish/context-hub** from the **gasskill** Gasclaw instance or when coordinating with the dedicated **`gasclaw-context`** container (same `gasclaw-*` prefix as other stacks for easy filtering).

## Stack facts

| Item | Value |
|------|--------|
| Container name | `gasclaw-context` |
| Gateway | `http://localhost:18797` (host and container) |
| Compose project | `name: gasclaw-context` in `/home/gascontext/gasclaw/docker-compose.yml` |
| Rig repo | `gastown-publish/context-hub` (via `GT_RIG_URL` in container `.env`) |
| Mayor tmux | `hq-mayor` (same pattern as other Gasclaw HQs) |

**Important:** Docker Compose project names must differ per stack. Both `/home/gasskill/gasclaw` and `/home/gascontext/gasclaw` use a parent folder named `gasclaw`; each compose file sets `name: gasskill` or `name: gasclaw-context` so `docker compose up` does not replace the wrong container.

## Quick checks

```bash
# All Gasclaw stacks (filter)
docker ps --filter name=gasclaw-

# This container
docker ps --filter name=gasclaw-context

# Gateway health
docker exec gasclaw-context curl -sf http://localhost:18797/health

# Mayor session
docker exec gasclaw-context tmux ls
docker exec -it gasclaw-context tmux attach -t hq-mayor   # interactive
```

## gashub (install in container)

Follow [gasclaw-management adding-a-project](https://github.com/gastown-publish/gasclaw-management/blob/main/docs/adding-a-project.md) step 11 — clone `context-hub` to `/opt/gashub`, `npm install` in `cli/`, symlink `gashub` / `gashub-mcp` into `/usr/local/bin`, then `gashub update`.

## npm registry

`@gastown/gashub` may not be on the public npm registry; install from the git repo until a release is published.

## Management scripts (host)

On `gpu-workspace`, gateways and agents include **`gasclaw-context`** in:

- [`scripts/restart-gateways.sh`](https://github.com/gastown-publish/gasclaw-management/blob/main/scripts/restart-gateways.sh)
- [`scripts/activate-agents.sh`](https://github.com/gastown-publish/gasclaw-management/blob/main/scripts/activate-agents.sh)
- [`scripts/watchdog.sh`](https://github.com/gastown-publish/gasclaw-management/blob/main/scripts/watchdog.sh)

Mayor / ops note: [gasclaw-context.md](https://github.com/gastown-publish/gasclaw-management/blob/main/docs/gasclaw-context.md) in **gasclaw-management**.
