# ARCHITECTURE OVERVIEW

**Source of Truth**: Code structure and GameServer.java initialization  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## SYSTEM ARCHITECTURE

```
┌──────────────────────────────────────────────────────────────┐
│                      L2 Game Clients                          │
│                   (Network Protocol: L2)                      │
└────────────────────────┬─────────────────────────────────────┘
                         │ TCP Port 7777 (Encrypted)
                         │
        ┌────────────────▼────────────────┐
        │                                 │
    ┌───┴─────────────────────────────────┴───┐
    │   Game Server                            │
    │   org.l2jmobius.gameserver.GameServer    │
    └───┬─────────────────────────────────────┬───┐
        │                                     │   │
   ┌────┴──────────────┐          ┌──────────┴──┐│
   │ World             │          │ Managers   ││
   │ (60,000+ entities)│          │ (52 singletons)│
   │ Creatures/Items   │          │ Sieges, Clans │
   └────┬──────────────┘          │ Instances, AI │
        │                         └───────────────┘
   ┌────┴────────────────────────────────────────┐
   │ Core Systems                                │
   │ ├─ Entity System (Creature hierarchy)      │
   │ ├─ Skill System (Learning, Casting)        │
   │ ├─ Item System (Inventory, Enchant)        │
   │ ├─ Combat System (PvP, PvE)               │
   │ ├─ Clan System (Wars, Halls)              │
   │ ├─ Siege System (Castles, Forts, Halls)   │
   │ ├─ Quest System                            │
   │ ├─ Zone System (PvP rules, effects)       │
   │ └─ ... (15+ more systems)
   └─────────┬──────────────────────────────────┘
             │
   ┌─────────┴──────────────────────────────┐
   │ Commons Library (Shared)                │
   │ ├─ ThreadPool (3 pools, async tasks)   │
   │ ├─ DatabaseFactory (HikariCP)          │
   │ ├─ ConnectionManager (Network)         │
   │ ├─ ConfigLoader (Runtime config)       │
   │ └─ Encryption (Blowfish)               │
   └─────────┬──────────────────────────────┘
             │ TCP Port 9014
   ┌─────────▼──────────────────────────────┐
   │ Login Server                            │
   │ org.l2jmobius.loginserver.LoginServer  │
   │ ├─ LoginController (Auth)              │
   │ ├─ GameServerTable (Available servers) │
   │ └─ SessionKey (Auth tokens)            │
   └─────────┬──────────────────────────────┘
             │
   ┌─────────▼──────────────────────────────┐
   │ Database (MySQL/MariaDB)               │
   │ ├─ Accounts & Characters               │
   │ ├─ Items & Skills                      │
   │ ├─ Clans & Relations                   │
   │ └─ ... (10 core tables)               │
   └────────────────────────────────────────┘
```

---

## INITIALIZATION SEQUENCE

When GameServer starts, initialization happens in this order:

