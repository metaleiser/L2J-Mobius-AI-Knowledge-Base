# THREADING ARCHITECTURE

**Document Type**: SOURCE_ENTRY  
**Source of Truth**: \commons/threads/ThreadPool.java\, \commons/config/ThreadConfig.java\  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## 1. SOURCE-OF-TRUTH PACKAGES / CLASSES

\org.l2jmobius.commons.threads\:
- **ThreadPool** — Singleton managing 3 thread pool executors
- Provides static methods: \execute()\, \schedule()\, \scheduleAtFixedRate()\, \scheduleWithFixedDelay()\

\org.l2jmobius.commons.config\:
- **ThreadConfig** — Static fields for pool sizes, loaded from \./config/Threads.ini\

---

## 2. ARCHITECTURE SUMMARY — THREE POOLS

### Pool 1: High-Priority Scheduled
- **Type**: \ScheduledThreadPoolExecutor\
- **Size**: \ThreadConfig.HIGH_PRIORITY_SCHEDULED_THREAD_POOL_SIZE\
- **Priority**: \Thread.PRIORITY_8\
- **Policy**: \CallerRunsPolicy\
- **Use**: Critical server tasks (status checks, emergency shutdown)

### Pool 2: Standard Scheduled
- **Type**: \ScheduledThreadPoolExecutor\
- **Size**: \ThreadConfig.SCHEDULED_THREAD_POOL_SIZE\ (typical 4-8)
- **Priority**: Normal
- **Policy**: \CallerRunsPolicy\, pre-started core threads
- **Use**: Recurring timers, NPC respawns, cooldowns, event scheduling

### Pool 3: Instant Execution
- **Core Size**: \ThreadConfig.INSTANT_THREAD_POOL_SIZE\ (typical 50-100)
- **Max Size**: \Integer.MAX_VALUE\ (unlimited)
- **Queue**: \LinkedBlockingQueue\ (unbounded)
- **Keep-Alive**: 1 minute idle timeout
- **Use**: Immediate tasks (skill casts, combat, packets, AI ticks, data save)

---

## 3. CONFIGURATION / LOADING RELATIONSHIPS

\\\
ThreadConfig.load()          ← reads ./config/Threads.ini via ConfigReader
  └─ ThreadPool.init()       ← reads ThreadConfig static fields
       ├─ new ScheduledThreadPoolExecutor(highPrioritySize, ...)
       ├─ new ScheduledThreadPoolExecutor(scheduledSize, ...)
       └─ new ThreadPoolExecutor(instantCore, max, 1min, queue, ...)
\\\

**Shutdown sequence**: No new tasks accepted → scheduled tasks finish gracefully → instant tasks wait/timeout → connections close.

---

## 4. USEFUL EVIDENCE REFERENCES

- Scheduling API: \ThreadPool.execute(Runnable)\ → Instant Pool; \schedule(Runnable, delay)\ → Scheduled Pool
- Task managers using ThreadPool: \GameTimeTaskManager\ (100ms), \ItemLifeTimeTaskManager\ (1min), \ItemsAutoDestroyTaskManager\, \DeadlockWatcher\
- Config keys (from ThreadConfig): \ScheduledThreadPoolSize\, \InstantThreadPoolSize\, \ThreadsForLoading\
- UNKNOWN values: exact config file location for ThreadConfig (not in standard paths)

---

## 5. RELEVANT CROSS-LINKS

- [COMMONS_ARCHITECTURE.md](../COMMONS_ARCHITECTURE.md) — ThreadPool in commons context
- [CONFIGURATION/CONFIGURATION_SYSTEM.md](../CONFIGURATION/CONFIGURATION_SYSTEM.md) — ThreadConfig loading mechanism
- [ARCHITECTURE_OVERVIEW.md](../ARCHITECTURE_OVERVIEW.md) — task scheduling pattern

---

**Source of Truth**: ThreadPool class, initialization sequence  
**Verified**: 2026-08-23  
**Status**: VERIFIED

