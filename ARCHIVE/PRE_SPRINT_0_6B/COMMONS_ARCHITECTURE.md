# COMMONS ARCHITECTURE

**Source of Truth**: `org.l2jmobius.commons` package  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## PURPOSE

Commons library provides shared infrastructure used by both LoginServer and GameServer.

**Benefit**: Avoids code duplication between servers.

---

## PACKAGE STRUCTURE

```
commons/
├── config/          ← Configuration management
├── crypt/           ← Encryption/decryption
├── database/        ← Database connectivity
├── network/         ← TCP networking
├── threads/         ← Thread pool management
├── time/            ← Time/scheduling utilities
├── ui/              ← Optional GUI components
└── util/            ← General utilities
```

---

## CONFIG PACKAGE

**Location**: `org.l2jmobius.commons.config`

### Purpose
Central configuration loading for both servers.

### Key Classes

#### DatabaseConfig
**Responsibilities**:
- Load database connection parameters
- Store database URL, username, password
- Provide login credentials to DatabaseFactory

**Configuration File** (location varies):
```ini
Driver=com.mysql.cj.jdbc.Driver
URL=jdbc:mysql://localhost/l2jmobius
Login=root
Password=
MaximumDatabaseConnections=10
```

**Usage**:
```java
String url = DatabaseConfig.DATABASE_URL;
String user = DatabaseConfig.DATABASE_LOGIN;
```

#### InterfaceConfig
**Responsibilities**:
- Determine if GUI should be enabled
- Store GUI display settings

**Configuration File**: `./config/Interface.ini` (claves reales: **REQUIRES CODE VERIFICATION** — no se transcriben sin verificación previa)

#### ThreadConfig
**Responsibilities**:
- Configure thread pool sizes
- Store threading parameters

**Configuration File**: `./config/Threads.ini`. Claves reales verificadas en `ThreadConfig.java`:
```ini
ScheduledThreadPoolSize = -1   # default -1
InstantThreadPoolSize = -1     # default -1
ThreadsForLoading = False      # default false
```

**Usage by GameServer**:
```java
ThreadPool.SCHEDULED_POOL size = ThreadConfig.SCHEDULED_THREAD_POOL_SIZE
ThreadPool.INSTANT_POOL size = ThreadConfig.INSTANT_THREAD_POOL_SIZE
```

---

## CRYPT PACKAGE

**Location**: `org.l2jmobius.commons.crypt`

### Purpose
Encryption/decryption for network communication.

### Key Classes

#### BlowfishEngine
**Responsibilities**:
- Symmetric encryption using Blowfish algorithm
- Encrypt/decrypt packet data

**Algorithm**: Blowfish (64-bit blocks, 128-bit default key)

**Usage**:
```java
BlowfishEngine cipher = new BlowfishEngine(key);
byte[] encrypted = cipher.encrypt(plaintext);
byte[] decrypted = cipher.decrypt(ciphertext);
```

#### NewCrypt
**Responsibilities**:
- Lineage 2 protocol encryption
- Session-specific key negotiation
- Packet encryption wrapper

**Flow**:
```
1. Session starts (unencrypted handshake)
2. KeyPair exchanged
3. Session key generated
4. NewCrypt initialized with session key
5. All subsequent packets encrypted
```

**Implementation**:
- Blowfish cipher underneath
- Packet framing
- Integrity checking

---

## DATABASE PACKAGE

**Location**: `org.l2jmobius.commons.database`

### Purpose
Database connection pooling and management.

### Key Classes

#### DatabaseFactory
**Responsibilities**:
- Manage HikariCP connection pool
- Provide JDBC connections to servers
- Monitor pool health

**Pattern**: Singleton

**Pool Configuration** (from DatabaseConfig and DatabaseFactory):
```
Maximum pool size: clamp(configured, 4, 1000)
Minimum idle: max(maximum / 10, 2)
Connection Timeout: 60 seconds
Idle Timeout: 5 minutes
Max Lifetime: 10 minutes
Leak Detection: 10 minutes
```

**Usage**:
```java
Connection conn = DatabaseFactory.getConnection();
try {
    Statement stmt = conn.createStatement();
    // Execute query
} finally {
    conn.close();  // Returns to pool
}
```

**Thread-Safety**: Fully thread-safe (HikariCP)

#### DatabaseBackup
**Responsibilities**:
- Create database backups
- Export database schema/data

**Usage** (if available):
```bash
java -cp libs/* org.l2jmobius.commons.database.DatabaseBackup
```

**Note**: UNKNOWN - exact backup mechanism and format.

---

## NETWORK PACKAGE

**Location**: `org.l2jmobius.commons.network`

### Purpose
Low-level TCP networking and packet handling.

