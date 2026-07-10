---
name: cswap-operator
description: "Activate when the user wants to operate the claude-swap (cswap) CLI to manage multiple self-owned Claude accounts — view per-account usage, switch the active account, or set up hands-off auto-rotation to dodge rate limits. Fires on intents like 'switch Claude account', 'I'm about to hit my rate limit', 'rotate accounts', 'show my Claude usage across accounts', 'run Claude Code as my other account', or 'set up cswap/claude-swap'. This skill drives the existing cswap CLI only (read usage via list/status/tui, switch with switch/auto, run pinned sessions with run); it never reimplements account swapping, OAuth, credential locking, or usage tracking — cswap owns all of that. Do NOT use for single-account setups with no rotation need, or for anything about swapping models or config profiles — cswap swaps Claude accounts, not models or profiles."
---

# cswap Operator — drive the `claude-swap` CLI across your own Claude accounts

> **Verified against `cswap` v0.18.1 (installed 2026-07-10).** Re-confirm with `cswap --help`
> after any upgrade — the live `--help` output is authoritative over this skill.
>
> **Windows operator note:** set `$env:PYTHONUTF8='1'` before running any `cswap` command —
> otherwise `list`/`tui`/`watch` crash with `UnicodeEncodeError` on box-drawing characters
> (a console-encoding quirk, not an account problem; see troubleshooting).

`claude-swap` (invoked as `cswap`) is a CLI that manages several **self-owned** Claude
accounts so you can rotate between them and avoid rate limits. It owns the hard parts —
account registration, OAuth/credential storage and locking, usage tracking, and
auto-rotation — and exposes a clean command surface to drive them. **This skill operates that
CLI; it never reimplements it.** Every usage number, threshold, and reset time the user sees
must be read from `cswap`, not produced by this skill.

## Operating rules

These six rules are binding for every action in this skill:

1. **Operate only the user's OWN accounts, and only ones already added by the user.** Never
   add, import, or switch to an account the user did not set up. This skill does not enroll
   accounts on the user's behalf beyond guiding the documented `add`/`add-token` flow.
2. **NEVER fabricate usage numbers, reset times, or thresholds.** Always read them from
   `cswap list` / `cswap status` / `cswap tui` and quote what those commands actually return.
   If a value is not available (or a flag is `[unverified]` on this machine), say so plainly
   rather than guessing.
3. **NEVER print OAuth tokens or `sk-ant-...` API keys** in output, logs, or examples. Refer
   to accounts **by index or email only**.
4. **STOP and confirm before any state-changing or hard-to-reverse action** — `switch`,
   enabling `auto`, `import` (can overwrite backups), `config set` / `config unset`,
   `remove`, and `purge`. State the exact command and its effect, then wait for the user.
   Read-only ops need no confirmation: `list`, `status`, `tui`, `watch`, any `--help`,
   `cswap config` (bare — read-only settings view), and `auto --dry-run`.
5. **Be platform-accurate about when a switch takes effect.** On **Linux and Windows** the
   new account is picked up automatically with no restart; on **macOS** the Keychain cache is
   ~30 seconds, or restart Claude Code for instant application. Do NOT claim a blanket
   "requires restart," and do NOT imply a seamless zero-latency in-place swap.
6. **NEVER silently install.** Installing or upgrading `cswap` runs a shell command — always
   show it and let the user run or explicitly confirm it. Do not run install/upgrade as part
   of another task without consent.

## Quick Reference Navigation

Open only the file your task needs (progressive disclosure).

| If the task is… | Read |
|-----------------|------|
| Look up a subcommand, flag, install/upgrade, or data/config path | [`references/commands.md`](references/commands.md) |
| Follow a recipe: setup, daily usage check, enable auto, parallel sessions, backup | [`references/workflows.md`](references/workflows.md) |
| "Why didn't my switch take effect?", token/refresh failures, platform path differences | [`references/troubleshooting.md`](references/troubleshooting.md) |

## Workflow

1. **Identify the need.** Is the user checking usage, dodging a rate limit, switching to a
   specific account, going hands-off with auto-rotation, or running a pinned session? That
   decides the command.
2. **Read current usage.** Run a read-only command — `cswap list` (per-account 5-hour + 7-day
   usage and reset times) or `cswap status` (current account/switch state). Quote what comes back;
   never invent numbers.
3. **Choose the action.** Map the need to `switch`, `auto`, or `run` (see the decision guide
   below).
4. **Confirm if state-changing.** For `switch`, `auto`, `import`, `config set/unset`,
   `remove`, or `purge`, show the exact command and wait for the user (rule 4).
5. **Run it.** Execute exactly the confirmed command.
6. **State when it takes effect.** Give the platform-correct answer (rule 5): Windows/Linux
   auto-pickup, or macOS ~30 s / restart.

## Decision guide — `switch` vs `auto` vs `run`

- **`switch`** — change the **active account** for subsequent Claude Code credential reads.
  One-shot. Use `cswap switch <N>` (or an email), or `cswap switch --strategy best` /
   `--strategy next-available` to let cswap pick. Add `--force` only when you intend to
  override cswap's safety checks.
- **`auto`** — **hands-off rotation** when an account nears its limit. cswap watches usage
  and rotates on its own. **Always preview with `--dry-run` first**, then run
  `cswap auto` (optionally `--threshold 85`). Tune the trigger via
  `cswap config set autoswitch.threshold <n>` (default `90`).
- **`run`** — launch a Claude Code invocation **pinned to a specific account** in an isolated
  session (parallel accounts at once): `cswap run <N> -- <claude args>`. Note:
  `--share-history` is **not supported on Windows yet** (macOS/Linux only).

## Quick commands

| Command | One-line purpose |
|---------|------------------|
| `cswap list` | Usage dashboard across all accounts (5-hour + 7-day windows, reset times). Read-only. |
| `cswap status` | Show current account/switch state (distinct from `list`). Read-only. |
| `cswap switch <N>` | Switch the active account by number or email. State-changing. |
| `cswap switch --strategy best` | Switch to the account with the most remaining 5h/7d quota. State-changing. |
| `cswap auto --dry-run` | Preview auto-rotation without changing anything. Read-only. |
| `cswap auto --threshold 85` | Hands-off rotation below a usage threshold. State-changing. |
| `cswap run <N> -- <args>` | Run Claude Code pinned to account N (isolated session). |
| `cswap tui` | Full-screen live dashboard. Read-only. |
