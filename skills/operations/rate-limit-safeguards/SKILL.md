# Rate Limit Safeguards & Task Queue System

## Overview

This skill documents the architecture and operational patterns for preventing API rate-limit
crashes in automated AI agent task loops. The core insight: **never inject tasks into a session that
cannot respond**. Instead, queue them and drain slowly on recovery.

Born from a real incident where a feedback loop of automated prompts into a rate-limited session
extended the lockout period catastrophically. The queue system eliminates that class of failure.

## When to Use

Apply these safeguards whenever:
- You are building or modifying a scheduled task injection system
- You are spawning 3+ agents in a single session
- You observe signs of rate-limit pressure (slow responses, early compaction, growing queue)
- You are designing any system that sends repeated automated prompts to an AI session

## Architecture

### Queue System

Tasks that cannot be injected immediately are stored in a JSON queue file and drained
slowly once the session recovers. No task is ever lost -- just spaced out.

```
/tmp/task_queue.json
```

Queue entry structure:
```json
{"type": "task-type-name", "message": "...", "queued_at": 1743500000}
```

**Dedup by type**: If the same task type is already in the queue, the duplicate is dropped
silently. This prevents backlog explosion for recurring tasks.

### Responsiveness Detection

`session_is_responsive()` uses two signals:
1. **Activity file modification time** -- checks the most recently modified activity file
   for your session. If modified within `RECENT_THRESHOLD=300` seconds (5 minutes), session
   is considered active.
2. **Human interaction file** -- checks a human interaction timestamp file against the same
   300-second threshold.

If either signal is fresh, the session is responsive. Both stale = unresponsive = queue mode.

### inject_or_queue Routing

Central routing function called by every scheduled task:

```bash
inject_or_queue "task-type" "$TASK_MESSAGE" "$SESSION"
```

Decision tree:
- `SESSION_RESPONSIVE == true` AND `CYCLE_INJECTIONS < MAX_INJECTIONS_PER_CYCLE`
  -> Acquire lock, inject via tmux send-keys, increment counter, release lock
- Any other condition
  -> `queue_task()` -- append to JSON queue, log "Queued task: <type>"

### Queue Drain on Recovery

At the start of each cycle, if the session is responsive and the queue is non-empty:

```bash
drain_queue 2 30 "$SESSION"
```

Parameters:
- `max_drain=2` -- drain at most 2 tasks per cycle
- `spacing=30` -- wait 30 seconds between each drained task
- Oldest entries drained first (sorted by `queued_at`)
- Drained count is added to `CYCLE_INJECTIONS` before any new injections

### Cycle Injection Cap

```bash
MAX_INJECTIONS_PER_CYCLE=3
```

Total injections (queue drain + new) per 30-minute cycle. This prevents API overload when
multiple tasks become due simultaneously (e.g., after a 2-hour outage clears).

Budget allocation example:
- Queue had 5 items -> drain 2 (uses 2 of 3 slots)
- 1 new task slot remaining for new tasks this cycle
- Remaining 3 queue items defer to next cycle automatically

### Alert System

When queue size reaches 10+ items, an alert is sent to the operator via your preferred
notification channel (Telegram, Slack, email, etc.) once per alert period. Use a guard file
to prevent repeated alerts. Alert resets when queue drops below 10.

## Implementation

### Key Configuration Values

| Variable | Value | Purpose |
|----------|-------|---------|
| `TASK_QUEUE_FILE` | `/tmp/task_queue.json` | Queue storage |
| `RECENT_THRESHOLD` | `300` (5 min) | Responsiveness window |
| `MAX_INJECTIONS_PER_CYCLE` | `3` | Per-cycle injection cap |
| `LOCK_TTL` | `900` (15 min) | Lock staleness threshold |
| Queue alert threshold | `10 items` | Alert trigger |
| Drain per cycle | `2` | Max queue drain per cycle |
| Drain spacing | `30s` | Delay between drained tasks |

### Key Functions