```
1. GUI Setup (optional)
   └─ InterfaceConfig.load()

2. Logging System
   └─ LogManager.readConfiguration()

3. Configuration Loading
   └─ ConfigLoader.init()
      ├─ GeneralConfig
      ├─ ServerConfig
      ├─ PlayerConfig
      ├─ RatesConfig
      ├─ PvpConfig
      ├─ NpcConfig
      └─ Configuration classes loaded from .ini files (16 main load calls)

4. Database System
   └─ DatabaseFactory.init()
      ├─ HikariCP connection pool configured
      └─ 20-100 connections ready

5. Thread Pool System
   └─ ThreadPool.init()
      ├─ High-Priority Scheduled Pool
      ├─ Standard Scheduled Pool
      └─ Instant Execution Pool

6. Core ID Management
   └─ IdManager.getInstance()

7. Scripting & Events
   ├─ EventDispatcher.getInstance()
   └─ ScriptEngine.getInstance()

8. World Initialization
   ├─ InstanceManager.getInstance()
   ├─ World.init()
   ├─ MapRegionData.getInstance()
   ├─ AnnouncementsTable.getInstance()
   └─ GlobalVariablesManager.getInstance()

9. Data Loading (Phase 1)
   ├─ CategoryData
   ├─ CubicData
   ├─ DynamicExpRateData
   └─ SecondaryAuthData

10. Skills Loading
    ├─ EffectHandler.getInstance().executeScript()
    ├─ EnchantSkillGroupsData
    ├─ SkillTreeData
    ├─ SkillData
    └─ PetSkillData

11. Items Loading
    ├─ ItemData
    ├─ EnchantItemData
    ├─ EnchantItemOptionsData
    ├─ ElementalAttributeData
    ├─ OptionData
    ├─ RecipeData
    ├─ MultisellData
    ├─ BuyListData
    ├─ FishData
    └─ ... (more item-related data)

12. Character System
    ├─ ClassListData
    ├─ InitialEquipmentData
    ├─ ExperienceData
    ├─ PlayerTemplateData
    ├─ CharInfoTable
    ├─ AdminData
    └─ CaptchaManager

13. Clan System
    ├─ ClanTable
    ├─ CHSiegeManager
    ├─ ClanHallTable
    └─ ClanHallAuctionManager

14. Geodata System
    └─ GeoEngine.getInstance()

15. NPC System
    ├─ NpcData
    ├─ SpawnData
    ├─ TeleporterData
    ├─ DoorData
    ├─ StaticObjectData
    └─... (NPC-related loading)

16. Game Systems
    ├─ SiegeManager & related managers
    ├─ DimensionalRiftManager
    ├─ ItemsOnGroundManager
    └─ TaskManagers

17. Network Listeners
    └─ ConnectionManager.start()
       ├─ Listens on port 7777 for clients
       └─ Registers packet handlers

18. Login Server Connection
    └─ LoginServerThread starts
       └─ Connects to LoginServer on port 9014
```

---

## COMPONENT INTERACTIONS

### Client-to-Game Server Flow

```
Client connects (TCP:7777)
       │
       ├─ GameClient instance created
       ├─ Session established
       ├─ Player character loaded from DB
       ├─ Player entity spawned in World
       │
       └─ Packets exchanged:
          ├─ ClientPacket (from client) → Handler → Database/World update
          ├─ ServerPacket (to client)   ← Manager/System update
          └─ ... (continuous exchange)
```

### Game Server-to-Login Server Flow

```
GameServer starts
       │
       ├─ LoginServerThread created
       ├─ Connects to LoginServer (TCP:9014)
       ├─ Registers itself in GameServerTable
       │
       └─ During client auth:
          ├─ Client connects to LoginServer
          ├─ LoginServer validates credentials
          ├─ SessionKey issued
          ├─ Client gets GameServer info
          └─ Client connects to GameServer with SessionKey
```

### World Management Flow

```
World (singleton)
       │
       ├─ Maintains ConcurrentHashMap of all creatures
       ├─ Organizes entities by WorldRegion (grid-based)
       │
       ├─ WorldRegion (geographical division)
       │   ├─ Contains nearby creatures
       │   └─ Optimizes visibility/broadcasts
       │
       └─ Creature visibility:
          ├─ Only entities in nearby regions sent to client
          └─ Regional broadcasts (e.g., area chat) limited
```

### Database Persistence Flow

```
Entity State Changes
       │
       └─ Async Task via ThreadPool
          └─ DatabaseFactory.getConnection()
             └─ HikariCP pool
                └─ MySQL/MariaDB
                   └─ Updates persisted
```

---

## MANAGER RESPONSIBILITIES

Managers under `gameserver/managers/` handle domain-specific logic; the current directory count is 58 classes, with 52 public `getInstance()` declarations and 6 classes requiring separate treatment.

| Manager Category | Examples | Purpose |
|------------------|----------|---------|
| **World Management** | InstanceManager, World | Entity lifecycle, visibility |
| **Combat/Duel** | DuelManager, AntiFeedManager | PvP mechanics |
| **Clan/Siege** | ClanTable, SiegeManager, CastleManager | Clan warfare, sieges |
| **Economic** | ItemManager, RecipeManager, MailManager | Trading, items, mail |
| **Events** | EventDropManager, SevenSignsManager | Special events |
| **Punish/Secure** | PunishmentManager, CaptchaManager | Moderation |
| **Pet/Summon** | PetManager, SummonManager | Pet systems |
| **Spawn/AI** | RaidBossSpawnManager, GrandBossManager | Spawning, boss management |

