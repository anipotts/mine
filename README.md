<!-- tested with: claude code v2.1.81 -->
# mine

if this helped you, star it — it helps others find it.

mines every claude code session into a local sqlite database. total recall for your dev work.

## install

```bash
claude plugin add anipotts/mine
```

requires `python3` (stdlib only, no pip) and `sqlite3` (ships with macOS/linux).

## usage

just type `/mine` followed by what you want to know. plain language works.

### daily dashboard

```
> /mine
```

```
📊 last 7 days

  date        sessions  api value
  2026-03-28  1         $49.05
  2026-03-27  5         $82.55
  2026-03-26  3         $15.64
  2026-03-25  8         $67.87

  top projects: fullstack (50 sessions), quantercise.com (36), rudy (28)
  top tools: Bash (10.4K), Read (9.3K), Edit (5.1K)
  cache hit rate: 95%
```

### search past sessions

```
> /mine search "websocket"
```

```
  3 sessions matched "websocket"

  1. imessage-mcp (2026-03-12) — "implement websocket transport for real-time sync"
     38 tool calls, 22 min active, $4.12
  2. rudy (2026-03-08) — "add websocket reconnection with exponential backoff"
     67 tool calls, 34 min active, $8.91
  3. fullstack (2026-02-28) — "debug websocket auth handshake failure"
     12 tool calls, 8 min active, $1.44
```

### find where sessions went wrong

```
> /mine mistakes
```

```
  session health across all projects

  signal                                    sessions   what it means
  burned (high cost, no commits)            23         effort spent, nothing shipped
  looped (same file edited 5+ times)        130        claude was stuck iterating
  compacted (context overflowed)            103        task was too big for one session
  abandoned (20+ tool calls, no commit)     41         work started but never landed

  worst offenders:
  1. fullstack (2026-03-14) — 143 tool calls, 2 compactions, 0 commits, $28.41
  2. antileak (2026-03-09) — 98 tool calls, 3 compactions, 0 commits, $19.22

  mine remembers these. next time you start a session on these projects,
  the SessionStart hook surfaces what went wrong so claude adjusts its approach.
```

### more intents

| command | what it does |
|---------|-------------|
| `/mine cost this month` | cost breakdown by project and model |
| `/mine top projects` | all projects ranked by sessions and API value |
| `/mine top tools` | tool usage breakdown |
| `/mine cache hit rate` | cache efficiency analysis |
| `/mine hotspots` | files you keep editing across sessions |
| `/mine loops` | where you got stuck (same file edited 5+ times) |
| `/mine story myapp` | narrative history of a project |
| `/mine compare this week vs last` | side-by-side period comparison |
| `/mine backfill` | re-parse recent session logs into the database |
| `/mine help` | full list of intents and examples |

---

## how it works

mine runs 5 hooks across the claude code session lifecycle via a single python dispatcher (`hooks/hook.py`):

| event | what it does |
|---|---|
| SessionEnd | parses session + subagent transcripts into mine.db (async) |
| SubagentStop | parses a single subagent transcript on completion |
| PreCompact | increments compaction_count + cost anomaly warning if >2x average |
| SessionStart | project move detection, solution recall, auto-backfill |
| PostToolUseFailure | records errors, surfaces past similar failures to prevent repeats |

one file, zero bash scripts, zero jq dependency. all JSON parsing via stdlib, all SQL via parameterized queries.

## config

create `~/.claude/mine.json` to toggle individual features:

```json
{
  "ingest": true,
  "search": true,
  "mistakes": true,
  "burn": true,
  "move_detect": true,
  "compact": true,
  "auto_backfill": true
}
```

all features default to enabled. set any to `false` to disable.

## database

everything lives at `~/.claude/mine.db`. schema: `scripts/schema.sql`.

key views:

| view | what it shows |
|---|---|
| `user_session_costs` | per-session cost, duration, tools (main sessions only) |
| `user_tool_calls` | tool calls with project context |
| `project_costs` | per-project cost, session count, date range |
| `daily_costs` | per-day cost and token totals |

```bash
# quick stats
python3 scripts/mine.py --stats

# backfill all history
python3 scripts/mine.py --workers 8
```

## part of [claude-code-tips](https://github.com/anipotts/claude-code-tips)

- [claude-code-tips](https://github.com/anipotts/claude-code-tips) — the patterns behind these tools
- [claudemon](https://github.com/anipotts/claudemon) — real-time session monitor
- [cc](https://github.com/anipotts/cc) — cross-session messaging
- [imessage-mcp](https://github.com/anipotts/imessage-mcp) — iMessage MCP server

## more from me

- [anipotts.com/thoughts](https://anipotts.com/thoughts) — long-form
- [buttondown.com/anipotts](https://buttondown.com/anipotts) — newsletter
- [@anipottsbuilds](https://instagram.com/anipottsbuilds) — short-form

## license

MIT
