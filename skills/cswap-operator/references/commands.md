# cswap command reference

> **Verified against `cswap` v0.18.1 (installed 2026-07-10); re-confirm with `cswap --help`
> after upgrades.** Most of the surface below is confirmed against the installed binary. A few
> items remain marked `[unverified]` because the live v0.18.1 `--help` does **not** show them —
> for those, re-derive from `cswap --help` / `cswap <sub> --help` and stop assuming. If the
> documented and live surfaces disagree, the **live `--help` output wins.**
>
> Upstream source: claude-swap — <https://github.com/realiti4/claude-swap>.

Each entry below tags the command **`read-only`** (safe to run without confirmation) or
**`state-changing`** (confirm with the user first — see SKILL.md rule 4).

**Aliases:** `ls` = `list`, `rm` = `remove`, `update` = `upgrade`.

**Global flags** (apply across subcommands): `--version`, `--debug`, `--token-status` (OAuth
token expiry state; use with `--list`), `--json` (use with `--list` / `--status` / `--switch` /
`--switch-to`), `--strategy {best,next-available}`, `--slot NUM`, `--email EMAIL`, `--account
NUM|EMAIL` (limit export to one account), `--force`, `--full` (include full `~/.claude.json` in
export; default: `oauthAccount` only). Bare `cswap` (no subcommand) opens the TUI dashboard.

---

## `add` — register the currently-logged-in account  ·  `state-changing`

Registers the Claude account you are **currently logged into** in Claude Code into a cswap
slot.

- Syntax: `cswap add --slot N`
- Flags: `--slot N` — the slot number to register the account into.
- Effect: writes a new account into cswap's credential store. Confirm before running.

## `add-token` — register an account from a raw token  ·  `state-changing`

Register an account directly from a token instead of the currently-logged-in session.

- Syntax: `cswap add-token [TOKEN|-] --slot N [--email EMAIL]`
- Token argument: `[TOKEN|-]` — pass the token inline, use `-` to read it from **stdin**, or
  omit it to be **prompted** interactively. The type is **auto-detected**: a setup token vs an
  `sk-ant-api...` API key.
- `--email EMAIL` — optional; defaults to `setup-token-{slot}@token.local` (or
  `api-key-{slot}@token.local` for API keys).
- `--slot N` — the slot number.
- **Never echo the token value** — see SKILL.md rule 3. Refer to the account by slot/email only.

## `list` — usage dashboard  ·  `read-only`

Per-account usage dashboard.

- Syntax: `cswap list [--json] [--token-status]`
- Flags: `--json` — machine-readable output. `--token-status` — global flag; show OAuth token
  expiry state alongside each account (use with `--list`).
- Shows **5-hour and 7-day usage windows and reset times, per account**. This is the primary
  source of usage numbers — always read from here, never invent (rule 2).

## `status` — current account/switch state  ·  `read-only`

Reads the current account/switch state — distinct from `list` (which is the usage dashboard).

- Syntax: `cswap status [--json]`
- Flags: `--json` — machine-readable output.
- `status` is a **separate command from `list`**. The grounding only confirms it exists, takes
  `--json`, and is separate from the `list` usage dashboard — it does **not** specify the
  output fields. Treat "what `status` actually prints" (active account, switch state, etc.) as
  `[unverified]` until you confirm via `cswap status --help`.

## `switch` — change the active account  ·  `state-changing`

One-shot change of the active account for subsequent Claude Code credential reads.

- Syntax: `cswap switch [account-number|email] [options]` — **bare `cswap switch` rotates to
  the next account**; `cswap switch <num|email>` switches to a specific one.
- Flags / options:
  - `[account-number|email]` — target a specific account.
  - `--strategy {best,next-available}` — pick the target by remaining 5h/7d quota. **`best`
    jumps to the account with the most headroom; `next-available` rotates to the next account,
    skipping any at their limit.**
  - `--force` — **two meanings**: overwrite existing accounts during `import`; and with
    `--switch-to`, activate the stored credentials **without backing up the current login
    first**. Use deliberately.
  - `--json` — machine-readable output.
  - `--include-api-key-accounts` — allow rotating onto `sk-ant-...` API-key accounts (off by
    default — see troubleshooting).
- Confirm before running. After it succeeds, state the **platform-correct "when it takes
  effect"** (see troubleshooting): Windows/Linux auto-pickup; macOS ~30 s or restart.

## `auto` — hands-off auto-rotation  ·  `state-changing` (except `--dry-run`)

A **foreground polling loop by default** — watches usage and switches automatically when the
active account nears its 5h/7d limit. (Defaults live in `settings.json` in the backup root;
flags override them.)

- Syntax: `cswap auto [--once] [--json] [--interval SECONDS] [--threshold PCT]
  [--cooldown SECONDS] [--include-api-key-accounts | --no-include-api-key-accounts] [--dry-run]
  [--debug]`
- Flags / options:
  - `--once` — evaluate once, maybe switch, then exit. **Exit codes: `0` switched · `1` error
    · `2` no action needed · `3` blocked (no viable target).**
  - `--json` — one machine-readable JSON event per line.
  - `--interval SECONDS` — poll interval in loop mode (**min 15; default 60**).
  - `--threshold PCT` — switch when the binding 5h/7d window reaches this utilization
    (**50–99.9; default 90**).
  - `--cooldown SECONDS` — minimum time between proactive switches (**default 300**).
  - `--include-api-key-accounts` / `--no-include-api-key-accounts` — allow/deny switching onto
    API-key accounts as a last resort (they bill per token; **default: excluded**).
  - `--dry-run` — **read-only**: evaluate and report, but never switch or write state. Always
    run this first.
  - `--debug` — verbose diagnostics.
