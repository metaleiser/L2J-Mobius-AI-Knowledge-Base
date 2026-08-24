# THREADING ARCHITECTURE

**Source of Truth**: `commons.threads.ThreadPool`, thread pool initialization  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## THREE THREAD POOLS

### 1. High-Priority Scheduled Pool

**Configuration**:
- Size: `ThreadConfig.HIGH_PRIORITY_SCHEDULED_THREAD_POOL_SIZE`
- Type: ScheduledThreadPoolExecutor
- Priority: PRIORITY_8 (highest)
- Policy: CallerRunsPolicy (caller thread executes if rejected)

**Purpose**: Critical server tasks that must complete

**Examples**:
```
Server status checks
Critical database operations
Emergency shutdown procedures
```

---

### 2. Standard Scheduled Pool

**Configuration**:
- Size: `ThreadConfig.SCHEDULED_THREAD_POOL_SIZE` (typical: 4-8)
- Type: ScheduledThreadPoolExecutor
- Priority: Normal
- Policy: CallerRunsPolicy
- Pre-started: Yes (all core threads started)

**Purpose**: Regular recurring tasks and timers

**Examples**:
```
NPC respawns (every 60 seconds)
Skill cooldown expiration
Event timers
Buff duration tracking
PvP event scheduling
```

**Task Duration**: Typical 100ms-1000ms

---

### 3. Instant Execution Pool

**Configuration**:
- Core Size: `ThreadConfig.INSTANT_THREAD_POOL_SIZE` (typical: 50-100)
- Max Size: Integer.MAX_VALUE (unlimited growth)
- Queue: LinkedBlockingQueue (unbounded)
- Keep-Alive: 1 minute
- Policy: Reject when queue full (standard behavior)

**Purpose**: Immediate execution of async tasks

**Examples**:
```
Process player skill cast
Handle combat damage
Send packets
Load/save player data
NPC AI ticks
Item enchantment
```

**Task Duration**: Typical 1-100ms

---

## SCHEDULING METHODS

All methods on `ThreadPool` class:

### Immediate Execution
```java
ThreadPool.execute(Runnable task)
  ├─ Uses: Instant Pool
  ├─ Returns: void
  └─ Executes ASAP
```

### Delayed Execution
```java
ThreadPool.schedule(Runnable task, long delayMs)
  ├─ Uses: Scheduled Pool
  ├─ Returns: ScheduledFuture
  └─ Executes after delay
```

### Fixed-Rate Repetition
```java
ThreadPool.scheduleAtFixedRate(Runnable, long initialDelayMs, long rateMs)
  ├─ Uses: Scheduled Pool
  ├─ First execution: after initialDelay
  ├─ Subsequent: every rateMs (regardless of task duration)
  └─ Returns: ScheduledFuture
```

**Issue**: If task takes longer than rate, next one starts immediately

### Fixed-Delay Repetition
```java
ThreadPool.scheduleWithFixedDelay(Runnable, long initialDelayMs, long delayMs)
  ├─ Uses: Scheduled Pool
  ├─ First execution: after initialDelay
  ├─ Delay between executions: delayMs AFTER task completes
  └─ Returns: ScheduledFuture
```

**Better for**: Variable-duration tasks

---

## USAGE EXAMPLES

### Immediate Task
```java
ThreadPool.execute(() -> {
    player.setXp(player.getXp() + 1000);
});
```

### Delayed Task
```java
ThreadPool.schedule(() -> {
    castle.stopSiege();
}, 3600000);  // 1 hour later
```

### Repeating Task
```java
ThreadPool.scheduleAtFixedRate(() -> {
    world.updateWeather();
}, 1000, 300000);  // Every 5 minutes, first after 1 sec
```

---

## TASK MANAGERS

**Special managers** that use ThreadPool for specific purposes:

| Manager | Purpose | Schedule |
|---------|---------|----------|
| GameTimeTaskManager | Day/night cycle | Every 100ms |
| ItemLifeTimeTaskManager | Item expiration | Every 1 minute |
| ItemsAutoDestroyTaskManager | Auto-destroy items | UNKNOWN |
| DeadlockWatcher | Detect frozen threads | UNKNOWN |

---

## THREAD SAFETY

**ThreadPool itself**: Fully thread-safe

**Guidelines for Tasks**:
```
✓ Use volatile fields for shared state
✓ Use AtomicInteger/AtomicLong for counters
✓ Use ConcurrentHashMap for shared maps
✓ Use CopyOnWriteArrayList for shared lists
✓ Synchronize critical sections

✗ Do NOT use non-concurrent collections
✗ Do NOT hold locks across thread boundaries
✗ Do NOT perform blocking I/O (use async)
✗ Do NOT create new threads (use ThreadPool)
```

---

## EXCEPTION HANDLING

**If task throws exception**:
```java
ThreadPool.execute(() -> {
    try {
        // Task code
    } catch (Exception e) {
        LOGGER.error("Task failed", e);
    }
});
```

**Uncaught exceptions**: 
- Logged by ThreadPool
- Thread survives (pool continues)
- UNKNOWN - exact error handling

---

## PERFORMANCE TUNING

### Increase Scheduled Pool Size
```
When: Many timers/events conflict
How: ThreadConfig.SCHEDULED_THREAD_POOL_SIZE = 16
Effect: More timers run concurrently
```

### Increase Instant Pool Size
```
When: Tasks queued too long
How: ThreadConfig.INSTANT_THREAD_POOL_SIZE = 200
Effect: More immediate tasks processed in parallel
```

### Monitor Queue Depth
```
Watch: ThreadPool queue size
If > 1000: Increase pool size
If = 0: Pool can be reduced
```

---

## DEADLOCK PREVENTION

**DeadlockWatcher**: Monitors for frozen threads

**How it works** (UNKNOWN - implementation details)

**Prevention tips**:
```
✓ Keep tasks short (< 100ms typical)
✓ Don't hold locks during I/O
✓ Don't wait for other tasks in a task
✓ Use timeouts for blocking operations
```

---

## COMMON PATTERNS

### Player Action Handler
```java
GameClient.onPacket(packet) {
    ThreadPool.execute(() -> {
        Player player = client.getPlayer();
        packet.handle(player);
        client.sendPacket(response);
    });
}
```

### Delayed Action
```java
player.setStunned(true);
ThreadPool.schedule(() -> {
    player.setStunned(false);
}, stunDurationMs);
```

### Periodic Check
```java
ThreadPool.scheduleAtFixedRate(() -> {
    for (Player player : world.getAllPlayers()) {
        player.updateStats();
    }
}, 1000, 10000);  // Every 10 seconds
```

---

## CONFIGURATION (commons.config.ThreadConfig)

**File**: UNKNOWN location  
**Typical Values**:
```
HIGH_PRIORITY_SCHEDULED_THREAD_POOL_SIZE=N
SCHEDULED_THREAD_POOL_SIZE=4-8
INSTANT_THREAD_POOL_SIZE=50-100
```

---

## SHUTDOWN

**On server stop**:
```
1. No new tasks accepted
2. Scheduled tasks: let finish (graceful)
3. Instant tasks: wait timeout or force stop
4. Connections: close gracefully
5. Shutdown complete
```

---

**Source of Truth**: ThreadPool class, initialization sequence  
**Verified**: 2026-08-23  
**Status**: VERIFIED (some UNKNOWN config values)
