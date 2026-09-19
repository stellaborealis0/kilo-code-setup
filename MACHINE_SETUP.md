# Machine Setup Instructions

## Prerequisites

- chezmoi v2.72.2+ installed on all machines
- `~/.env.kilo.secrets` file with API keys (local, not in git)
- SSH access between machines

## IM (iMac) - macOS

### Status: ✅ Complete

```bash
# Verify LaunchAgent is loaded
launchctl list | grep kilo

# Expected output:
# -	0	com.gerardo.kilo.env-loader
```

### Manual Setup (if needed)

```bash
# Load LaunchAgent
launchctl load ~/Library/LaunchAgents/com.gerardo.kilo.env-loader.plist

# Verify env vars
launchctl print gui/$(id -u) | grep -E "REQUESTY_API_KEY|NVIDIA_API_KEY|TYPESAFE_API_KEY"
```

---

## MI (Mac Mini M4) - macOS

### Status: ✅ Complete

Same as IM. LaunchAgent is loaded and env vars are verified in `launchctl print gui/$(id -u)`.

### Manual Setup (if needed)

```bash
# Same as IM
launchctl load ~/Library/LaunchAgents/com.gerardo.kilo.env-loader.plist
```

---

## TO (192.168.1.180) - Ubuntu Linux

### Status: ✅ Complete

```bash
# Verify systemd service
systemctl --user status kilo-env.service

# Expected: Active: active (exited)
```

### Manual Setup (if needed)

```bash
# Enable and start service
systemctl --user daemon-reload
systemctl --user enable kilo-env.service
systemctl --user restart kilo-env.service

# Verify env vars
systemctl --user show-environment | grep -E "REQUESTY_API_KEY|NVIDIA_API_KEY|TYPESAFE_API_KEY"
```

---

## WS (192.168.1.160) - Ubuntu Linux

### Status: ✅ Complete

Same as TO. Systemd service is enabled and active.

### Manual Setup (if needed)

```bash
# Same as TO
systemctl --user enable kilo-env.service
systemctl --user restart kilo-env.service
```

---

## MB (192.168.1.161 / 100.108.248.34) - Tailscale

### Status: ❌ Blocked

SSH connection refused on LAN IP, Tailscale IP timed out. Cannot deploy until network connectivity is restored.

### Recovery Steps (when connectivity is restored)

```bash
# 1. SSH into MB
ssh gerardo@192.168.1.161  # or 192.168.1.169 (MI) then SSH from there

# 2. Install chezmoi if not present
curl -fsSL https://chezmoi.io/get | sh

# 3. Initialize chezmoi
chezmoi init https://github.com/stellaborealis0/dotfiles.git

# 4. Apply configuration
chezmoi apply --force --no-tty

# 5. Load LaunchAgent (macOS) or enable systemd service (Linux)
# For macOS:
launchctl load ~/Library/LaunchAgents/com.gerardo.kilo.env-loader.plist

# For Linux:
systemctl --user enable kilo-env.service
systemctl --user restart kilo-env.service

# 6. Restart VS Code
# Quit VS Code completely and reopen
```

---

## Common Issues

### Issue: Env vars not available in VS Code

**Solution**: Restart VS Code completely (Cmd+Q on macOS, close window on Linux).

### Issue: LaunchAgent not loading on macOS

**Solution**:
```bash
# Unload and reload
launchctl unload ~/Library/LaunchAgents/com.gerardo.kilo.env-loader.plist
launchctl load ~/Library/LaunchAgents/com.gerardo.kilo.env-loader.plist
```

### Issue: Systemd service not running on Linux

**Solution**:
```bash
# Check status
systemctl --user status kilo-env.service

# Restart if needed
systemctl --user restart kilo-env.service
```

### Issue: chezmoi apply fails

**Solution**:
```bash
# Force apply (overwrites local changes)
chezmoi apply --force --no-tty

# Or reset to repo state
chezmoi cd
git reset --hard origin/master
chezmoi apply
```

---

## Verification Commands

### macOS

```bash
# Check LaunchAgent
launchctl list | grep kilo

# Check env vars in launchd
launchctl print gui/$(id -u) | grep -E "REQUESTY_API_KEY|NVIDIA_API_KEY|TYPESAFE_API_KEY"

# Test env var availability
launchctl getenv NVIDIA_API_KEY
```

### Linux

```bash
# Check systemd service
systemctl --user status kilo-env.service

# Check env vars
systemctl --user show-environment | grep -E "REQUESTY_API_KEY|NVIDIA_API_KEY|TYPESAFE_API_KEY"

# Test env var availability
echo $NVIDIA_API_KEY
```

---

## Network Requirements

- **IM**: Direct access to all machines
- **TO/WS/MI**: SSH access from IM
- **MB**: Tailscale connection from IM (currently unreachable)

## API Keys Location

- **Local**: `~/.env.kilo.secrets` (not in git, machine-specific)
- **Template**: `.env.kilo.secrets.template` (redacted, safe to commit)
- **Config**: `~/.config/kilo/kilo.jsonc` (uses `{env:VAR}` placeholders)
