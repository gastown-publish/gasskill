---
name: openclaw-gateway
description: Manage and troubleshoot the OpenClawMaster Tailscale gateway. Use for monitoring connectivity, diagnosing connection issues, managing gateway services, SSH access, remote commands, and automated health checks. Integrates with Tailscale, SSH, and Tmux for persistent remote sessions.
---

# OpenClaw Gateway Skill

Manage the `openclawmaster` (100.103.11.116) Tailscale gateway - your bridge to the OpenClaw infrastructure.

## Quick Status

```bash
# Check if gateway is reachable
ping -c 3 100.103.11.116

# Check Tailscale status
tailscale status | grep openclawmaster

# Full diagnostic
openclaw-doctor
```

## Gateway Information

| Property | Value |
|----------|-------|
| **Tailscale IP** | 100.103.11.116 |
| **Hostname** | openclawmaster |
| **Tailnet** | taile8dc37.ts.net |
| **OS** | Linux |
| **Services** | SSH (22), Tailscale |

## Connectivity Checks

### Basic Health

```bash
# Quick ping test
alias openclaw-ping='ping -c 3 100.103.11.116'

# Full connectivity test
openclaw-check() {
  echo "=== OpenClawMaster Health Check ==="
  echo ""
  
  # Ping
  echo -n "Ping: "
  if ping -c 3 -W 5 100.103.11.116 &>/dev/null; then
    echo "✓ OK"
  else
    echo "✗ FAIL"
  fi
  
  # Tailscale
  echo -n "Tailscale: "
  if tailscale status | grep -q "openclawmaster.*active"; then
    echo "✓ Active"
  elif tailscale status | grep -q "openclawmaster"; then
    echo "⚠ Connected but idle"
  else
    echo "✗ Not found"
  fi
  
  # SSH
  echo -n "SSH (port 22): "
  if timeout 5 bash -c "</dev/tcp/100.103.11.116/22" 2>/dev/null; then
    echo "✓ Reachable"
  else
    echo "✗ Unreachable"
  fi
  
  echo ""
  echo "Details:"
  tailscale status | grep openclawmaster || echo "  No Tailscale entry"
}
```

### Automated Monitoring

```bash
# Run the guardian monitor
openclaw-guardian-ctl start

# Check guardian status
openclaw-guardian-ctl status

# View logs
tail -f ~/.logs/openclaw-guardian.log
```

## SSH Access

### Direct SSH

```bash
# Via Tailscale IP
ssh user@100.103.11.116

# Via MagicDNS
ssh user@openclawmaster.taile8dc37.ts.net
```

### Persistent Sessions (Tmux)

```bash
# Create persistent remote workspace
openclaw-tmux() {
  local session="${1:-openclaw-work}"
  
  # Create local tmux session with remote windows
  tmux new -s "$session" -d 2>/dev/null || true
  
  # Window 0: Local gateway monitor
  tmux rename-window -t "$session:0" 'monitor'
  tmux send-keys -t "$session:0" 'watch -n 10 "tailscale status | grep openclaw"' C-m
  
  # Window 1: Remote logs
  tmux new-window -t "$session" -n 'logs'
  tmux send-keys -t "$session:1" 'ssh 100.103.11.116 "sudo journalctl -f"' C-m
  
  # Window 2: Remote shell
  tmux new-window -t "$session" -n 'shell'
  tmux send-keys -t "$session:2" 'ssh 100.103.11.116' C-m
  
  tmux attach -t "$session"
}
```

## Remote Commands

### Execute Without Interactive Shell

```bash
# Run command on openclawmaster
openclaw-run() {
  ssh 100.103.11.116 "$@"
}

# Examples
openclaw-run uptime
openclaw-run "df -h"
openclaw-run "docker ps"
openclaw-run "systemctl status tailscaled"
```

### File Transfer

```bash
# Copy to openclawmaster
openclaw-put() {
  local src="$1"
  local dest="${2:-.}"
  scp "$src" "100.103.11.116:$dest"
}

# Copy from openclawmaster
openclaw-get() {
  local src="$1"
  local dest="${2:-.}"
  scp "100.103.11.116:$src" "$dest"
}
```

## Service Management

### Tailscale on OpenClawMaster

