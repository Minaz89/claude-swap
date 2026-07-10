# cswap-operator — a Claude Code skill for claude-swap

A [Claude Code](https://claude.com/claude-code) **skill** that teaches Claude to *operate*
the `cswap` CLI for you, in plain English — check per-account usage, switch accounts, and set
up auto-rotation — without you memorizing any commands.

It **drives** the existing `cswap` CLI; it does not reimplement account switching, OAuth, or
credential storage — the tool in this repo owns all of that.

> Say *"show my Claude usage across accounts"*, *"I'm about to hit my rate limit — switch me
> to a fresh account"*, or *"set up cswap auto-rotation at 90%"* and Claude runs the right
> `cswap` commands, confirming before anything that changes state.

## What's in the box

```
cswap-operator/
├── SKILL.md                     # trigger description + operating rules + workflow + decision guide
└── references/
    ├── commands.md              # every subcommand & flag (verified against cswap v0.18.1)
    ├── workflows.md             # setup · daily check→switch · auto · parallel run · backup
    └── troubleshooting.md       # when a switch takes effect, tokens, per-platform paths, Windows UTF-8 fix
```

## Prerequisites

1. **Claude Code** (CLI, desktop, or IDE extension) — the skill needs terminal access to run
   `cswap`. It does not work in the plain web chat.
2. **`cswap` installed** — see the [main README](../../README.md): `uv tool install claude-swap`
   (or `pipx install claude-swap`).

## Install the skill (plug and play)

Skills live in `~/.claude/skills/`. Drop this folder in and reload Claude Code.

**macOS / Linux**
```bash
git clone --depth 1 https://github.com/Minaz89/claude-swap.git
mkdir -p ~/.claude/skills
cp -r claude-swap/skills/cswap-operator ~/.claude/skills/
```

**Windows (PowerShell)**
```powershell
git clone --depth 1 https://github.com/Minaz89/claude-swap.git
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills" | Out-Null
Copy-Item -Recurse -Force claude-swap\skills\cswap-operator "$env:USERPROFILE\.claude\skills\"
```

Then **restart / reload Claude Code** so it discovers the skill. That's it.

## Use it

The skill is **model-invoked** — just talk to Claude naturally and it activates when your
message matches (usage, switching, rate limits, auto-rotation). Or invoke it explicitly by
typing `/cswap-operator`.

Claude runs read-only commands (`cswap list`, `cswap status`) directly and **confirms with you
before any state-changing command** (`switch`, `auto`, `import`, `config set`, `remove`,
`purge`). It never fabricates a usage number and never prints your tokens.

## Windows note

`cswap list` / `tui` / `watch` render box-drawing characters that crash on the legacy cp1252
console. Set UTF-8 first:

```powershell
$env:PYTHONUTF8 = '1'   # add to your PowerShell $PROFILE to make it permanent
```

The skill applies this automatically when it runs `cswap` on Windows.

## Scope

Manages **multiple accounts you own**. It does not swap *models* or *config profiles* — `cswap`
swaps accounts. For single-account use with no rotation, you don't need it.

---

Part of [Minaz89/claude-swap](https://github.com/Minaz89/claude-swap). Licensed under the same
terms as this repository ([MIT](../../LICENSE)).
