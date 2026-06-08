# agenttop

A terminal dashboard — `htop`-style — for the two things running quietly in the background on your Mac:

1. **Your background agents** — every scheduled/persistent job macOS runs for you via **launchd** (`~/Library/LaunchAgents`, `/Library/LaunchAgents`, `/Library/LaunchDaemons`).
2. **Your Claude spend** — how many tokens Claude Code is burning, per model, per project, per session, plus your live Anthropic rate-limit budget.

One screen, two panes, auto-refreshing every 2 seconds. Zero dependencies — pure Python 3 standard library (`curses` + `json`).

```
┌ LAUNCHD  30 jobs (12 ok, 1 fail) ─────────┬ CLAUDE  window: 5h (v cycle) ──────────────┐
│ ● com.user.backup        every 1h   ok    │ Anthropic limits (live from API headers)   │
│ ⚠ com.user.sync          at load    exit 1│   5h    0% ███········· reset 4h 12m allowed │
│ ○ com.user.reindex       Mon 03:00  idle  │   7d    5% █···········  reset 6d 1h allowed  │
│ ...                                        │ Local activity (window: 5h)                │
│                                            │   in 13.3M  out 107.6M  cache 53.3G         │
│                                            │ By model / By project / Sessions / 24h chart│
└────────────────────────────────────────────┴─────────────────────────────────────────────┘
```

---

## Requirements

- **macOS** — uses `launchctl` and `plutil` for the launchd pane.
- **Python 3** — standard library only, nothing to `pip install`.
- **Claude Code CLI** (`claude`) on `PATH` — only needed to fetch live Anthropic rate-limit headers (the right pane's "Anthropic limits" box). Everything else works without it.

## Install

It's a single self-contained script. Drop it on your `PATH` and make it executable:

```sh
chmod +x ~/.local/bin/agenttop
agenttop
```

---

## Left pane — launchd (your background agents)

Lists every launch agent and daemon, with live status pulled from `launchctl list`:

- **Status** — `●` running · `✗` failed (non-zero last exit) · `⚠` running but last exit errored · `○` idle / never run.
- **Schedule** — decoded from the plist: `every 1h`, `Mon 03:00`, `keep-alive`, `at load`, `on path change`, …
- **PID**, **last exit code**, and **log size**.
- Noise like Adobe / Grammarly / OneDrive helpers is filtered out.

**Actions on the selected job:**

| key | action |
|-----|--------|
| `Enter` | live-follow its log file (stdout + stderr), any key returns |
| `R` | restart (`launchctl kickstart -k`) |
| `s` | stop (`launchctl kill SIGTERM`) |
| `/` | filter by label (Enter applies, Esc cancels) · `c` clears |
| `j`/`k` or `↓`/`↑` | move · `g`/`G` top/bottom |

> System daemons in `/Library/LaunchDaemons` need `sudo` to restart — agenttop tells you the exact command when permission is denied.

## Right pane — Claude usage & spend

Reads your Claude Code session logs in `~/.claude/projects/**/*.jsonl` and aggregates them. Token counts are the exact `usage` numbers Anthropic returned for each request — no estimation.

- **Anthropic limits (live)** — real `anthropic-ratelimit-*` headers, fetched by making one tiny `claude -p .` call (costs **1 billable message**), cached for an hour. Shows your 5h and 7d utilization, reset countdown, and status.
- **Local activity** — input / output / cache tokens, session and message counts for the current window.
- **By model** and **By project** — ranked bar breakdowns.
- **Sessions** — active sessions with git branch, model, last-seen, and total tokens. `Enter` live-tails one (every new assistant message streams in as a row).
- **Last 24h** — peak tokens/min per hour, as twin sparklines (the same metric Anthropic's Console rate-limit graph shows).

| key | action |
|-----|--------|
| `v` | cycle window: `5h` → `today` → `7d` |
| `Enter` | live-tail the selected session |
| `j`/`k` or `↓`/`↑` | move selection |

## Global keys

| key | action |
|-----|--------|
| `Tab` | switch pane focus |
| `r` | force refresh |
| `f` | fetch Anthropic limits now (costs 1 message) |
| `q` | quit |

---

## Performance — the persistent index

The Claude pane parses your entire `~/.claude/projects` history (often hundreds of MB, millions of JSONL lines). Parsing that from scratch every launch takes minutes, so agenttop keeps a **persistent index cache**:

- **First launch** parses everything once (~2–3 min on a large history) and writes `~/.cache/agenttop/index-cache.pkl`.
- **Every launch after** reloads that cache and reads only the bytes appended since — **~2 seconds**.
- Events older than **31 days** are dropped on save (no view looks back further), so memory and the cache stay bounded as history grows.
- The cache is rebuildable — delete it any time; it's regenerated on the next run. It also self-invalidates on a schema-version bump or corruption.

> Heads up: on a large history the cache file can be a few hundred MB. It lives in `~/.cache/agenttop/` alongside the cached rate-limit headers.

## Command-line (non-interactive) modes

For scripts, cron, or a quick glance without the TUI:

```sh
agenttop --once                  # print the launchd job table once and exit
agenttop --once-claude           # print a Claude usage summary (makes 1 billable call)
agenttop --once-claude --no-fetch # same, but use cached rate-limit headers (free)
```

These share the same on-disk index cache, so running `--once-claude` also warms it for the TUI.

---

## Where it keeps state

| path | what |
|------|------|
| `~/.cache/agenttop/index-cache.pkl` | parsed Claude usage index (rebuildable) |
| `~/.cache/agenttop/anthropic-limits.json` | last fetched rate-limit headers (1h TTL) |

Both are caches — safe to delete; agenttop regenerates them.