| Function | Purpose |
|----------|---------|
| `session_is_responsive()` | Returns 0 if session active, 1 if stale |
| `inject_or_queue(type, msg, session)` | Routes task to injection or queue |
| `queue_task(type, message)` | Appends to JSON queue with dedup |
| `drain_queue(max, spacing, session)` | Drains N oldest tasks with spacing |
| `queue_size()` | Returns current queue length |
| `acquire_lock(name)` | Creates lock file. Returns 1 if held. |
| `release_lock(name)` | Removes lock file (caller must be holder) |

### Lock System

Prevents multiple simultaneous tmux injections:

- **Lock file**: `/tmp/task_inject_lock`
- **TTL**: 15 minutes (`LOCK_TTL=900`)
- **Stale detection**: If lock older than TTL, automatically reclaimed (prevents a crashed
  task from permanently blocking the queue)
- `inject_or_queue` handles lock internally. Callers do NOT call acquire/release directly.

## Agent Spawning Guidelines

These are separate from the task queue but address the same root cause: API overload from
too many concurrent requests.

| Scenario | Approach |
|----------|---------|
| 1-2 independent agents | Parallel is fine |
| 3+ agents | Sequential -- wait for each to complete before spawning next |
| Background agents | Max 2 concurrent. Use `run_in_background: true`. |
| After spawning background agent | Wait at least 30 seconds before spawning another |

**Principle**: Prefer sequential spawning with spacing over concurrent hard caps. Spacing
prevents API burst without sacrificing throughput on single-agent tasks.

## What Caused the Original Crash

**Root cause**: Automated prompts kept firing into a rate-limited session.

**Mechanism**:
1. The AI session hit API rate limits -- session became unresponsive
2. Scheduled tasks continued injecting every 30 minutes
3. Each injection consumed API tokens (the tmux send-keys itself triggered API calls)
4. The token consumption extended the rate-limit lockout period
5. Extended lockout meant more tasks piled up, compounding the problem
6. Feedback loop sustained itself for hours

**Fix applied**: The queue system prevents injection entirely when the session is unresponsive.
Tasks accumulate in JSON without touching the API. When the session recovers, they drain slowly
(2 per cycle, 30s spacing, under the 3-injection cap), staying well under per-minute limits.

## Recovery Protocol

When rate-limited:

1. **Automatic handling** -- the queue system accumulates tasks in the queue file
   without injecting them. No manual intervention needed.

2. **When session recovers** -- next cycle detects responsiveness, drains up to 2 queued
   tasks with 30-second spacing, respecting the 3-injection cap.

3. **If outage exceeds ~5 cycles (~150 min)** -- queue grows to 10+ items, operator is alerted.
   They can assess whether manual intervention is needed.

4. **Resume normal operations** -- once session activity resumes, the queue drains automatically
   over subsequent cycles.

5. **Manual drain** -- the queue self-resolves. No manual intervention needed unless the alert
   fires and the operator determines the backlog is stale/irrelevant (in which case:
   `echo "[]" > /tmp/task_queue.json`).

## Signs of Approaching Rate Limits

Watch for these early warning signals:
- Response times increasing noticeably
- Tool calls taking longer than usual
- Context compaction firing earlier than expected
- Task queue growing (check: `python3 -c "import json; print(len(json.load(open('/tmp/task_queue.json'))))"`)
- Alert notification about queue size

## Adoption Guide

To adopt this pattern in your own AI agent setup:

1. Implement the queue functions:
   - `session_is_responsive()`
   - `queue_task()`
   - `drain_queue()`
   - `queue_size()`
   - `inject_or_queue()`
   - `acquire_lock()` / `release_lock()`

2. Replace all direct `tmux send-keys` injection calls with `inject_or_queue "type" "$MSG" "$SESSION"`

3. Add the drain block at the start of your main loop (before any new injections):
   ```bash
   SESSION_RESPONSIVE="false"
   CYCLE_INJECTIONS=0
   if session_is_responsive; then
       SESSION_RESPONSIVE="true"
       Q_SIZE=$(queue_size)
       if [[ $Q_SIZE -gt 0 ]]; then
           DRAINED=$(drain_queue 2 30 "$SESSION")
           CYCLE_INJECTIONS=$((CYCLE_INJECTIONS + DRAINED))
       fi
   fi
   ```

4. Set alert thresholds appropriate for your notification channel and operator preferences.
