# Fallback Recovery Procedures

## Overview

This document provides step-by-step recovery procedures for common issues with the Kilo Code configuration across multiple machines.

---

## Quick Recovery Checklist

| Issue | Quick Fix |
|-------|-----------|
| Env vars not in VS Code | Restart VS Code (Cmd+Q / close window) |
| LaunchAgent not loaded | `launchctl load ~/Library/LaunchAgents/com.gerardo.kilo.env-loader.plist` |
| Systemd service not running | `systemctl --user restart kilo-env.service` |
| chezmoi out of sync | `chezmoi apply --force --no-tty` |
| API keys not working | Check `~/.env.kilo.secrets` exists and has valid keys |
| Model not responding | Switch to fallback model (see below) |

---

## Issue 1: Environment Variables Not Available in VS Code

### Symptoms
- Kilo Code shows "API key not found" errors
- Models fail to load
- `{env:VAR}` placeholders not resolved

### Diagnosis

**macOS (IM, MI)**:
```bash
launchctl getenv NVIDIA_API_KEY
# Should return the API key value
```

**Linux (TO, WS)**:
```bash
systemctl --user show-environment | grep NVIDIA_API_KEY
# Should show NVIDIA_API_KEY=value
```

### Recovery Steps

1. **Restart VS Code** (most common fix)
   - macOS: Cmd+Q to fully quit, then reopen
   - Linux: Close window completely, then reopen

2. **If restart doesn't work, reload environment loader**:

   **macOS**:
   ```bash
   launchctl unload ~/Library/LaunchAgents/com.gerardo.kilo.env-loader.plist
   launchctl load ~/Library/LaunchAgents/com.gerardo.kilo.env-loader.plist
   ```

   **Linux**:
   ```bash
   systemctl --user restart kilo-env.service
   ```

3. **Verify env vars are loaded**:
   ```bash
   # macOS
   launchctl print gui/$(id -u) | grep NVIDIA_API_KEY
   
   # Linux
   systemctl --user show-environment | grep NVIDIA_API_KEY
   ```

4. **Restart VS Code again** after reloading

---

## Issue 2: chezmoi Configuration Out of Sync

### Symptoms
- `chezmoi status` shows modified files
- Configuration changes not applied
- Errors during `chezmoi apply`

### Diagnosis

```bash
chezmoi status
# Should show empty output if everything is in sync
```

### Recovery Steps

1. **Pull latest changes**:
   ```bash
   chezmoi cd
   git pull origin master
   ```

2. **Force apply** (overwrites local changes):
   ```bash
   chezmoi apply --force --no-tty
   ```

3. **If conflicts persist, reset to repo state**:
   ```bash
   chezmoi cd
   git reset --hard origin/master
   chezmoi apply --force --no-tty
   ```

4. **Verify**:
   ```bash
   chezmoi status
   # Should be empty
   ```

---

## Issue 3: API Keys Missing or Invalid

### Symptoms
- "Invalid API key" errors
- Models fail to authenticate
- 401/403 errors from providers

### Diagnosis

```bash
# Check if secrets file exists
ls -la ~/.env.kilo.secrets

# Source and verify keys
source ~/.env.kilo.secrets
echo $NVIDIA_API_KEY | cut -c1-20
# Should show first 20 chars of key
```

### Recovery Steps

1. **Verify secrets file exists**:
   ```bash
   ls -la ~/.env.kilo.secrets
   ```

2. **If missing, copy from backup**:
   ```bash
   # From another machine
   scp gerardo@IM_IP:~/.env.kilo.secrets ~/.env.kilo.secrets
   chmod 600 ~/.env.kilo.secrets
   ```

3. **If keys are invalid, regenerate**:
   - NVIDIA: https://build.nvidia.com/settings/api-keys
   - DashScope: https://bailian.console.aliyun.com/ → API-KEY管理
   - Requesty: https://app.requesty.ai/settings/keys

4. **Update secrets file**:
   ```bash
   nano ~/.env.kilo.secrets
   # Update the invalid keys
   ```

5. **Reload environment**:
   ```bash
   source ~/.env.kilo.secrets
   
   # macOS
   launchctl setenv NVIDIA_API_KEY "$NVIDIA_API_KEY"
   launchctl setenv REQUESTY_API_KEY "$REQUESTY_API_KEY"
   launchctl setenv TYPESAFE_API_KEY "$TYPESAFE_API_KEY"
   
   # Linux
   systemctl --user restart kilo-env.service
   ```

6. **Restart VS Code**

---

## Issue 4: Model Not Responding / Timeout

### Symptoms
- Kilo Code hangs or times out
- "Model unavailable" errors
- Slow responses

### Recovery Steps

1. **Switch to fallback model**:

   Edit `~/.config/kilo/kilo.jsonc`:
   ```json
   {
     "model": "requesty/bedrock/claude-sonnet-4-6@eu-central-1",
     "small_model": "nvidia/meta/llama-3.1-8b-instruct"
   }
   ```

2. **Restart VS Code**

3. **If still failing, try NVIDIA directly**:
   ```json
   {
     "model": "nvidia/meta/llama-3.1-70b-instruct",
     "small_model": "nvidia/meta/llama-3.1-8b-instruct"
   }
   ```

