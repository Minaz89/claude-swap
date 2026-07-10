# cswap troubleshooting

Platform-accurate answers to the common "why isn't this working?" questions. Verified against
`cswap` v0.18.1 (2026-07-10); anything not shown in the live `--help` is marked `[unverified]`
— re-derive from `cswap --help`.

---

## Windows: `cswap list` / `tui` crash with `UnicodeEncodeError` (force UTF-8)

On Windows, `cswap list` / `tui` / `watch` render Unicode box-drawing characters
(`├ └ ●`) for the usage bars. In a console on the legacy **cp1252** code page (the
Windows PowerShell 5.1 default), Python crashes mid-render:

```
UnicodeEncodeError: 'charmap' codec can't encode character '├' ...
```

This is a console-encoding bug, **not** a credentials or account problem — the account data
is intact (the crash happens while *printing* it). Force Python to UTF-8 before running cswap:

```powershell
$env:PYTHONUTF8 = '1'   # per-session; add to your PowerShell $PROFILE to make it permanent
cswap list
```

(`$env:PYTHONIOENCODING = 'utf-8'` or `chcp 65001` work too.) **When operating cswap on
Windows, always set `$env:PYTHONUTF8='1'` first** so read-only commands don't die on the
box characters. Reproduced on cswap v0.18.1, 2026-07-10.

## When does a switch take effect?

The answer depends on the platform — do **not** claim a blanket "requires restart," and do
**not** imply a seamless zero-latency in-place swap (SKILL.md rule 5).

| Platform | When the switched account takes effect |
|----------|----------------------------------------|
| **Windows** | Automatically — **no restart needed.** Claude Code picks up the new account on its next credential read. |
| **Linux / WSL** | Automatically — **no restart needed.** |
| **macOS** | Keychain cache is ~**30 seconds**. For instant application, **restart Claude Code.** |

## Token refresh vs. account swap

These are different operations and do not collide:

- A **token refresh** is the normal background re-issue of an OAuth token for an account that
  is still valid.
- An **account swap** is switching which account Claude Code reads credentials from.
- cswap **holds the credential locks**, so a swap never races or collides with an in-flight
  refresh. You do not need to stop Claude Code to switch.

`[unverified]` exact lock granularity — confirm via `cswap --help` / the README if it matters
to a debugging session.

## Expired token vs. dead refresh token

**Tip:** see each account's OAuth token expiry state with `cswap --list --token-status`
(read-only) — use it to tell a token that merely needs a refresh from one that is actually dead.

- **Expired token** → re-register the account: `cswap add --slot N` (state-changing — confirm
  first). Log into that account in Claude Code first, then run `add`.
- **Dead refresh token** (the account can no longer refresh at all) → cswap **quarantines**
  that account until you re-login and re-add it. It will not be rotated onto automatically.
  Re-register with `cswap add --slot N` after logging in.

## Windows: no `--share-history` yet

`cswap run --share-history` is **not supported on Windows** (macOS/Linux only). On Windows,
omit the flag — each `run` session is history-isolated.

## Per-platform data & config paths

If a file or credential "isn't where you expect," check the right platform location:

| Platform | Credentials | Config dir |
|----------|-------------|------------|
| **Windows** | `~/.claude-swap-backup/credentials/` | `~/.claude-swap-backup/` |
| **macOS** | system **Keychain** (not a file) | `~/.claude-swap-backup/` |
| **Linux / WSL** | `${XDG_DATA_HOME:-~/.local/share}/claude-swap/credentials/` | `${XDG_DATA_HOME:-~/.local/share}/claude-swap/` *(NOT `~/.claude-swap-backup/`)* |

Under the config dir: sessions `{config-dir}/sessions/`, tool prefs
`{config-dir}/settings.json`, auto-switch state `{config-dir}/autoswitch_state.json`.

On macOS, credentials live in the **Keychain**, not on disk — so a "missing credentials file"
on macOS is expected. On Windows the backup root is `~/.claude-swap-backup/`. Read settings with
bare `cswap config` (read-only). (`[unverified]` `cswap config path` is **not shown in the
v0.18.1 `--help`** — do not assume it exists.)

## API-key accounts are not rotated onto by default

Accounts registered as API keys (`sk-ant-...`) are **excluded from automatic rotation** unless
you explicitly opt in. To include them:

```powershell
# State-changing — confirm first. Allow auto/switch to rotate onto API-key accounts:
cswap auto --include-api-key-accounts
cswap switch --strategy best --include-api-key-accounts
```

**Never print the `sk-ant-...` key itself** — refer to these accounts by index/email only
(SKILL.md rule 3).

## Still stuck?

- Confirm the live surface: `cswap --help` and `cswap <sub> --help`.
- Read state directly: `cswap list` (usage), `cswap status` (current account/switch state), `cswap config` (all settings, read-only).
- Never guess a usage number, threshold, or reset time — if cswap doesn't surface it, say so.
