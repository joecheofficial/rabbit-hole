# Changelog

## 2.0.2 (2026-10-06)

Late-night nudge, and a clock fix that made quiet hours start late.

Added
- Late-night signal: inside quiet hours, the UserPromptSubmit hook adds a required `Rabbit Hole signal: late night (HH:MM, inside quiet hours)` line telling the assistant to open its reply by naming the time and suggesting bed, with an offer to hand the rest to background agents. It repeats at most every 45 minutes per session and is logged as kind `late-night`. `RABBIT_HOLE_LATE_NIGHT=off` turns it off.
- `RABBIT_HOLE_NOW` (ISO timestamp) pins the hook's clock, for tests.

Fixed
- `localClock` applied `% 24` to minutes as well as hours, so 00:48 read as 00:00 and 23:31 as 23:07, which skewed quiet-hours boundaries and the times shown in reminders. Minutes now wrap at 60. Regression test covers :48 and :31.

## 2.0.1 (2026-10-02)

Loop guard no longer fires on ordinary Bash-heavy work; frequency rule now measures churn on the same target.

Fixed
- The "same tool 5+ times in the last 10 calls" rule tripped on nearly every call in a normal session, because agents use Bash, Read and Edit for most of their work. Workhorse tools (Bash, Read, Edit, Write, MultiEdit, NotebookEdit, Grep, Glob, LS, TodoWrite, Task, Agent, WebFetch, WebSearch) are now exempt from the raw tool-name count.

Added
- `loop-churn` signal: a workhorse tool aimed at the same target 5 or more times in the last 10 calls. For Bash the target is the normalised command (whitespace collapsed, numbers and quoted strings stripped, leading `cd <dir> &&` dropped, first 60 characters); for file tools it is the file path. The advisory names the target and fires only on a call that hits it.
- Tests for the quiet cases (ten different Bash commands, Bash and Read taking turns, a shared `cd` prefix) and for each trip.

Changed
- `loop-frequency` now applies to non-workhorse tools only (MCP tools, for example). Threshold unchanged.
- `loop-alternation` is documented and tested as comparing (tool + input) pairs: two tools taking turns with different input never trips it.
- The session trace stores a hash of each call's target next to the input hash. Traces written by 2.0.0 load unchanged.

## 2.0.0 (2026-10-01)

Always-on rewrite. Rabbit Hole no longer waits to be invoked; once installed it watches every Claude Code session through two hooks and stays silent until a signal trips.

Added
- One-question intake on the first turn: outcome in one sentence plus strictness (strict / mid / loose), stored per session by the hook.
- `hooks/claude-code/user-prompt-submit.sh`: session state, cadence-limited reminder with elapsed time, local time, quiet hours, mission and strictness.
- `hooks/claude-code/pre-tool-use.sh`: mechanical loop guard (3 identical calls in a row, 5+ uses of one tool in 10 calls, A-B-A-B alternation), advisory only, every trip logged.
- "Still broken" rule: three turns that read as still failing end the iterate loop and ask one diagnostic question.
- Topic-switch drift signal with an intervention budget: first 3 switches free, at most 2 full checkpoints per session, one-liners after that.
- Park, don't chase: `rabbit-hole park` and an append-only parking lot per project.
- Fatigue beyond the clock: quiet hours AND loop or churn evidence.
- Decision budget: checkpoints offer at most 3 choices, recommended first.
- `bin/rabbit-hole` CLI: `park`, `mission`, `check`, `done`, `log`, `stats`, `config`.
- `done` writes one resumption line to `resume.md`; the next session's intake shows it.
- `config.example.json` and `config.md`, `references/signals.md`, `install.sh`, `uninstall.sh`, tests under `tests/`.

Changed
- Checkpoint format keeps `Rabbit Hole check:` with Mission / Risk / Why, now with three choices instead of five.
- Prompt templates genericised.

Removed
- Startup-context and wrapper scripts tied to a specific private setup. Replaced by the two hooks plus an AGENTS.md line for Codex.

## 1.x

Advisory skill text only, armed by startup-context scripts.
