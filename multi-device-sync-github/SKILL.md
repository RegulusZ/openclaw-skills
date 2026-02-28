---
name: multi-device-sync-github
description: Multi-device OpenClaw data synchronization using GitHub. Manages workspace data sync across multiple machines (Ubuntu, Mac, etc.) with automatic push on file changes, periodic pull, and conflict detection. Use when setting up or managing OpenClaw across multiple devices, configuring data synchronization, resolving sync conflicts, or adding new devices to the sync network.
---

# Multi-Device Sync via GitHub

Synchronize OpenClaw workspace data across multiple machines using a private GitHub repository.

## Features

- **Automatic push**: File changes trigger immediate git commit + push
- **Periodic pull**: Every 5 minutes, pull remote changes
- **Conflict detection**: Manual resolution required on conflicts
- **Multi-device support**: Each device uses distinct file prefixes
- **Selective sync**: Only essential data synced, logs/temp excluded

## Architecture

```
Device A (Ubuntu) ◄────► GitHub Repo ◄────► Device B (Mac)
       │                      │                    │
   auto-push              central              auto-push
  定时pull                 hub               定时pull
```

## Quick Start

### 1. Initialize Repository

On first device:
```bash
# Create private repo on GitHub, then:
git clone git@github.com:YOURNAME/openclaw-sync.git ~/openclaw-sync
cd ~/openclaw-sync

# Copy initial workspace files
cp -r ~/.openclaw/workspace/{SOUL.md,USER.md,MEMORY.md,memory,skills,TOOLS.md} .

# Setup git
./scripts/sync-init --device-name ubuntu
```

### 2. Configure OpenClaw

Edit `~/.config/openclaw/sync-config.yaml`:
```yaml
repo_url: "git@github.com:YOURNAME/openclaw-sync.git"
sync_interval_minutes: 5
device_name: "ubuntu"  # Change per device: macmini, laptop, etc.
conflict_strategy: "notify"
auto_pull_on_start: true

paths:
  sync:
    - "SOUL.md"
    - "USER.md"
    - "MEMORY.md"
    - "memory/"
    - "skills/"
    - "TOOLS.md"
  ignore:
    - "logs/"
    - "temp/"
    - "*.log"
```

### 3. Start Sync

```bash
# Manual start
./scripts/sync-daemon start

# Or via systemd
systemctl --user enable openclaw-sync
systemctl --user start openclaw-sync
```

## File Naming Convention

Memory files use device prefix to avoid conflicts:

```
memory/
├── ubuntu-2026-02-28.md      # Ubuntu device
├── macmini-2026-02-28.md     # Mac Mini device
└── laptop-2026-02-28.md      # Laptop device
```

Shared files (no prefix):
- `SOUL.md`
- `USER.md`
- `MEMORY.md`
- `TOOLS.md`

## Commands

| Command | Description |
|---------|-------------|
| `sync-init` | Initialize git repo and config |
| `sync-status` | Check sync status |
| `sync-now` | Immediate pull + push |
| `sync-pull` | Manual pull |
| `sync-push` | Manual push |
| `sync-resolve` | Interactive conflict resolution |
| `sync-daemon start/stop/restart` | Manage background sync |

## Conflict Resolution

When conflicts detected:

1. Notification sent via local Feishu bot
2. Auto-sync paused
3. Run `sync-resolve` to:
   - View conflicting files
   - Choose: keep-local / keep-remote / merge-manual
4. Resume sync

## Adding New Device

1. Clone repo: `git clone git@github.com:YOURNAME/openclaw-sync.git`
2. Run `./scripts/sync-init --device-name NEWNAME`
3. Update `device_name` in config
4. Start daemon: `./scripts/sync-daemon start`

## Troubleshooting

See [references/troubleshooting.md](references/troubleshooting.md) for common issues.