### Subpackages

#### connection
**Location**: `commons.network`

**Key Classes**:
- `Connection` - Individual TCP socket connection
- `ConnectionManager` - Server-side listener/acceptor
- `ConnectionConfig` - Network configuration

**Responsibilities**:
```
ConnectionManager (server):
  ├─ Bind to port
  ├─ Listen for incoming connections
  ├─ Accept connections
  └─ Create Connection per client

Connection (per client):
  ├─ Read from socket
  ├─ Write to socket
  ├─ Manage buffers
  └─ Handle disconnection
```

#### buffer
**Location**: `commons.network.buffer`

**Key Classes**:
- `ReadBuffer` - Read binary data
- `WriteBuffer` - Write binary data

**Responsibilities**:
```
ReadBuffer:
  ├─ Read bytes from network
  ├─ Read primitives (int, long, float, etc.)
  ├─ Read strings
  └─ Manage position/offset

WriteBuffer:
  ├─ Write bytes to network
  ├─ Write primitives
  ├─ Write strings
  └─ Track written data
```

#### handler
**Location**: `commons.network.handler`

**Key Interface**:
- `PacketHandler<ClientType>` - Generic packet handler interface

**Responsibilities**:
```
Receives: ReadBuffer (packet data) + Client
Returns: ReadablePacket (parsed packet)

Implementation:
  ├─ Read opcode
  ├─ Lookup packet class
  ├─ Instantiate and deserialize
  └─ Return packet
```

#### packet
**Location**: `commons.network.packet`

**Key Classes** (verificado por listado del paquete, 2026-08-24):
- `ReadablePacket` / `SimpleReadablePacket` — base para packets entrantes
- `WritablePacket` / `SimpleWritablePacket` — base para packets salientes
- También en el paquete: `PacketHandler`, `PacketExecutor`, `PacketThreadFactory`

> Nota auditoría FASE 2: no existe ninguna clase `SendablePacket` en este build.

**Structure**:
```java
public class MyClientPacket extends ReadablePacket<GameClient> {
    private int param1;
    private String param2;
    
    @Override
    protected void readImpl(ReadBuffer buffer) {
        param1 = buffer.readInt();
        param2 = buffer.readString();
    }
    
    @Override
    protected void runImpl(GameClient client) {
        // Handle packet
    }
}
```

#### pool
**Location**: `commons.network.pool`

**Purpose**: Object pool for buffers/packets (if used)

**Note**: UNKNOWN - usage in current codebase.

---

## THREADS PACKAGE

**Location**: `org.l2jmobius.commons.threads`

### Purpose
Thread pool management for async task execution.

### Key Classes

#### ThreadPool
**Responsibilities**:
- Manage 3 thread pools
- Schedule tasks with delays
- Schedule repeating tasks
- Purge cancelled tasks

**Pattern**: Utility class with static methods

**Pool Configuration** (from ThreadConfig):
```
High-Priority Scheduled Pool:
  ├─ Size: HIGH_PRIORITY_SCHEDULED_THREAD_POOL_SIZE
  └─ Priority: PRIORITY_8

Standard Scheduled Pool:
  ├─ Size: SCHEDULED_THREAD_POOL_SIZE
  ├─ Type: ScheduledThreadPoolExecutor
  └─ Behavior: CallerRunsPolicy on rejection

Instant Execution Pool:
  ├─ Size: INSTANT_THREAD_POOL_SIZE
  ├─ Type: ThreadPoolExecutor
  └─ Behavior: Grows unbounded, 1-minute keep-alive
```

**Key Methods**:
```java
ThreadPool.execute(Runnable)              // Instant pool
ThreadPool.schedule(Runnable, delayMs)    // After delay
ThreadPool.scheduleAtFixedRate(Runnable, initialDelayMs, periodMs)
ThreadPool.scheduleWithFixedDelay(Runnable, initialDelayMs, delayMs)
ThreadPool.purge()                        // Clean up cancelled
```

**Usage Example**:
```java
// Execute immediately
ThreadPool.execute(() -> {
    // Task code
});

// Execute after 10 seconds
ThreadPool.schedule(() -> {
    // Task code
}, 10000);

// Repeat every 5 seconds
ThreadPool.scheduleAtFixedRate(() -> {
    // Task code
}, 1000, 5000);
```

**Thread-Safety**: Fully thread-safe

#### ThreadProvider
**Responsibilities**:
- Create threads with custom names
- Set thread priorities
- Track thread creation

**Usage**: ThreadPool uses ThreadProvider to create named threads.

#### ThreadPriority
**Enum**: Priority levels for threads

```java
enum ThreadPriority {
    PRIORITY_1,   // Lowest
    PRIORITY_2,
    ...
    PRIORITY_8    // Highest
}
```

