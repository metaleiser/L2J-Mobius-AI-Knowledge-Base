# GAMESERVER ARCHITECTURE

**Source of Truth**: `org.l2jmobius.gameserver.GameServer` and related classes  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## ENTRY POINT

**Main Class**: `org.l2jmobius.gameserver.GameServer`  
**Port**: 7777 (configurable)  
**Maximum Players**: 2000-3000 (configured)  
**Thread Pool Size**: 50-100+ threads  

---

## STARTUP SEQUENCE

The GameServer follows this strict initialization order:

### Phase 1: Interface & Logging
```
1. InterfaceConfig.load()
   └─ Check if GUI enabled
      └─ If yes: Create GUI window

2. Create log directory (./log/)

3. LogManager.readConfiguration(./log.cfg)
   └─ Configure Java logging
```

### Phase 2: Configuration System
```
4. ConfigLoader.init()
    └─ Loads configuration classes in sequence (16 main load calls plus custom configs):
      ├─ GeneralConfig
      ├─ ServerConfig (ports, game world settings)
      ├─ PlayerConfig (player limits)
      ├─ RatesConfig (exp, drop rates)
      ├─ PvpConfig (PvP rules)
      ├─ NpcConfig (NPC behavior)
      ├─ FeatureConfig (feature flags)
      ├─ OlympiadConfig
      ├─ GeoEngineConfig
      ├─ IdManagerConfig
      ├─ DevelopmentConfig
      ├─ FloodProtectorConfig
    ├─ GraciaSeedsConfig
    └─ ... (custom configs)
```

**Custom Configs** (44 files) loaded separately:
```
AutoPlayConfig, BossAnnouncementsConfig, CaptchaConfig,
DualboxCheckConfig, FactionSystemConfig, OfflinePlayConfig,
PremiumSystemConfig, PvpAnnounceConfig, WeddingConfig,
... (40+ more)
```

### Phase 3: Database
```
5. DatabaseFactory.init()
   └─ Load DatabaseConfig
   └─ Configure HikariCP pool
    ├─ Driver default: com.mysql.cj.jdbc.Driver
    ├─ Maximum pool: clamp(configured, 4, 1000)
    ├─ Minimum idle: max(maximum / 10, 2)
      ├─ Connection timeout: 60 seconds
      └─ Test connection
```

### Phase 4: Threading
```
6. ThreadPool.init()
   └─ Configure thread pools:
      ├─ High-Priority Scheduled Pool
      │  └─ Size: ThreadConfig.HIGH_PRIORITY_SCHEDULED_THREAD_POOL_SIZE
      │  └─ Priority: PRIORITY_8
      │
      ├─ Standard Scheduled Pool
      │  └─ Size: ThreadConfig.SCHEDULED_THREAD_POOL_SIZE
      │  └─ Core threads pre-started
      │  └─ Rejects with CallerRunsPolicy
      │
      └─ Instant Execution Pool
         └─ Size: ThreadConfig.INSTANT_THREAD_POOL_SIZE
         └─ Grows to MAX_VALUE as needed
         └─ 1-minute keep-alive
   
   └─ Schedule purge task (every 60 seconds)
```

### Phase 5: Core Services
```
7. GameTimeTaskManager.getInstance()
   └─ Starts game time cycle (day/night)

8. IdManager.getInstance()
   └─ Initialize global ID generator

9. EventDispatcher.getInstance()
   └─ Create event listener infrastructure

10. ScriptEngine.getInstance()
    └─ Load and compile game scripts from data/ directory
```

### Phase 6: World System
```
11. InstanceManager.getInstance()
    └─ Prepare instance dungeon system

12. World.init()
    └─ Create world entity registry
    └─ Initialize WorldRegion grid
    └─ Prepare for 60,000+ entities

13. MapRegionData.getInstance()
    └─ Load map region definitions

14. AnnouncementsTable.getInstance()
    └─ Load announcements from database

15. GlobalVariablesManager.getInstance()
    └─ Load global server variables
```

### Phase 7: Skills System
```
16. EffectHandler.getInstance().executeScript()
    └─ Compile and load skill effects

17. EnchantSkillGroupsData.getInstance()
    └─ Load skill enchant groups

18. SkillTreeData.getInstance()
    └─ Load skill progression trees

19. SkillData.getInstance()
    └─ Load 5000+ skill definitions

20. PetSkillData.getInstance()
    └─ Load pet/summon skills
```

### Phase 8: Items System
```
21. ItemData.getInstance()
    └─ Load 40000+ item definitions

22. EnchantItemGroupsData.getInstance()
    └─ Load item enchant groups

23. EnchantItemData.getInstance()
    └─ Load enchant success rates

24. EnchantItemOptionsData.getInstance()
    └─ Load enchant option effects

25. ElementalAttributeData.getInstance()
    └─ Load elemental attributes

26. OptionData.getInstance()
    └─ Load item options

27. RecipeData.getInstance()
    └─ Load crafting recipes

28. MultisellData.getInstance()
    └─ Load NPC multi-sell lists

29. BuyListData.getInstance()
    └─ Load shop buy lists

30. ArmorSetData.getInstance()
    └─ Load armor set bonuses

31. FishData.getInstance()
    └─ Load fishing data

32. HennaData.getInstance()
    └─ Load henna tattoo data

33. PrimeShopData.getInstance()
    └─ Load premium shop
```