---

## DATA FLOW

### Configuration Data

```
dist/game/config/*.ini
   └─ Loaded by ConfigLoader at startup
      └─ Cached in static fields (GeneralConfig, etc.)
         └─ Read by GameServer components
```

### XML Data Files

```
dist/game/data/items.xml
   └─ Loaded by ItemData at startup
      └─ Parsed into memory objects
         └─ Cached in ItemData singleton
            └─ Accessed by Item system, handlers
```

### Database Data

```
MySQL Database
   ├─ Character loading (on login)
   ├─ Item ownership (inventory)
   ├─ Clan membership
   ├─ Quest progress
   └─ World state persistence
```

---

## KEY ARCHITECTURAL PATTERNS

### 1. Singleton Pattern
The manager directory is not uniformly singleton-based; see [MANAGERS_INDEX.md](INDEXES/MANAGERS_INDEX.md) for the audited partial inventory.

```java
public class ManagerName {
    private static final ManagerName INSTANCE = new ManagerName();
    public static ManagerName getInstance() { return INSTANCE; }
}
```

### 2. Observer Pattern
**EventDispatcher** coordinates event listeners.

```
EventType registered with EventDispatcher
   └─ Listeners attach callbacks
      └─ Events trigger listener execution
```

### 3. Handler Pattern
Command routing for different input types.

```
ClientPacket → GamePacketHandler (dispatcher)
   └─ Routes to specific packet handler
      └─ Handler executes command
         └─ Possibly calls Manager or System
```

### 4. Data Access Pattern
**TableData** classes load and cache XML/SQL data.

```
XML File → Data Loader → Memory Objects → Singleton Cache
   └─ Systems access cache, not files
```

### 5. Task Scheduling Pattern
**ThreadPool** manages all async operations.

```
Manager/Handler → ThreadPool.schedule(task)
   └─ Scheduled on appropriate thread pool
      └─ Executed asynchronously
```

---

## THREAD POOL ARCHITECTURE

**3 Thread Pools** coordinate all concurrent work:

1. **High-Priority Scheduled Pool**
   - Critical server tasks
   - Configured pool size
   - PRIORITY_8 threads

2. **Standard Scheduled Pool**
   - Regular timers and periodic tasks
   - Typical: 4-8 threads
   - Tasks: respawns, events, cooldowns

3. **Instant Execution Pool**
   - Immediate task execution
   - Typical: 50-100 threads
   - Grows dynamically as needed
   - 1-minute thread idle timeout

---

## NETWORK PROTOCOL

**Layer 1: TCP Connection**
- Port 7777 (configurable)
- Persistent per-client connection

**Layer 2: Encryption**
- Blowfish cipher
- NewCrypt protocol variant
- Session-specific key

**Layer 3: Packet Protocol**
- Byte-aligned packet format
- Opcode-based routing
- Request/Response pattern

**Layer 4: Game Protocol**
- 280 client packet source files observed; enum totals require verification
- 389 server packet source files observed; enum totals require verification
- State-based messaging

---

## NEXT STEPS FOR UNDERSTANDING

1. **Game Server Details**: [GAMESERVER_ARCHITECTURE.md](GAMESERVER_ARCHITECTURE.md)
2. **Login Server Details**: [LOGINSERVER_ARCHITECTURE.md](LOGINSERVER_ARCHITECTURE.md)
3. **Network System**: [NETWORK/NETWORK_ARCHITECTURE.md](NETWORK/NETWORK_ARCHITECTURE.md)
4. **Database System**: [DATABASE/DATABASE_ARCHITECTURE.md](DATABASE/DATABASE_ARCHITECTURE.md)
5. **Threading Model**: [THREADING/THREADING_ARCHITECTURE.md](THREADING/THREADING_ARCHITECTURE.md)

---

**Source of Truth**: GameServer.java, source structure analysis  
**Verified**: 2026-08-23  
**Status**: VERIFIED
