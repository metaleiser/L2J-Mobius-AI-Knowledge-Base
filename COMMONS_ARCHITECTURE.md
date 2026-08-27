# COMMONS ARCHITECTURE

**Source of Truth**: \org.l2jmobius.commons\ package  
**Verified**: 2026-08-23  
**Status**: VERIFIED  
**Sprint 0.6B**: Compressed to remove duplicated generic material.

---

## 1. PURPOSE

Commons library provides shared infrastructure used by both LoginServer and GameServer. Avoids code duplication between servers.

---

## 2. PACKAGE MAP

\\\
commons/
├── config/      ← Configuration management
│   ├── DatabaseConfig    (DB credentials from ./config/Database.ini)
│   ├── InterfaceConfig   (GUI on/off from ./config/Interface.ini)
│   └── ThreadConfig      (pool sizes from ./config/Threads.ini)
├── crypt/       ← Encryption/decryption
│   ├── BlowfishEngine    (64-bit blocks, 128-bit key)
│   └── NewCrypt          (L2 protocol: handshake → key exchange → encrypted)
├── database/    ← DatabaseFactory (HikariCP wrapper singleton)
├── network/     ← ConnectionManager (TCP acceptor), SelectorConfig
├── threads/     ← ThreadPool (3 executors), configurable sizes
├── time/        ← TimeUtil (game time tracking)
├── ui/          ← Gui (optional Swing window)
└── util/        ← ConfigReader, Rnd, IXmlReader, HexUtil, StringUtil, TraceUtil
\\\

---

## 3. MAJOR CLASSES AND ACTUAL ROLES

### Config Package (\commons.config\)
- **DatabaseConfig** — DB connection parameters (Driver, URL, Login, Password, MaxConnections)
- **InterfaceConfig** — GUI enable/disable flag
- **ThreadConfig** — Thread pool sizes for scheduled and instant pools

### Crypt Package (\commons.crypt\)
- **BlowfishEngine** — Symmetric Blowfish encryption
- **NewCrypt** — Lineage 2 protocol: session handshake → KeyPair exchange → session key → encrypted packets

### Database Package (\commons.database\)
- **DatabaseFactory** — HikariCP connection pool singleton. \init()\ reads DatabaseConfig, creates pool.
- **ConnectionFactory** — Connection acquisition wrapper

### Network Package (\commons.network\)
- **ConnectionManager** — TCP server socket acceptor, manages client connections
- Uses Netty NIO internally (channel-oriented, non-blocking I/O)

### Threads Package (\commons.threads\)
- **ThreadPool** — Singleton managing 3 executors:
  1. High-priority ScheduledPool (PRIORITY_8, CallerRunsPolicy)
  2. Standard ScheduledPool (normal priority, pre-started core threads)
  3. Instant Pool (core size configurable, max Integer.MAX_VALUE, 1-min keep-alive)
- Configurable via \ThreadConfig\ fields

### Util Package (\commons.util\)
- **ConfigReader** — \.ini\ file parser (wrapper around \java.util.Properties\)
- **Rnd** — Thread-safe random number generation
- **IXmlReader** — Interface for XML document parsing
- **HexUtil** / **StringUtil** / **TraceUtil** — Formatting and debug helpers

---

## 4. LOGINSERVER vs GAMESERVER USAGE DISTINCTION

| Component | LoginServer | GameServer |
|-----------|-------------|------------|
| DatabaseFactory | Account table access | Character/item/clan persistence |
| ThreadPool | Limited async operations | All async tasks system-wide |
| BlowfishEngine + NewCrypt | Client authentication | Client communication (7777) |
| ConnectionManager | Port 9014 listener | Port 7777 listener |
| ConfigReader | All config files | All config files |
| InterfaceConfig | GUI on/off | GUI on/off |
| Rnd | — | Drop rates, damage variance |
| TimeUtil | — | Game time tracking |
| StringUtil / HexUtil | — | Text processing, packet debugging |

---

## 5. INITIALIZATION RELATIONSHIP

Both servers initialize commons in the same order:

\\\
1. ConfigLoader.init()        ← loads all config files via ConfigReader
2. DatabaseFactory.init()     ← HikariCP connection pool configured
3. ThreadPool.init()          ← 3 thread pools created
4. ConnectionManager.start()  ← TCP listener for client connections
5. Encryption per client      ← session key negotiation on connect
\\\

---

## 6. ARCHITECTURAL DEPENDENCIES

- **CONFIGURATION/** — detailed config file inventory: [CONFIGURATION/CONFIGURATION_SYSTEM.md](CONFIGURATION/CONFIGURATION_SYSTEM.md)
- **DATABASE/** — database design and connection flow: [DATABASE/DATABASE_ARCHITECTURE.md](DATABASE/DATABASE_ARCHITECTURE.md)
- **NETWORK/** — network protocol layers and packet structure: [NETWORK/NETWORK_ARCHITECTURE.md](NETWORK/NETWORK_ARCHITECTURE.md)
- **THREADING/** — thread pool architecture and scheduling: [THREADING/THREADING_ARCHITECTURE.md](THREADING/THREADING_ARCHITECTURE.md)

---

**Source of Truth**: commons package structure and class analysis  
**Verified**: 2026-08-23  
**Status**: VERIFIED