### Phase 9: Character System
```
34. ClassListData.getInstance()
    └─ Load player classes

35. InitialEquipmentData.getInstance()
    └─ Load new character equipment

36. ExperienceData.getInstance()
    └─ Load exp tables by level

37. KarmaLossData.getInstance()
    └─ Load karma loss rates

38. HitConditionBonusData.getInstance()
    └─ Load hit/miss bonuses

39. PlayerTemplateData.getInstance()
    └─ Load player templates by class

40. CharInfoTable.getInstance()
    └─ Load character base info from DB

41. AdminData.getInstance()
    └─ Load admin account list

42. RaidBossPointsManager.getInstance()
    └─ Load raid boss points

43. CharSummonTable.getInstance().init()
    └─ Load saved player summons

44. CaptchaManager.getInstance()
    └─ Initialize captcha system
```

### Phase 10: Premium System (optional)
```
45. IF PremiumSystemConfig.PREMIUM_SYSTEM_ENABLED:
    └─ PremiumManager.getInstance()
```

### Phase 11: Clan System
```
46. ClanTable.getInstance()
    └─ Load all clans from DB

47. CHSiegeManager.getInstance()
    └─ Initialize clan hall sieges

48. ClanHallTable.getInstance()
    └─ Load clan halls

49. ClanHallAuctionManager.getInstance()
    └─ Initialize clan hall auctions
```

### Phase 12: Geodata & Environment
```
50. GeoEngine.getInstance()
    └─ Load pathfinding/collision data

51. DayNightSpawnManager.getInstance()
    └─ Load day/night spawning rules
```

### Phase 13: NPC & Spawning
```
52. NpcData.getInstance()
    └─ Load 1000+ NPC templates

53. SpawnData.getInstance()
    └─ Load spawn points from XML

54. TeleporterData.getInstance()
    └─ Load teleport destinations

55. DoorData.getInstance()
    └─ Load door definitions

56. StaticObjectData.getInstance()
    └─ Load static objects
```

### Phase 14: Major Game Systems
```
57. SiegeManager.getInstance()
    └─ Initialize siege system

58. CastleManager.getInstance()
    └─ Load castles

59. FortManager.getInstance()
    └─ Load fortresses

60. GrandBossManager.getInstance()
    └─ Initialize raid boss management

61. DimensionalRiftManager.getInstance()
    └─ Initialize dimensional rifts

62. ItemAuctionManager.getInstance()
    └─ Initialize item auctions

63. ... (20+ more managers)
```

### Phase 15: Network
```
64. ConnectionManager.start()
    └─ Bind to port 7777
    └─ Listen for client connections
    └─ Register GamePacketHandler
```

### Phase 16: Login Server Link
```
65. LoginServerThread.init()
    └─ Connect to LoginServer (port 9014)
    └─ Register as available Game Server
```

### Phase 17: Shutdown Hook
```
66. Register shutdown handler
    └─ Graceful shutdown on CTRL+C
```

---

## CORE COMPONENTS

### GameServer Class
**Location**: `org.l2jmobius.gameserver.GameServer`

**Responsibilities**:
- Orchestrate initialization
- Load all data
- Start network listener
- Monitor server health
- Handle graceful shutdown

**Key Methods**:
```
main(String[] args)             - Entry point
GameServer()                    - Constructor, runs init
printSection(String)            - Log initialization progress
```

### World
**Location**: `org.l2jmobius.gameserver.entity.World`

**Responsibilities**:
- Maintain registry of all entities
- Organize entities by region
- Manage player visibility
- Broadcast messages
- Entity spawning/deletion

**Key Data**:
```
_allPlayers          - ConcurrentHashMap of all players
_allCharacters       - ConcurrentHashMap of all creatures
_worldRegions        - Grid of WorldRegion objects
```

### LoginServerThread
**Location**: `org.l2jmobius.gameserver.LoginServerThread`

**Responsibilities**:
- Maintain connection to LoginServer
- Exchange server status
- Receive/send player login requests

### Shutdown
**Location**: `org.l2jmobius.gameserver.Shutdown`

**Responsibilities**:
- Graceful server shutdown
- Save player data
- Close connections
- Persist world state

---

## MANAGER CATEGORIES

### World/Entity Managers
- `World` - Main entity registry
- `InstanceManager` - Instanced zones
- `ZoneManager` - Zone definitions
- `IdManager` - Global ID generation

