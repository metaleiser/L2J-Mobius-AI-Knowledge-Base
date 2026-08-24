# PROJECT OVERVIEW

**Project**: L2J Mobius CT 2.6 HighFive  
**Edition**: Chaotic Throne 2.6 HighFive Part 5  
**Type**: MMORPG Server (Lineage 2)  
**Language**: Java  
**Build System**: Apache Ant  
**Java Version Required**: Java 25  

---

## SOURCE OF TRUTH

**Location**: Raíz del proyecto del servidor en este workspace (subcarpeta `L2J_Mobius_CT_2.6_HighFive`)  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## ARCHITECTURE AT A GLANCE

```
┌─────────────────────────────────────────────────────────┐
│ L2J Mobius Server Project                                │
├─────────────────────────────────────────────────────────┤
│                                                           │
│  ┌──────────────────┐         ┌──────────────────┐      │
│  │   Login Server   │◄───────►│   Game Server    │      │
│  │   :9014          │         │   :7777          │      │
│  └──────────────────┘         └──────────────────┘      │
│         ▲                              ▲                  │
│         │                              │                  │
│   ┌─────┴──────────┐            ┌─────┴──────────┐      │
│   │ Auth Clients   │            │ Game Clients   │      │
│   └────────────────┘            └────────────────┘      │
│                                                           │
│  ┌──────────────────────────────────────────────┐       │
│  │         Shared Commons Library                │       │
│  │  (Config, Database, Network, Threading, etc) │       │
│  └──────────────────────────────────────────────┘       │
│                                                           │
│  ┌──────────────────────────────────────────────┐       │
│  │  MySQL/MariaDB Database (HikariCP Pool)      │       │
│  └──────────────────────────────────────────────┘       │
│                                                           │
│  ┌──────────────────────────────────────────────┐       │
│  │  Data Files (XML, Scripts, Geodata)          │       │
│  └──────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────┘
```

---

## PROJECT COMPONENTS

### 1. LOGIN SERVER
Manages player authentication and session creation.

**Key Files**:
- `org.l2jmobius.loginserver.LoginServer` - Entry point
- Port: 9014 (default)
- Authenticates against database
- Issues SessionKey to validated clients
- Maintains list of available Game Servers

**Responsibilities**:
- Account validation
- Session creation
- Game Server registration

---

### 2. GAME SERVER
Main game world server.

**Key Files**:
- `org.l2jmobius.gameserver.GameServer` - Entry point
- Port: 7777 (default)
- Hosts all game world entities
- Manages all game systems
- Persists state to database

**Responsibilities**:
- Player management
- NPC management
- Combat systems
- Item systems
- Quest systems
- Clan systems
- Siege systems
- And 20+ more systems

---

### 3. COMMONS LIBRARY
Shared infrastructure used by both servers.

**Packages**:
- `org.l2jmobius.commons.config` - Configuration management
- `org.l2jmobius.commons.crypt` - Blowfish encryption
- `org.l2jmobius.commons.database` - HikariCP connection pool
- `org.l2jmobius.commons.network` - TCP connection management
- `org.l2jmobius.commons.threads` - Thread pool management
- `org.l2jmobius.commons.time` - Scheduling utilities
- `org.l2jmobius.commons.ui` - Optional GUI
- `org.l2jmobius.commons.util` - General utilities

---

### 4. TOOLS
Administrative utilities (not part of running servers).

**Tools**:
- `AccountManager` - Create/manage accounts
- `DatabaseInstaller` - Database setup
- `GameServerRegister` - Register Game Server to Login Server
- `Search` - Database search utility

---

## BUILD CONFIGURATION

**Build File**: `build.xml`  
**Build System**: Apache Ant 1.8.2+  
**Source Directory**: `java/org/l2jmobius/`  
**Output Directory**: `build/bin/` → `build/dist/`

### Build Targets

| Target | Output | Excludes |
|--------|--------|----------|
| `LoginServer.jar` | `build/dist/libs/LoginServer.jar` | gameserver/*, DatabaseInstaller, Search |
| `GameServer.jar` | `build/dist/libs/GameServer.jar` | loginserver/*, AccountManager, GameServerRegister |

Both JARs require shared libraries in `dist/libs/`

---

## KEY TECHNOLOGIES

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Database | JDBC driver/URL configurable; instalador con URLs JDBC MySQL | Character/item/clan persistence |
| Connection Pool | HikariCP | Database connection management |
| Threading | Java ScheduledThreadPoolExecutor | Async task execution |
| Encryption | Blowfish | Network packet encryption |
| Scripting | Java Runtime Compilation | Dynamic script loading |
| Build | Apache Ant | Compilation and JAR creation |
| Logging | Java Logging | Server diagnostics |

---

## DATABASE CHARACTERISTICS

**Type**: Relational (MySQL/MariaDB)  
**Connection Pool**: HikariCP  
**Pool Size**: Configurable (typical: 20-100 connections)  
**Tables**: 10+ core tables  
**Data Files**: 40+ XML configuration files  

---

## NETWORK CONFIGURATION

| Server | Port | Purpose |
|--------|------|---------|
| Login Server | 9014 | Client authentication |
| Game Server | 7777 | Game world (client gameplay) |
| Internal | Varies | Game-to-Login communication |

**Protocol**: TCP, Blowfish encrypted, L2 protocol variant

---

## DIRECTORY STRUCTURE

```
L2J_Mobius_CT_2.6_HighFive/
├── build.xml                 ← Ant build file
├── java/                     ← Source code
│   └── org/l2jmobius/
│       ├── commons/          ← Shared library
│       ├── gameserver/       ← Game server code
│       ├── loginserver/      ← Login server code
│       ├── tools/            ← Utilities
│       └── log/              ← Logging setup
├── dist/                     ← Distribution/runtime files
│   ├── game/                 ← Game Server runtime
│   ├── login/                ← Login Server runtime
│   ├── db_installer/         ← DB installer
│   └── libs/                 ← Required JAR libraries
├── launcher/                 ← IDE launch configs
└── readme.txt               ← Project notes
```

---

## PERFORMANCE TARGETS

- **Max Concurrent Players**: 2000-3000 (per design)
- **Database Connections**: Typically 20-100 active
- **Thread Pool Size**: Game Server typically 50-100 threads
- **Network Bandwidth**: UNKNOWN (server-dependent)

---

## VERSION INFORMATION

- **Project Version**: CT 2.6 HighFive Part 5
- **Java Minimum**: Java 25
- **Build Date**: Unknown (runtime configurable)
- **Last Verified**: 2026-08-23

---

## WHAT TO READ NEXT

1. [ARCHITECTURE_OVERVIEW.md](ARCHITECTURE_OVERVIEW.md) - How components interact
2. [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md) - Detailed directory layout
3. [GAMESERVER_ARCHITECTURE.md](GAMESERVER_ARCHITECTURE.md) - Game Server specifics
4. [LOGINSERVER_ARCHITECTURE.md](LOGINSERVER_ARCHITECTURE.md) - Login Server specifics

---

**Source of Truth**: Verified against actual code structure  
**Verified**: 2026-08-23  
**Status**: VERIFIED