- **Confirm first before enabling the live `auto` loop** (without `--dry-run`) — it is
  state-changing (rule 4).

## `run` — launch Claude Code pinned to one account  ·  `state-changing` (launches a process)

Run a Claude Code invocation pinned to a specific account in an isolated session — useful for
running multiple accounts in parallel. **This terminal only.**

- Syntax: `cswap run <account-number|email> [-- <claude args>]`
- Flags / options:
  - `<account-number|email>` — the account to pin the session to.
  - `--` — everything after it is passed through to Claude Code verbatim.
  - `--share-history` — share history across `run` sessions. **Not supported on Windows yet**
    (macOS/Linux only) — see troubleshooting.

## `tui` — full-screen live dashboard  ·  `read-only`

Full-screen interactive live dashboard. Bare `cswap` (no subcommand) also opens this TUI. No
documented flags (`[unverified]` if any exist — confirm via `cswap tui --help`).

## `watch` — full-screen live dashboard  ·  `read-only`

Full-screen live dashboard. No documented flags (`[unverified]` vs `tui` differences — confirm
via `cswap watch --help`).

## `config` — read and change cswap settings  ·  mixed

- Syntax: `cswap config` (read all settings) or `cswap config set KEY VALUE` (change one).
- **Bare `cswap config`** — show all settings. **read-only.**
- `set KEY VALUE` — change one setting. **state-changing** (e.g.
  `cswap config set autoswitch.threshold 85`). Confirm first (rule 4).
- `[unverified]` `config get`, `config unset`, and `config path` are **not shown in the v0.18.1
  `--help`** — do not assume they exist. To read a single value, run bare `cswap config` (it
  prints all keys) or confirm via `cswap config --help`. Do not run `cswap config get ...` or
  `cswap config path` as if confirmed.
- Real keys and defaults (all under the `autoswitch.*` namespace):

  | Key | Default |
  |-----|---------|
  | `autoswitch.threshold` | `90` |
  | `autoswitch.intervalSeconds` | `60` |
  | `autoswitch.cooldownSeconds` | `300` |
  | `autoswitch.hysteresisPct` | `10` |
  | `autoswitch.strategy` | `best` |
  | `autoswitch.includeApiKeyAccounts` | `false` |
  | `autoswitch.unhealthyTicks` | `3` |

## `remove` — remove an account  ·  `state-changing`

Remove a registered account. Alias: `rm`.

- Syntax: `cswap remove <account-number|email>` (alias: `cswap rm ...`)
- Hard to reverse (the account would need re-adding). Confirm first (rule 4).

## `export` — back up accounts/config  ·  `read-only` (writes a file you name)

Export account data to a file (or stdout).

- Syntax: `cswap export <path>`
  - `<path>` — output file. Use `-` to write to **stdout**.
- Flags / options:
  - `--account NUM|EMAIL` — limit the export to a single account.
  - `--full` — include the full `~/.claude.json` in the export. **Default (without `--full`) is
    the `oauthAccount` entry only.**

## `import` — restore/migrate from an export  ·  `state-changing`

Import accounts/state from an export file.

- Syntax: `cswap import <path> [--force]`
  - `<path>` — input file. Use `-` to read from **stdin**.
  - `--force` — overwrite existing accounts during import. **With `--switch-to`, activate the
    stored credentials without backing up the current login first.**
- **Can overwrite existing accounts/backups** — confirm before running (rule 4).

## `upgrade`  ·  `state-changing`

Upgrade cswap itself. Alias: `update`. No documented flags (`[unverified]` — confirm via
`cswap upgrade --help`). Equivalent to the package-manager upgrade commands below.

## `purge`  ·  `state-changing` (destructive)

Remove **all claude-swap data from the system.** No documented flags. **Destructive and hard to
reverse** — confirm explicitly (rule 4) before running.

## `menubar`  ·  macOS only

macOS menubar helper. Not available on Windows/Linux.

---

## Install / upgrade

Installing or upgrading runs a shell command — **show it and let the user run or confirm it**
(rule 6). This skill does not silently install.

```powershell
# Preferred install (uv)
uv tool install claude-swap

# Alternative (pipx)
pipx install claude-swap

# Upgrade
uv tool upgrade claude-swap
pipx upgrade claude-swap
```

```powershell
# macOS-only: install the menubar helper extra
uv tool install 'claude-swap[menubar]'
```

---

## Data & config paths

`config-dir` below is the platform's cswap config directory.

| Platform | Credentials | Config dir |
|----------|-------------|------------|
| **Windows** | `~/.claude-swap-backup/credentials/` | `~/.claude-swap-backup/` |
| **macOS** | system **Keychain** | `~/.claude-swap-backup/` |
| **Linux / WSL** | `${XDG_DATA_HOME:-~/.local/share}/claude-swap/credentials/` | `${XDG_DATA_HOME:-~/.local/share}/claude-swap/` *(NOT `~/.claude-swap-backup/`)* |

Under the config dir:

- Sessions → `{config-dir}/sessions/`
- Tool preferences → `{config-dir}/settings.json`
- Auto-switch state → `{config-dir}/autoswitch_state.json`

`[unverified]` `cswap config path` is **not shown in the v0.18.1 `--help`** — do not assume it
exists. The paths above come from the platform table; read settings with bare `cswap config`,
and confirm any path command via `cswap config --help`.
