# ARCHITECTURE OVERVIEW

**Source of Truth**: Code structure and \GameServer.java\ / \LoginServer.java\ initialization  
**Verified**: 2026-08-23  
**Status**: VERIFIED  
**Sprint 0.6B**: Compressed to remove material duplicated in domain-specific documents.

---

## 1. SYSTEM ARCHITECTURE

\\\
┌──────────────────────────────────────────────────────────┐
│                    L2 Game Clients                        │
│                (Network Protocol: L2)                     │
└───────────────────────┬──────────────────────────────────┘
                        │ TCP Port 7777 (Encrypted)
         ┌──────────────▼──────────────┐
         │ GameServer                  │
         │ org.l2jmobius.gameserver    │
         │  ├─ World (60k+ entities)   │
         │  ├─ Managers (52 singletons)│
         │  │  Sieges, Clans, Instances│
         │  │  AI, Zones, Events       │
         │  └─ Core Systems (15+)      │
         │     Entity, Skill, Item,    │
         │     Combat, Clan, Siege,    │
         │     Quest, Zone, ...        │
         └──────────────┬──────────────┘
                        │ Commons shared lib
         ┌──────────────▼──────────────┐
         │ Commons Library              │
         │ config/ crypt/ database/    │
         │ network/ threads/ time/     │
         │ ui/ util/                   │
         └──────────────┬──────────────┘
                        │ TCP Port 9014
         ┌──────────────▼──────────────┐
         │ LoginServer                 │
         │ org.l2jmobius.loginserver   │
         │ LoginController (Auth)      │
         │ GameServerTable (available) │
         │ SessionKey (tokens)         │
         └──────────────┬──────────────┘
                        │
         ┌──────────────▼──────────────┐
         │ Database (MySQL/MariaDB)    │
         │ Accounts, Characters,       │
         │ Items, Skills, Clans,       │
         │ Quests, World state         │
         └─────────────────────────────┘
\\\

**Cross-system relationships**: LoginServer authenticates users and directs them to GameServer via GameServerTable. GameServer delegates all shared infrastructure (config loading, database pooling, threading, network I/O, encryption) to Commons. GameServer loads static data from XML files at startup and persists runtime state to the database.

---

## 2. INITIALIZATION SEQUENCE (GameServer)

Essential steps — full detail in [GAMESERVER_ARCHITECTURE.md](GAMESERVER_ARCHITECTURE.md):

1. **GUI Setup** (optional) — \InterfaceConfig.load()\
2. **Logging** — \LogManager.readConfiguration()\
3. **Configuration** — \ConfigLoader.init()\ loads 16+ core \.ini\ files via \ConfigReader\
4. **Database** — \DatabaseFactory.init()\ configures HikariCP pool (20-100 connections)
5. **Thread Pools** — \ThreadPool.init()\ creates 3 pools (high-priority scheduled, standard scheduled, instant execution)
6. **Data Loading** — XML tables (items, skills, NPC templates, etc.) loaded by \*Data\ singletons
7. **Script Engine** — \ScriptEngine.getInstance().load()\ loads \Scripts.xml\ and compiles handlers
8. **World / Managers** — World region grid initialized, manager singletons instantiated
9. **Network** — \ConnectionManager\ opens TCP port 7777 for client connections
10. **Ready** — Server announces availability to LoginServer

---

## 3. KEY ARCHITECTURAL PATTERNS

| Pattern | Where Used | Description |
|---------|-----------|-------------|
| Singleton | Managers | ~52 classes with \getInstance()\; partial audit in [MANAGERS_INDEX.md](INDEXES/MANAGERS_INDEX.md) |
| Observer | Event system | \EventDispatcher\ — \EventType\ registered, listeners attach callbacks, events trigger execution |
| Handler | Packets/Scripts | \ClientPacket → GamePacketHandler (dispatcher) → specific handler → Manager/System\ |
| Data Access | XML/SQL loading | \XML File → *Data singleton loader → Memory Objects → Singleton Cache → Systems\ |
| Task Scheduling | Async operations | \Component → ThreadPool.schedule(task) → scheduled/instant pool → async execution\ |

---

## 4. CRITICAL CROSS-SYSTEM RELATIONSHIPS

- **Zones** define PvP rules, effects, and geographic regions — referenced by Combat, Siege, World systems.
- **Skills** use \AbnormalType\ for stacking/replacement in \EffectList\ — see [SKILLS/SKILL_SEMANTIC_REFERENCE.md](SKILLS/SKILL_SEMANTIC_REFERENCE.md) and [REFERENCE/ABNORMAL_TYPE_CATALOG.md](REFERENCE/ABNORMAL_TYPE_CATALOG.md).
- **Quests** are script-driven, extend base handler classes — see [QUESTS/QUEST_ENGINE_REFERENCE.md](QUESTS/QUEST_ENGINE_REFERENCE.md).
- **Scripting** dynamically loads via \ScriptEngine\ + \ScriptExecutor\ (Java compilation at runtime) — see [SCRIPTING/SCRIPT_ENGINE.md](SCRIPTING/SCRIPT_ENGINE.md).
- **Database** connection pool managed by Commons \DatabaseFactory\ (HikariCP) — see [DATABASE/DATABASE_ARCHITECTURE.md](DATABASE/DATABASE_ARCHITECTURE.md).
- **Network** uses Netty NIO (channel-oriented, non-blocking) — see [NETWORK/NETWORK_ARCHITECTURE.md](NETWORK/NETWORK_ARCHITECTURE.md).

---

## 5. AUTHORITATIVE DOCUMENTS

| Domain | Document |
|--------|----------|
| Game Server Detail | [GAMESERVER_ARCHITECTURE.md](GAMESERVER_ARCHITECTURE.md) |
| Commons Library | [COMMONS_ARCHITECTURE.md](COMMONS_ARCHITECTURE.md) |
| Network Protocol | [NETWORK/NETWORK_ARCHITECTURE.md](NETWORK/NETWORK_ARCHITECTURE.md) |
| Threading Model | [THREADING/THREADING_ARCHITECTURE.md](THREADING/THREADING_ARCHITECTURE.md) |
| Database Design | [DATABASE/DATABASE_ARCHITECTURE.md](DATABASE/DATABASE_ARCHITECTURE.md) |
| Configuration | [CONFIGURATION/CONFIGURATION_SYSTEM.md](CONFIGURATION/CONFIGURATION_SYSTEM.md) |
| Build & Deploy | [BUILD_AND_DEPLOYMENT.md](BUILD_AND_DEPLOYMENT.md) |
| Source vs Runtime | [SOURCE_VS_RUNTIME.md](SOURCE_VS_RUNTIME.md) |

---

**Source of Truth**: GameServer.java, source structure analysis  
**Verified**: 2026-08-23  
**Status**: VERIFIED
