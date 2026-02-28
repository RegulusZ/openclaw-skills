# Troubleshooting

## Common Issues

### Sync not starting

**Check:**
```bash
sync-status
```

**If daemon not running:**
```bash
sync-daemon start
```

**If config missing:**
```bash
sync-init --device-name <your-device>
```

### Git authentication failed

**Problem:** `git push` asks for password or fails with auth error

**Solution:**
1. Use SSH key auth: `git remote set-url origin git@github.com:USER/REPO.git`
2. Ensure key is added: `ssh-add -l`
3. Test: `ssh -T git@github.com`

### Conflict loop

**Problem:** Keep getting conflicts after resolving

**Cause:** Both devices modifying same file simultaneously

**Solution:**
- Use device-prefixed memory files (auto-created)
- Avoid editing shared files (SOUL.md, USER.md) on multiple devices at once
- Increase sync frequency if needed

### Large repo / slow sync

**Problem:** Sync takes too long

**Check:**
```bash
cd ~/openclaw-sync
du -sh .
git count-objects -vH
```

**Solution:**
- Add large files to `.gitignore`
- Clean old logs: `git log --oneline | wc -l`
- Consider git gc: `git gc --aggressive`

### Daemon stops unexpectedly

**Check logs:**
```bash
tail ~/.openclaw/sync-daemon.log
```

**Common causes:**
- Repo deleted or moved
- Git credentials expired
- Disk full

### Device name collision

**Problem:** Two devices using same name

**Solution:**
```bash
# Edit config
nano ~/.config/openclaw/sync-config.yaml
# Change device_name, then re-init
sync-init --device-name NEWNAME
```

## Debug Mode

Enable verbose logging:
```bash
export SYNC_DEBUG=1
sync-daemon restart
```

## Manual Recovery

If everything breaks:

```bash
# 1. Stop daemon
sync-daemon stop

# 2. Backup local changes
cp -r ~/openclaw-sync ~/openclaw-sync-backup-$(date +%s)

# 3. Reset to remote
cd ~/openclaw-sync
git fetch origin
git reset --hard origin/HEAD

# 4. Re-apply local changes manually

# 5. Restart
sync-daemon start
```