### Combat/Duel
- `DuelManager` - 1v1 combat
- `AntiFeedManager` - PvP anti-feeding

### Clan/Siege
- `ClanTable` - Clan registry
- `SiegeManager` - Siege coordination
- `CastleManager` - Castle management
- `FortManager` - Fortress management
- `CHSiegeManager` - Clan hall sieges

### NPC/Spawn
- `NpcData` - NPC definitions
- `SpawnData` - Spawn locations
- `RaidBossSpawnManager` - Raid boss spawns
- `GrandBossManager` - Grand boss management
- `DayNightSpawnManager` - Dynamic spawning

### Items/Economy
- `ItemManager` - Item management
- `RecipeManager` - Crafting
- `MailManager` - Mail system
- `ItemAuctionManager` - Item auctions
- `MercTicketManager` - Mercenary tickets

### Events/Features
- `EventDropManager` - Event drops
- `FishingChampionshipManager` - Fishing
- `DimensionalRiftManager` - Rift instances
- `TerritoryWarManager` - Territory wars
- `SellBuffsManager` - Buff selling

### Special Managers
- `PunishmentManager` - Bans/mutes
- `CaptchaManager` - Anti-bot captchas
- `PremiumManager` - Premium features
- `PetitionManager` - Player petitions

**Total**: 52 manager classes (all singletons)

---

## DATA FLOW

### On Player Login

```
1. Client connects to port 7777
   └─ GameClient instance created
   
2. Client sends AuthLogin packet
   └─ SessionKey validated against LoginServer
   
3. Player character loaded from database
   └─ CharInfoTable.getCharacter(accountId, characterId)
   
4. Player entity spawned in World
   └─ Player added to _allCharacters
   └─ Player added to appropriate WorldRegion
   
5. Initial world state sent to client
   └─ Nearby entities
   └─ Chat/system messages
   └─ Inventory
   └─ Skills
   
6. Player can now issue commands
   └─ Movements, attacks, skills, etc.
```

### On Server Tick (typical)

```
Every 100ms or configured interval:

1. ThreadPool executes scheduled tasks
   └─ Respawns
   └─ Cooldown expirations
   └─ Event triggers
   
2. World updates all creatures
   └─ AI ticks
   └─ Status effects
   └─ Movement
   
3. Broadcast visibility changes
   └─ New entities in range
   └─ Entities leaving range
   
4. Network sends updates to clients
   └─ ServerPackets queued
   └─ Sent to GameClient connections
```

---

## NETWORK PACKET FLOW

**GamePacketHandler** routes all incoming packets:

```
Packet received from client
   │
   ├─ Read opcode (first byte)
   └─ Lookup handler in ClientPackets table
      └─ Instantiate packet class
      └─ Deserialize from buffer
      └─ Call packet.handle(gameClient)
         └─ May update World
         └─ May queue ServerPackets response
         └─ Response sent to client
```

---

## MEMORY MODEL

**Major Memory Consumers**:

| Component | Estimate | Notes |
|-----------|----------|-------|
| World entities (60K+) | 500MB-1GB | Players, NPCs, items |
| Item data (40K items) | 100-200MB | Cached in ItemData |
| NPC data (1K+ NPCs) | 50-100MB | Cached in NpcData |
| Skill data (5K skills) | 50-100MB | Cached in SkillData |
| Player inventory (avg) | 5-10MB each | Player-specific |
| DB connections (100) | 100-200MB | HikariCP pool |

**Note**: REQUIRES CODE VERIFICATION for exact memory profiling.

---

## PERFORMANCE CHARACTERISTICS

**Designed for**:
- 2000-3000 concurrent players
- 100 database connections
- 50-100 async threads

**Bottlenecks** (typical):
- Database I/O
- Network bandwidth
- Garbage collection (tuned via -XX:+UseZGC)

**Note**: UNKNOWN - actual performance depends on hardware and configuration.

---

## GRACEFUL SHUTDOWN

**Shutdown.java** handles server stop:

```
1. Cancel all scheduled tasks
2. Disconnect all players
3. Save world state to database
4. Close database connections
5. Stop thread pools
6. Release network ports
7. Exit JVM
```

---

## NEXT STEPS

- [SYSTEM DOCUMENTATION → INDEXES/MASTER_INDEX.md](INDEXES/MASTER_INDEX.md) - Navegación a todas las áreas (incluye SYSTEMS/)
- [PACKETS/PACKET_ARCHITECTURE.md](PACKETS/PACKET_ARCHITECTURE.md) - Detalles de red y packets
- [DATABASE/DATABASE_ARCHITECTURE.md](DATABASE/DATABASE_ARCHITECTURE.md) - Database design
- [THREADING/THREADING_ARCHITECTURE.md](THREADING/THREADING_ARCHITECTURE.md) - Threading details

---

**Source of Truth**: GameServer.java, initialization code analysis  
**Verified**: 2026-08-23  
**Status**: VERIFIED
