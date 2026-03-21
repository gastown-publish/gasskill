---
name: tmux-manager
description: Comprehensive tmux session, window, and pane management. Use when creating persistent terminal sessions, managing remote development environments, running background processes, or organizing multiple terminal workspaces. Supports session persistence, window layouts, pane synchronization, and remote tmux over SSH.
---

# Tmux Manager Skill

Master tmux for persistent terminal sessions, remote development, and organized workspace management.

## Quick Reference

| Task | Command |
|------|---------|
| New session | `tmux new -s <name>` |
| Attach | `tmux attach -t <name>` |
| Detach | `Ctrl+b d` |
| List | `tmux ls` |
| Kill | `tmux kill-session -t <name>` |
| New window | `Ctrl+b c` |
| Switch window | `Ctrl+b <n>` |
| Split vertical | `Ctrl+b %` |
| Split horizontal | `Ctrl+b "` |
| Synchronize | `Ctrl+b :setw synchronize-panes` |

## Session Management

### Create Persistent Sessions

```bash
# Create named session
tmux new -s myproject

# Create detached session with command
tmux new -s server -d "npm run dev"

# Create session with multiple windows
tmux new -s dev -n editor
tmux new-window -t dev -n server
tmux new-window -t dev -n logs
```

### Session Persistence Patterns

```bash
# Layout: Development Environment
tmux new -s dev -d
tmux rename-window -t dev:0 'code'
tmux new-window -t dev -n 'server'
tmux new-window -t dev -n 'test'
tmux new-window -t dev -n 'db'

# Send commands to each window
tmux send-keys -t dev:0 'vim' C-m
tmux send-keys -t dev:1 'npm run dev' C-m
tmux send-keys -t dev:2 'npm test -- --watch' C-m
tmux send-keys -t dev:3 'psql db_name' C-m

# Attach to session
tmux attach -t dev
```

## Window Layouts

### Pane Configurations

```bash
# 3-pane layout (editor + terminal + logs)
tmux new -s work -d
tmux split-window -h -t work:0
tmux split-window -v -t work:0.1
tmux send-keys -t work:0.0 'vim' C-m
tmux send-keys -t work:0.1 'npm run dev' C-m
tmux send-keys -t work:0.2 'tail -f logs/app.log' C-m

# 4-pane grid (monitoring)
tmux new -s monitor -d
tmux split-window -h
tmux split-window -v
tmux select-pane -t 0
tmux split-window -v
```

### Synchronized Panes

```bash
# Run same command in multiple panes (e.g., multiple servers)
Ctrl+b :setw synchronize-panes on

# Disable synchronization
Ctrl+b :setw synchronize-panes off

# Script to create synchronized cluster
tmux new -s cluster -d
tmux split-window -h
tmux split-window -h
tmux select-layout even-horizontal
tmux setw synchronize-panes on
```

## Remote Tmux Over SSH

### Gateway Pattern for Remote Work

```bash
# SSH into remote and attach/create tmux
toad ssh openclawmaster -- "tmux new -s remote-dev -A -d"

# Run command in remote tmux
toad ssh openclawmaster -- "tmux send-keys -t remote-dev 'cd /app && npm start' C-m"

# List remote sessions
toad ssh openclawmaster -- "tmux ls"
```

### Persistent Remote Development

```bash
# Setup script for openclawmaster
toad ssh openclawmaster << 'EOF'
  # Create workspace session
  tmux new -s workspace -d 2>/dev/null || tmux kill-session -t workspace 2>/dev/null; tmux new -s workspace -d
  
  # Setup windows
  tmux rename-window -t workspace:0 'logs'
  tmux new-window -t workspace -n 'app'
  tmux new-window -t workspace -n 'shell'
  
  # Start services
  tmux send-keys -t workspace:0 'cd /var/log && tail -f syslog' C-m
  tmux send-keys -t workspace:1 'cd /opt/app && ./start.sh' C-m
  
  echo "Workspace ready. Attach with: tmux attach -t workspace"
EOF
```

## Multi-Node Tmux (Swarm Pattern)

### Run Command Across Multiple Hosts

