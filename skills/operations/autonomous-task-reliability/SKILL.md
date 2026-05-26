---
name: autonomous-task-reliability
description: Ensure overnight and autonomous tasks actually execute. Scheduled loop, idle detector, pre-shutdown dispatch, watchdog with auto-fix. Solves the planned-but-not-started gap.
version: 1.0.0
category: operations
tags: [automation, reliability, overnight, watchdog, idle-detection, scheduled-tasks]
author: PureBrain
---

# Autonomous Task Reliability

## The Problem

AI sessions end. Overnight work doesn't happen. The agent says "I'll dispatch the team tonight" but the session closes before it actually does. The scratchpad says "planned" but nothing was started. The human wakes up disappointed.

This skill solves that with 4 structural mechanisms that make task execution inevitable, not aspirational.

## Component 1: Persistent Scheduled Loop

**Purpose:** A persistent process that runs independently of chat sessions. Even if the AI session ends, scheduled tasks keep firing.

**Pattern:** Chat session = interactive work. Scheduled loop = automated work. They are decoupled.

**Implementation:**

```bash
#!/bin/bash
# scheduled_loop.sh - runs in a persistent tmux session
# Launch: tmux new-session -d -s task-loop './tools/scheduled_loop.sh'

CYCLE_SECONDS=1800  # 30 min between cycles
STATE_FILE="./data/scheduled-tasks-state.json"
LOG_FILE="./logs/scheduled_loop.log"

while true; do
    CURRENT_HOUR=$(date -u +%H)
    CURRENT_DOW=$(date -u +%u)  # 1=Mon, 7=Sun

    # Check each registered task
    # Example: daily morning briefing at 13 UTC (8 AM EST)
    if [[ "$CURRENT_HOUR" == "13" ]]; then
        if ! task_ran_today "morning-briefing"; then
            log "Firing morning briefing..."
            # Inject wake prompt into the AI's tmux session
            tmux send-keys -t ai-primary "[TASK] Morning briefing due. Execute now." Enter
            record_task_result "morning-briefing" "injected"
        fi
    fi

    # Add more task triggers here...

    touch /tmp/loop_heartbeat  # Watchdog liveness signal
    sleep "$CYCLE_SECONDS"
done
```

**Task Registry:** A JSON file tracks what tasks exist, when they should run, and when they last ran:

```json
{
    "morning-briefing": {
        "schedule": "Daily",
        "description": "Send morning status update",
        "last_run": "2026-05-08",
        "status": "completed"
    }
}
```

**Key principle:** The scheduled loop never stops. It runs 24/7 in tmux. The AI session can crash, restart, or be idle. The loop keeps firing.

## Component 2: Pre-Shutdown Dispatch Hook

**Purpose:** Before any session closes, force-check for queued tasks and dispatch them.

**Pattern:** Never let "planned but not started" survive a session boundary.

**Implementation:**

```python
def pre_shutdown_check():
    """Run this before every session end / context compaction."""

    # Read scratchpad for pending tasks
    scratchpad = Path("scratchpad.md").read_text()

    # Look for undispatched work
    pending_markers = [
        "TODO overnight",
        "will dispatch",
        "queue for tonight",
        "run after session",
        "planned but not started"
    ]

    pending = []
    for marker in pending_markers:
        if marker.lower() in scratchpad.lower():
            pending.append(marker)

    if pending:
        print(f"BLOCKING SHUTDOWN: {len(pending)} undispatched tasks found")
        print("Dispatching now before session ends...")

        # Force dispatch each pending task
        for task in pending:
            dispatch_task(task)  # Spawn agent, run script, etc.

        print("All tasks dispatched. Safe to close.")
    else:
        print("No pending tasks. Clean shutdown.")
```

**Integration:** Wire this into:
- Context compaction events (80%/90% warnings)
- Manual session end commands
- Portal restart/resume flows

## Component 3: Idle Detector

**Purpose:** When the human stops messaging, automatically switch to autonomous work mode.

**Pattern:** Silence = permission to work autonomously.

**Implementation:**