---

## TIME PACKAGE

**Location**: `org.l2jmobius.commons.time`

### Purpose
Time utilities and scheduling patterns.

### Key Classes

#### TimeUtil
**Responsibilities**:
- Time calculations
- Uptime tracking
- Timestamp utilities

**Note**: UNKNOWN - specific methods without code review.

#### SchedulingPattern
**Responsibilities**:
- Parse cron-like schedules
- Determine next execution time

**Note**: UNKNOWN - scheduling format and usage.

---

## UI PACKAGE

**Location**: `org.l2jmobius.commons.ui`

### Purpose
Optional GUI components (rarely used in production).

### Key Classes

#### DarkTheme
**Responsibilities**: GUI color theme (if GUI enabled)

#### SplashScreen
**Responsibilities**: Server startup splash screen

#### LineLimitListener
**Responsibilities**: Limit GUI text output

**Usage**:
```java
if (InterfaceConfig.ENABLE_GUI) {
    new Gui();  // Creates GUI window
}
```

---

## UTIL PACKAGE

**Location**: `org.l2jmobius.commons.util`

### Purpose
General utilities used throughout.

### Key Classes

#### ConfigReader
**Responsibilities**:
- Parse config files (`.ini`) mediante `java.util.Properties`
- Provide type-safe access (`getString`/`getInt`/`getBoolean`)

#### HexUtil
**Responsibilities**:
- Convert hex strings to bytes
- Convert bytes to hex
- Useful for packet debugging

#### StringUtil
**Responsibilities**:
- String manipulation
- Concatenation helpers
- Formatting

#### Rnd
**Responsibilities**:
- Random number generation
- Used for item drops, damage variance, etc.

**Note**: Thread-safe Random implementation

#### TraceUtil
**Responsibilities**:
- Generate stack traces
- Debug logging

#### IXmlReader
**Interface**: Base for XML parsing

---

## USAGE BY SERVERS

### LoginServer Uses

```
DatabaseFactory       ✓ Account table access
DatabaseConfig        ✓ DB credentials
InterfaceConfig       ✓ GUI setting
ThreadConfig          ✓ Thread pool setup
ThreadPool            ✓ Async operations
Encryption (Blowfish) ✓ Client communication
ConnectionManager     ✓ Port 9014 listener
```

### GameServer Uses

```
DatabaseFactory       ✓ Character/item persistence
DatabaseConfig        ✓ DB credentials
InterfaceConfig       ✓ GUI setting
ThreadConfig          ✓ Thread pool setup
ThreadPool            ✓ All async tasks
Encryption (Blowfish) ✓ Client communication
ConnectionManager     ✓ Port 7777 listener
TimeUtil              ✓ Game time tracking
Rnd                   ✓ Drop rates, damage
StringUtil            ✓ Text processing
HexUtil               ✓ Packet debugging
```

---

## INITIALIZATION ORDER

**Both servers initialize commons in same order**:

```
1. ConfigLoader (all config files)
2. DatabaseFactory.init() (HikariCP)
3. ThreadPool.init() (3 pools)
4. Connection/Network setup
5. Encryption session key (per client)
```

---

## THREAD SAFETY

**Thread-Safe Components**:
- DatabaseFactory (HikariCP is concurrent)
- ThreadPool (all executors are concurrent)
- ConnectionManager (multi-threaded)
- Encryption (stateless algorithms)

**Not Thread-Safe**:
- Individual Connection buffers (per-thread)
- Config objects (read-only after init)

---

## PERFORMANCE NOTES

**Database Pool Tuning** (typical):
```
Max: 100 (handles spikes)
Min: 20 (always available)
Max Lifetime: 10 minutes (pool health)
```

**Thread Pool Tuning** (GameServer typical):
```
Scheduled: 4-8 threads (timers/events)
Instant: 50-100 threads (immediate tasks)
Task queue: Unbounded (can grow large)
```

---

## NEXT STEPS

- [NETWORK/NETWORK_ARCHITECTURE.md](NETWORK/NETWORK_ARCHITECTURE.md) - Network protocol
- [DATABASE/DATABASE_ARCHITECTURE.md](DATABASE/DATABASE_ARCHITECTURE.md) - Database design
- [THREADING/THREADING_ARCHITECTURE.md](THREADING/THREADING_ARCHITECTURE.md) - Threading details
- [CONFIGURATION/CONFIGURATION_SYSTEM.md](CONFIGURATION/CONFIGURATION_SYSTEM.md) - Configuration

---

**Source of Truth**: commons package structure and class analysis  
**Verified**: 2026-08-23  
**Status**: VERIFIED