```bash
# Create monitoring session for cluster
tmux new -s cluster -d

# Create panes for each node
for node in openclawmaster gpu-server-1 geekcon; do
  tmux split-window -t cluster
done
tmux select-layout tiled

# SSH into each and run command
pane=0
for node in openclawmaster gpu-server-1 geekcon; do
  tmux send-keys -t cluster.$pane "ssh $node" C-m
  tmux send-keys -t cluster.$pane "htop" C-m
  ((pane++))
done
```

## Tmux + Subagents

### Persistent Agent Workspaces

```python
# /// script
# requires-python = ">=3.9"
# dependencies = []
# ///

"""
Tmux Session Manager for Subagents
Creates persistent workspaces for long-running agent tasks.
"""

import subprocess
import json

def create_agent_session(session_name, tasks):
    """
    Create tmux session with window per task.
    
    tasks: [{"name": "", "command": "", "dir": ""}]
    """
    # Create session
    subprocess.run(["tmux", "new", "-s", session_name, "-d"], check=True)
    
    for i, task in enumerate(tasks):
        if i > 0:
            subprocess.run([
                "tmux", "new-window", "-t", session_name, "-n", task["name"]
            ])
        
        window = f"{session_name}:{i}"
        
        # Change directory if specified
        if "dir" in task:
            subprocess.run([
                "tmux", "send-keys", "-t", window, 
                f"cd {task['dir']}", "C-m"
            ])
        
        # Run command
        subprocess.run([
            "tmux", "send-keys", "-t", window,
            task["command"], "C-m"
        ])
    
    return session_name

# Example: Create dev environment
tasks = [
    {"name": "api", "command": "python -m uvicorn api:app --reload", "dir": "/app"},
    {"name": "worker", "command": "python worker.py", "dir": "/app"},
    {"name": "logs", "command": "tail -f logs/app.log"},
]
create_agent_session("agent-work", tasks)
```

## Automation Scripts

### Session Recovery

```bash
#!/bin/bash
# recover-sessions.sh - Restore tmux sessions after reboot

SESSIONS_FILE="${HOME}/.tmux-sessions"

# Save current sessions
tmux-save() {
  tmux ls -F "#{session_name}" > "$SESSIONS_FILE"
  echo "Saved $(wc -l < "$SESSIONS_FILE") sessions"
}

# Restore saved sessions
tmux-restore() {
  while IFS= read -r session; do
    if ! tmux has-session -t "$session" 2>/dev/null; then
      tmux new -s "$session" -d
      echo "Restored: $session"
    fi
  done < "$SESSIONS_FILE"
}

# Auto-save on exit
trap tmux-save EXIT
```

### Workspace Templates

```bash
# workspace-openclaw.sh
tmux-has() { tmux has-session -t "$1" 2>/dev/null; }

create-openclaw-workspace() {
  local session="openclaw-dev"
  
  if tmux-has "$session"; then
    echo "Attaching to existing session..."
    tmux attach -t "$session"
    return
  fi
  
  # Create new session
  tmux new -s "$session" -d
  
  # Window 0: Main shell
  tmux rename-window -t "$session:0" 'shell'
  
  # Window 1: Remote logs
  tmux new-window -t "$session" -n 'logs'
  tmux send-keys -t "$session:1" 'ssh openclawmaster "tail -f /var/log/syslog"' C-m
  
  # Window 2: Monitoring
  tmux new-window -t "$session" -n 'monitor'
  tmux send-keys -t "$session:2" 'watch -n 5 "tailscale status | grep openclaw"' C-m
  
  # Window 3: Remote shell
  tmux new-window -t "$session" -n 'remote'
  tmux send-keys -t "$session:3" 'ssh openclawmaster' C-m
  
  tmux attach -t "$session"
}

create-openclaw-workspace
```

## Best Practices

1. **Session naming**: Use descriptive names (project-env, task-name)
2. **Window naming**: Rename windows to reflect purpose (Ctrl+b ,)
3. **Pane synchronization**: Use for running same command on multiple nodes
4. **Remote work**: Always use tmux on remote to survive disconnects
5. **Session persistence**: Save session configs for quick recreation

## Integration with AI Tools

- Use tmux for long-running agent tasks
- Spawn subagents in separate tmux windows
- Monitor agent output with `tmux capture-pane`
- Coordinate multi-node work via synchronized panes