```python
import time

IDLE_THRESHOLD_SECONDS = 1800  # 30 minutes
last_human_message = time.time()

def on_human_message():
    """Call this every time the human sends a message."""
    global last_human_message
    last_human_message = time.time()

def check_idle():
    """Call this periodically (e.g., every 5 min)."""
    idle_seconds = time.time() - last_human_message

    if idle_seconds > IDLE_THRESHOLD_SECONDS:
        enter_autonomous_mode()

def enter_autonomous_mode():
    """Human is idle. Time to work independently."""

    # 1. Check scratchpad for pending tasks
    pending = get_pending_tasks()

    # 2. Check task queue for due tasks
    due_tasks = get_due_tasks()

    # 3. Dispatch everything
    dispatched = []
    for task in pending + due_tasks:
        dispatch_task(task)
        dispatched.append(task)

    # 4. Notify human what was dispatched
    if dispatched:
        send_notification(
            f"You've been idle for 30 min. "
            f"Dispatched {len(dispatched)} tasks: "
            f"{', '.join(dispatched)}"
        )
```

**Integration:** The idle detector runs as a background check within the scheduled loop, not inside the chat session.

## Component 4: Watchdog

**Purpose:** Verify that scheduled tasks actually produced results. Trust but verify.

**Pattern:** Every task must prove it ran. The watchdog has permissions to fix and relaunch.

**Implementation:**

```bash
#!/bin/bash
# watchdog.sh - monitors the scheduled loop health

HEARTBEAT_FILE="/tmp/loop_heartbeat"
STALE_THRESHOLD=7200  # 2 hours
LOOP_SESSION="task-loop"

while true; do
    # Check 1: Is the scheduled loop process alive?
    if [ -f "$HEARTBEAT_FILE" ]; then
        heartbeat_age=$(($(date +%s) - $(stat -c %Y "$HEARTBEAT_FILE")))
        if [ "$heartbeat_age" -gt "$STALE_THRESHOLD" ]; then
            echo "[WATCHDOG] Scheduled loop stale ($heartbeat_age sec). Restarting..."
            tmux kill-session -t "$LOOP_SESSION" 2>/dev/null
            sleep 2
            tmux new-session -d -s "$LOOP_SESSION" './tools/scheduled_loop.sh'
            notify "Scheduled loop was stale. Restarted automatically."
        fi
    else
        echo "[WATCHDOG] No heartbeat file. Scheduled loop may not be running."
        tmux new-session -d -s "$LOOP_SESSION" './tools/scheduled_loop.sh'
        notify "Scheduled loop was not running. Started automatically."
    fi

    # Check 2: Did scheduled tasks actually produce results?
    check_task_results

    sleep 1800  # Check every 30 min
done

check_task_results() {
    # Read task state
    # For each task that should have run today:
    #   - Did it actually run? (check last_run date)
    #   - Did it succeed? (check status)
    #   - If missed: attempt relaunch
    #   - If failed 3x: notify human

    local today=$(date +%Y-%m-%d)
    # Parse scheduled-tasks-state.json and verify each entry
}
```

**Watchdog permissions:**
- Can restart the scheduled loop process
- Can re-inject missed task prompts
- Can send alerts to the human
- Cannot modify task definitions (only execute them)

## Integration Guide

1. **Set up tmux sessions:**
   ```bash
   tmux new-session -d -s task-loop './tools/scheduled_loop.sh'
   tmux new-session -d -s watchdog './tools/watchdog.sh'
   ```

2. **Register tasks** in your state file with schedule, description, and expected frequency.

3. **Wire the pre-shutdown hook** into your session management (context compaction, portal restart, manual close).

4. **Configure the idle detector** with your preferred threshold (15-30 min recommended).

5. **Test with a simple task** (e.g., "send a status message every hour") before adding complex overnight tasks.

## Key Principles

- The scheduled loop is the backbone. Everything else is defense-in-depth.
- Sessions are ephemeral. Scheduled work must not depend on session state.
- "Planned but not started" is a structural failure, not a memory failure. Fix it with hooks, not reminders.
- The watchdog has permissions to fix, not just report. A watchdog that only barks is useless.
- Random jitter on timing makes automation look natural (1-10 min variance on start times).