4. **Check provider status**:
   ```bash
   # Test NVIDIA
   curl -H "Authorization: Bearer $NVIDIA_API_KEY" \
     https://integrate.api.nvidia.com/v1/models | head -20
   
   # Test DashScope
   curl -H "Authorization: Bearer $DASHSCOPE_API_KEY" \
     https://coding-intl.dashscope.aliyuncs.com/v1/models | head -20
   ```

---

## Issue 5: Machine Unreachable (MB)

### Symptoms
- SSH connection refused
- Tailscale IP timed out
- Cannot deploy configuration

### Recovery Steps

1. **Check network connectivity**:
   ```bash
   # From IM
   ping 192.168.1.161  # LAN IP
   ping 100.108.248.34  # Tailscale IP
   ```

2. **If LAN works but Tailscale doesn't**:
   ```bash
   # SSH via LAN
   ssh gerardo@192.168.1.161
   ```

3. **If neither works, check Tailscale**:
   ```bash
   # On another machine
   tailscale status
   # Check if MB appears
   ```

4. **When connectivity is restored**:
   ```bash
   # SSH into MB
   ssh gerardo@192.168.1.161
   
   # Install chezmoi if needed
   curl -fsSL https://chezmoi.io/get | sh
   
   # Initialize and apply
   chezmoi init https://github.com/stellaborealis0/dotfiles.git
   chezmoi apply --force --no-tty
   
   # Load environment
   launchctl load ~/Library/LaunchAgents/com.gerardo.kilo.env-loader.plist  # macOS
   # OR
   systemctl --user enable kilo-env.service  # Linux
   systemctl --user restart kilo-env.service
   
   # Restart VS Code
   ```

---

## Issue 6: VS Code Extension Not Loading

### Symptoms
- Kilo Code extension not appearing
- Commands not available in Command Palette
- Extension crashes on startup

### Recovery Steps

1. **Reload VS Code window**:
   - Cmd+Shift+P → "Developer: Reload Window"

2. **If still not working, reinstall extension**:
   ```bash
   # Uninstall
   code --uninstall-extension kilocode.kilo-code
   
   # Reinstall
   code --install-extension kilocode.kilo-code
   ```

3. **Check extension logs**:
   - Help → Toggle Developer Tools → Console tab

4. **Clear extension cache**:
   ```bash
   rm -rf ~/.vscode/extensions/kilocode.kilo-code-*
   # Reinstall extension
   ```

---

## Issue 7: Configuration File Corrupted

### Symptoms
- JSON parsing errors
- `kilo.jsonc` syntax errors
- Config not loading

### Recovery Steps

1. **Backup current config**:
   ```bash
   cp ~/.config/kilo/kilo.jsonc ~/.config/kilo/kilo.jsonc.broken
   ```

2. **Reset to chezmoi template**:
   ```bash
   chezmoi apply ~/.config/kilo/kilo.jsonc --force
   ```

3. **Or manually recreate from template**:
   ```bash
   chezmoi cd
   cat private_dot_config/kilo/kilo.jsonc.tmpl | \
     sed 's/{{.*}}/alibaba-coding-plan\/qwen3.7-plus/g' > \
     ~/.config/kilo/kilo.jsonc
   ```

4. **Validate JSON**:
   ```bash
   python3 -m json.tool ~/.config/kilo/kilo.jsonc > /dev/null
   # Should succeed without errors
   ```

---

## Emergency Recovery (Complete Reset)

If all else fails, perform a complete reset:

### macOS (IM, MI)

```bash
# 1. Unload LaunchAgent
launchctl unload ~/Library/LaunchAgents/com.gerardo.kilo.env-loader.plist

# 2. Remove config
rm -rf ~/.config/kilo

# 3. Reset chezmoi
chezmoi cd
git reset --hard origin/master
chezmoi apply --force --no-tty

# 4. Reload LaunchAgent
launchctl load ~/Library/LaunchAgents/com.gerardo.kilo.env-loader.plist

# 5. Restart VS Code
```

### Linux (TO, WS)

```bash
# 1. Stop service
systemctl --user stop kilo-env.service

# 2. Remove config
rm -rf ~/.config/kilo

# 3. Reset chezmoi
chezmoi cd
git reset --hard origin/master
chezmoi apply --force --no-tty

# 4. Restart service
systemctl --user daemon-reload
systemctl --user restart kilo-env.service

# 5. Restart VS Code
```

---

## Verification After Recovery

After any recovery, verify:

```bash
# 1. Check env vars are loaded
# macOS
launchctl print gui/$(id -u) | grep -E "NVIDIA_API_KEY|REQUESTY_API_KEY"

# Linux
systemctl --user show-environment | grep -E "NVIDIA_API_KEY|REQUESTY_API_KEY"

# 2. Check config is valid
python3 -m json.tool ~/.config/kilo/kilo.jsonc > /dev/null && echo "Config OK"

# 3. Test model access
curl -H "Authorization: Bearer $NVIDIA_API_KEY" \
  https://integrate.api.nvidia.com/v1/models | grep -c "id" && echo "API OK"

# 4. Restart VS Code and test Kilo Code
```

---

## Contact / Support

If issues persist after following these procedures:

1. Check logs:
   - macOS: `/tmp/kilo_env_loader_stderr.log`
   - Linux: `journalctl --user -u kilo-env.service`

2. Verify all prerequisites:
   - API keys valid and not expired
   - Network connectivity to providers
   - VS Code extension up to date
   - chezmoi configuration in sync

3. Re-run verification commands from [MACHINE_SETUP.md](MACHINE_SETUP.md)

---

## Last Updated

2026-09-19
