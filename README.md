# Kilo Code Configuration - Multi-Machine Setup

This repository contains the complete configuration and documentation for Kilo Code setup across multiple machines (IM, TO, WS, MI, MB).

## Overview

- **Purpose**: Automatic API key loading for VS Code/Kilo Code across all machines
- **Method**: Dotfiles management via chezmoi + environment persistence
- **Machines**: IM (iMac), TO (Linux), WS (Linux), MI (Mac Mini), MB (pending)
- **Status**: ✅ Complete for IM, TO, WS, MI | ❌ MB unreachable

## Files

| File | Description |
|------|-------------|
| `.env.kilo.secrets.backup` | Full backup of API keys (local only, not in git) |
| `.env.kilo.secrets.template` | Redacted template with placeholders (safe to commit) |
| `README.md` | This file - setup overview |
| `MACHINE_SETUP.md` | Machine-specific setup instructions |
| `MODEL_RECOMMENDATIONS.md` | Model recommendations per KC mode |
| `FALLBACK_RECOVERY.md` | Recovery procedures for common issues |

## Quick Start

### 1. Copy API Keys Template

```bash
cp .env.kilo.secrets.template ~/.env.kilo.secrets
# Edit ~/.env.kilo.secrets with your actual API keys
```

### 2. Sync to All Machines

```bash
scp ~/.env.kilo.secrets gerardo@192.168.1.169:~/.env.kilo.secrets  # MI
scp ~/.env.kilo.secrets eviwork@192.168.1.180:~/.env.kilo.secrets  # TO
scp ~/.env.kilo.secrets eviwork@192.168.1.160:~/.env.kilo.secrets  # WS
```

### 3. Apply Chezmoi Configuration

```bash
# On each machine
chezmoi apply --force --no-tty
```

### 4. Restart VS Code

Quit VS Code completely (Cmd+Q on macOS) and reopen. The environment variables will be loaded automatically.

## Verification

### macOS (IM, MI)

```bash
# Check LaunchAgent is loaded
launchctl list | grep kilo

# Verify env vars in launchd
launchctl print gui/$(id -u) | grep -E "REQUESTY_API_KEY|NVIDIA_API_KEY|TYPESAFE_API_KEY"
```

### Linux (TO, WS)

```bash
# Check systemd service
systemctl --user status kilo-env.service

# Verify env vars
systemctl --user show-environment | grep -E "REQUESTY_API_KEY|NVIDIA_API_KEY|TYPESAFE_API_KEY"
```

## Architecture

```
~/.env.kilo.secrets (local, not synced)
         ↓
chezmoi template (dot_env.kilo.tmpl)
         ↓
~/.config/kilo/kilo.jsonc (with {env:VAR} placeholders)
         ↓
Environment persistence:
  - macOS: LaunchAgent → launchctl setenv
  - Linux: systemd user service → systemctl import-environment
         ↓
VS Code / Kilo Code reads {env:VAR} and resolves to actual keys
```

### Linux-Specific: VS Code Wrapper

On Linux, GUI apps launched from the desktop don't inherit systemd user environment. A wrapper script ensures API keys are available:

```
~/.local/bin/code-with-env (wrapper script)
         ↓
Sources ~/.env.kilo.secrets
         ↓
Launches /usr/share/code/code with env vars
         ↓
~/.local/share/applications/code.desktop (uses wrapper)
```

## Troubleshooting

See [FALLBACK_RECOVERY.md](FALLBACK_RECOVERY.md) for detailed recovery procedures.

## Model Recommendations

See [MODEL_RECOMMENDATIONS.md](MODEL_RECOMMENDATIONS.md) for optimal model selection per KC mode.

## Machine Status

| Machine | Status | Mechanism | Verified |
|---------|--------|-----------|----------|
| **IM** (iMac) | ✅ Active | LaunchAgent | Env vars in `launchctl getenv` |
| **MI** (Mac Mini) | ✅ Active | LaunchAgent | Env vars in `launchctl print` |
| **TO** (192.168.1.180) | ✅ Active | systemd service | Env vars in `systemctl --user show-environment` |
| **WS** (192.168.1.160) | ✅ Active | systemd service | Env vars in `systemctl --user show-environment` |
| **MB** (192.168.1.161) | ❌ Blocked | Pending | SSH unreachable |

## Last Updated

2026-09-19
