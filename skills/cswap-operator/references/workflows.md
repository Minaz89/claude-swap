# cswap workflows

Short, PowerShell-first recipes. State-changing steps are called out — confirm with the user
before running them (SKILL.md rule 4). Read-only steps are safe to run anytime. Verified
against `cswap` v0.18.1 (2026-07-10); confirm the live surface with `cswap --help` after any
upgrade.

---

## 1. First-time setup

Install cswap, then register each already-logged-in account into its own slot.

```powershell
# 1. Install (preferred) — let the user run or confirm this (rule 6)
uv tool install claude-swap

# 2. In Claude Code, log into account #1, then register it
cswap add --slot 1

# 3. Log out of account #1 and into account #2 in Claude Code, then register it
cswap add --slot 2

# 4. Repeat per account (slot 3, 4, ...). Then verify:
cswap list
```

Notes:
- `add` registers **the currently-logged-in Claude Code account** into the slot — switch the
  login in Claude Code between each `add`.
- After setup, `cswap list` shows every account with its usage dashboard.

## 2. Daily check — usage → switch if low

Read usage, decide, then switch (state-changing — confirm first).

```powershell
# Read-only: see per-account 5h + 7d usage and reset times
cswap list

# Read-only: read the current account/switch state
cswap status
```

If an account is near its limit and you want to switch:

```powershell
# State-changing — confirm with the user first.
# Switch to a specific account by number (or email):
cswap switch 2

# …or switch to the account with the most remaining 5h/7d quota:
cswap switch --strategy best
```

After a successful switch, state the **platform-correct effect** (rule 5):
- **Windows / Linux**: the new account is picked up automatically — **no restart needed**.
- **macOS**: Keychain cache is ~30 s, or restart Claude Code for instant application.

## 3. Enable + tune auto-rotation

Preview first (read-only), then enable (state-changing), then tune the threshold.

```powershell
# Read-only: preview what auto-rotation would do right now
cswap auto --dry-run

# State-changing — confirm first. Run continuously with the default 90% threshold:
cswap auto

# …or set a custom threshold at run time:
cswap auto --threshold 85
```

Tune the persistent trigger threshold (state-changing — confirm first):

```powershell
# Set the autoswitch threshold (default is 90)
cswap config set autoswitch.threshold 85

# Read-only: verify it — bare `config` shows all keys (threshold included)
cswap config
```

All `autoswitch.*` settings use the same `cswap config set autoswitch.<key> <value>` pattern.
The real keys and their defaults:

| Key | Default |
|-----|---------|
| `autoswitch.threshold` | `90` |
| `autoswitch.intervalSeconds` | `60` |
| `autoswitch.cooldownSeconds` | `300` |
| `autoswitch.hysteresisPct` | `10` |
| `autoswitch.strategy` | `best` |
| `autoswitch.includeApiKeyAccounts` | `false` |
| `autoswitch.unhealthyTicks` | `3` |

## 4. Parallel sessions with `run`

Launch a Claude Code invocation **pinned to one account** in an isolated session, so you can
run several accounts at once.

```powershell
# State-changing (launches a process). Pin a session to account 2 and pass args through:
cswap run 2 -- --print "hello from account 2"

# Pin by email instead of number:
cswap run you@example.com -- <claude args>
```

> **Windows note:** `--share-history` is **not supported on Windows yet** (macOS/Linux only).
> On Windows, omit it; each `run` session is history-isolated.

## 5. Backup / restore / migrate

Export a full backup, then import it on another machine (or to restore). **`import` can
overwrite existing accounts/backups — confirm before running, and use `--force` deliberately.**

```powershell
# Read-only-ish (writes a file you name): full backup. `--full` includes the entire
# ~/.claude.json (the default export is oauthAccount only).
cswap export --full cswap-backup.json

# Export a single account instead:
cswap export --account 2 slot2.json

# State-changing — confirm first. Restore (overwrites matching entries without --force):
cswap import cswap-backup.json

# Force-overwrite existing entries on import (state-changing — confirm explicitly):
cswap import cswap-backup.json --force
```

## 6. Remove an account

Remove a registered account. **Hard to reverse** (the account must be re-added) — confirm
first (rule 4).

```powershell
# State-changing — confirm with the user before running
cswap remove 2
```

Verify afterward with `cswap list` (read-only).