```bash
# Check tailscale status on remote
openclaw-run "sudo tailscale status"

# Restart tailscale if needed
openclaw-run "sudo systemctl restart tailscaled"

# Check tailscale logs
openclaw-run "sudo journalctl -u tailscaled -f"
```

### Gateway as Exit Node

```bash
# Advertise exit node (run on openclawmaster)
openclaw-run "sudo tailscale up --advertise-exit-node"

# Enable in admin panel, then use from local:
tailscale up --exit-node=100.103.11.116

# Disable exit node
tailscale up --exit-node=
```

### Subnet Routing

```bash
# Advertise routes (run on openclawmaster)
openclaw-run "sudo tailscale up --advertise-routes=192.168.1.0/24,10.0.0.0/24"

# Enable routes in admin console, then local can reach those subnets
```

## Troubleshooting

### Connection Issues

```bash
openclaw-fix() {
  echo "Attempting to fix openclawmaster connection..."
  
  # 1. Check local tailscale
  echo "1. Checking local Tailscale..."
  if ! tailscale status &>/dev/null; then
    echo "   Local Tailscale not running! Start it:"
    echo "   sudo tailscale up"
    return 1
  fi
  
  # 2. Try to ping
  echo "2. Testing ping..."
  if ! ping -c 1 -W 3 100.103.11.116 &>/dev/null; then
    echo "   Ping failed. Checking Tailscale status..."
    tailscale status | grep openclawmaster || echo "   openclawmaster not in peer list"
    
    echo ""
    echo "Possible fixes:"
    echo "  - Check if openclawmaster is online in Tailscale admin console"
    echo "  - SSH to openclawmaster via alternative method and check tailscaled"
    echo "  - Restart local Tailscale: sudo tailscale down && sudo tailscale up"
  fi
  
  # 3. Check SSH
  echo "3. Testing SSH port..."
  if ! timeout 5 bash -c "</dev/tcp/100.103.11.116/22" 2>/dev/null; then
    echo "   SSH port not responding"
    echo "   - Check if openclawmaster is powered on"
    echo "   - Check firewall on openclawmaster"
  fi
}
```

### Common Issues

| Issue | Diagnosis | Fix |
|-------|-----------|-----|
| "Destination unreachable" | Network path issue | Check Tailscale on both ends |
| "Connection refused" | SSH not running | `openclaw-run "sudo systemctl start ssh"` |
| "Permission denied" | Auth issue | Check SSH keys, `ssh-add -l` |
| High latency | Relay connection | Check direct connectivity, restart tailscale |

### Recovery Procedures

```bash
# If openclawmaster is completely unreachable via Tailscale:
# 1. Check if it has internet access (may need alternative access method)
# 2. Restart tailscale on openclawmaster
# 3. If all else fails, check hardware/power

# Emergency restart (if you have IPMI/IDRAC/etc access)
# openclaw-run "sudo reboot"  # Won't work if tailscale is down
```

## Automation

### Guardian Integration

The guardian monitor (`~/.local/bin/openclaw-guardian.sh`) provides:

- **Continuous monitoring**: Every 60 seconds
- **Health checks**: Ping, Tailscale status, SSH port
- **Auto-fix**: Triggers Kimi CLI after 3 failures
- **Logging**: `~/.logs/openclaw-guardian.log`

```bash
# Start guardian
openclaw-guardian-ctl start

# Stop guardian
openclaw-guardian-ctl stop

# Check status
openclaw-guardian-ctl status
```

### Subagent Integration

```python
# Use Task tool to manage openclawmaster
Task({
  subagent_type: "Bash",
  description: "Check openclawmaster services",
  prompt: """
    SSH into openclawmaster (100.103.11.116) and check:
    1. System uptime
    2. Disk usage (df -h)
    3. Running docker containers
    4. Tailscale status
    
    Return a summary report.
  """
})
```

## Best Practices

1. **Always use tmux** for long-running remote sessions
2. **Monitor via guardian** for proactive issue detection
3. **Use MagicDNS** names instead of IPs when possible
4. **Set up SSH keys** for passwordless access
5. **Keep a backup access method** (console/IPMI) for emergencies

## Related Skills

- `tmux-manager` - For persistent remote sessions
- `ssh-expert` - Advanced SSH configuration
- `network-debug` - Network troubleshooting
- `tailscale-admin` - Tailscale administration
