# PROJECT STRUCTURE

**Source of Truth**: Raíz del proyecto del servidor en este workspace (subcarpeta `L2J_Mobius_CT_2.6_HighFive`)  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## PHYSICAL LAYOUT

```
L2J_Mobius_CT_2.6_HighFive/
│
├── build.xml                  ← Apache Ant build configuration
├── readme.txt                 ← Project notes (version history)
│
├── java/
│   └── org/l2jmobius/         ← All source code
│
├── dist/                      ← Runtime distribution
│   ├── game/                  ← Game Server runtime folder
│   │   ├── config/            ← Configuration files (loaded at runtime)
│   │   ├── data/              ← XML data files (items, NPCs, skills, etc.)
│   │   ├── log/               ← Logs (generado en runtime; NO existe en el snapshot fuente)
│   │   ├── GameServer.sh      ← Linux launcher
│   │   ├── GameServer.vbs     ← Windows launcher
│   │   └── java.cfg           ← Java runtime config
│   │
│   ├── login/                 ← Login Server runtime folder
│   │   ├── config/            ← Configuration files
│   │   ├── data/              ← Data files
│   │   ├── LoginServer.sh     ← Linux launcher
│   │   └── LoginServer.vbs    ← Windows launcher
│   │
│   ├── db_installer/          ← Database installation tool
│   ├── backup/                ← Database backups
│   ├── libs/                  ← Required Java libraries (JARs)
│   └── images/                ← Server images/resources
│
├── launcher/                  ← IDE launch configurations
│   ├── Gameserver.launch      ← Eclipse/IDE launcher (GameServer)
│   ├── Loginserver.launch     ← Eclipse/IDE launcher (LoginServer)
│   └── Search.launch          ← Eclipse/IDE launcher (Search tool)
│
└── .project, .classpath       ← Eclipse project files
```

---

## SOURCE CODE ORGANIZATION

```
java/org/l2jmobius/
│
├── commons/                   ← Shared infrastructure
│   ├── config/                ← Configuration loading
│   ├── crypt/                 ← Encryption (Blowfish)
│   ├── database/              ← Database (HikariCP)
│   ├── network/               ← TCP networking
│   │   ├── buffer/            ← Byte buffer management
│   │   ├── handler/           ← Packet handlers
│   │   ├── packet/            ← Packet structures
│   │   └── pool/              ← Connection pooling
│   ├── threads/               ← Thread pool management
│   ├── time/                  ← Time utilities
│   ├── ui/                    ← Optional GUI components
│   └── util/                  ← General utilities
│
├── gameserver/                ← Game Server (MAIN)
│   ├── GameServer.java        ← Entry point
│   ├── LoginServerThread.java ← Connection to login server
│   ├── Shutdown.java          ← Graceful shutdown
│   │
│   ├── ai/                    ← AI system (13 types)
│   │   ├── AbstractAI.java
│   │   ├── CreatureAI.java
│   │   ├── PlayableAI.java
│   │   ├── PlayerAI.java
│   │   ├── AttackableAI.java
│   │   └── ... (8 more AI types)
│   │
│   ├── cache/                 ← Caching systems
│   │   ├── HtmCache.java      ← NPC dialog caching
│   │   └── RelationCache.java ← Relationship cache
│   │
│   ├── communitybbs/          ← Community board system
│   │
│   ├── config/                ← Runtime configuration
│   │   ├── ConfigLoader.java  ← Central config loading
│   │   ├── GeneralConfig.java ← General settings
│   │   ├── ServerConfig.java  ← Server settings
│   │   ├── PlayerConfig.java  ← Player limits
│   │   ├── RatesConfig.java   ← Drop/exp rates
│   │   ├── PvpConfig.java     ← PvP rules
│   │   ├── NpcConfig.java     ← NPC behavior
│   │   └── custom/            ← 44 custom feature configs
│   │
│   ├── data/                  ← Data loading
│   │   ├── sql/               ← SQL table loaders (10 tables)
│   │   ├── xml/               ← XML data loaders (40+ files)
│   │   ├── enums/             ← Enum definitions
│   │   ├── holders/           ← Data holder classes
│   │   ├── AugmentationData.java
│   │   ├── BotReportTable.java
│   │   └── SpawnTable.java
│   │
│   ├── entity/                ← Game world entities
│   │   ├── World.java         ← World master
│   │   ├── WorldObject.java   ← Base entity class
│   │   ├── WorldRegion.java   ← Geographic region
│   │   ├── Location.java      ← Coordinate system
│   │   ├── actor/             ← Living entities
│   │   │   ├── Creature.java  ← Base creature class
│   │   │   ├── Player.java    ← Player character
│   │   │   ├── Npc.java       ← Non-player character
│   │   │   ├── Summon.java    ← Summons/pets
│   │   │   ├── instance/      ← 40+ NPC subtypes
│   │   │   ├── stat/          ← Stat systems
│   │   │   └── status/        ← Buff/debuff systems
│   │   ├── clan/              ← Clan system
│   │   ├── residences/        ← Castle/hall system
│   │   ├── itemcontainer/     ← Inventory types
│   │   ├── item/              ← Item system
│   │   ├── zone/              ← Zone system
│   │   ├── spawns/            ← Spawn system
│   │   ├── groups/            ← Party/group system
│   │   └── cubic/             ← Cubic system
│   │
│   ├── geoengine/             ← Pathfinding/geo
│   │   ├── GeoEngine.java
│   │   ├── geodata/           ← Geo binary files
│   │   ├── pathfinding/       ← A* algorithm
│   │   └── util/              ← Geo utilities
│   │
│   ├── handler/               ← Event handlers
│   │   ├── ActionClickHandler.java
│   │   ├── AdminCommandHandler.java
│   │   ├── BypassHandler.java
│   │   ├── ChatHandler.java
│   │   ├── ItemHandler.java
│   │   └── ... (8 more handler types)
│   │
│   ├── managers/              ← 58 Java classes (singleton status varies)
│   │   ├── AirShipManager.java
│   │   ├── CastleManager.java
│   │   ├── ClanHallAuctionManager.java
   │   ├── DuelManager.java
   │   ├── InstanceManager.java
   │   ├── IdManager.java
   │   └── ... (46 more managers)
   │
   ├── mechanics/             ← Game mechanics
   │   ├── announce/          ← Announcements
   │   ├── buylist/           ← Shop lists
   │   ├── effects/           ← Skill effects
   │   ├── events/            ← Event system
   │   ├── fishing/           ← Fishing system
   │   ├── olympiad/          ← Olympiad competition
   │   ├── punishement/       ← Punishment system
   │   ├── siege/             ← Siege mechanics
   │   ├── skill/             ← Skill mechanics
   │   ├── sevensigns/        ← Seven Signs event
   │   └── ... (more subsystems)
   │
   ├── network/               ← Game network
   │   ├── GameClient.java    ← Client connection
   │   ├── GamePacketHandler.java ← Packet dispatcher
│   ├── clientpackets/     ← 280 source files observed
│   ├── serverpackets/     ← 389 source files observed
   │   ├── Encryption.java    ← Session encryption
   │   └── enums/             ← Packet enums
   │
   ├── scripting/             ← Script engine
   │   ├── ScriptEngine.java  ← Main engine
   │   ├── engine/            ← Compilation & loading
   │   └── annotations/       ← Script annotations
   │
   ├── taskmanagers/          ← Scheduled tasks
   │   ├── GameTimeTaskManager.java
   │   ├── ItemLifeTimeTaskManager.java
   │   └── ... (more task managers)
   │
   ├── security/              ← Security systems
   │
   ├── interfaces/            ← Interface definitions
   │
   ├── ui/                    ← Optional GUI
   │
   └── util/                  ← Utilities
│
├── loginserver/               ← Login Server
│   ├── LoginServer.java       ← Entry point
│   ├── GameServerListener.java← Listens for Game Servers
│   ├── GameServerThread.java  ← Per-server connection
│   │
│   ├── config/                ← Login config
│   ├── controller/            ← LoginController
│   ├── data/                  ← GameServerTable (servers list)
│   ├── enums/                 ← Login enums
│   ├── network/               ← Login packets
│   ├── ui/                    ← Optional GUI
│   └── util/                  ← Utilities
│
├── tools/                     ← Administrative tools
│   ├── AccountManager.java
│   ├── DatabaseInstaller.java
│   ├── GameServerRegister.java
│   └── Search.java
│
└── log/                       ← Logging system
    ├── ServerLogManager.java
    ├── filter/
    ├── formatter/
    └── handler/
```

---

## KEY DIRECTORIES EXPLAINED

### `commons/`
Shared infrastructure used by both Login and Game servers.
- Database connection management
- Network protocols
- Thread scheduling
- Encryption/decryption
- Configuration loading

### `gameserver/`
The Game Server - contains all game logic, entities, and systems.
- Most complex component
- 20+ interconnected systems
- 58 clases en managers/ (52 con patrón getInstance(), 6 sin él)
- 669 network packet classes (280 client + 389 server)
- Handles all game world simulation

### `loginserver/`
Authentication server for client login and server registration.
- Minimal compared to Game Server
- Manages accounts and sessions
- Coordinates Game Server list
- Issues auth tokens

### `tools/`
Utilities run separately from servers.
- NOT part of server runtime
- Admin utilities only
- Database tools

### `dist/`
Runtime distribution (not source code).
- Configuration files (game/login)
- Data files (XML)
- Launcher scripts
- JAR libraries

---

## CONFIGURATION DIRECTORIES

**Game Server**: `dist/game/config/`  
**Login Server**: `dist/login/config/`  

Configuration files are loaded at runtime, not embedded in JAR.

---

## DATA DIRECTORIES

**Game Server**: `dist/game/data/`  
**Login Server**: `dist/login/data/`  

Contains XML configuration files for items, NPCs, skills, etc.

---

## TOTAL COMPONENTS

| Component | Count | Notes |
|-----------|-------|-------|
| Managers | 52 | All singletons |
| Custom Configs | 44 | Feature-specific |
| SQL Tables | 10 | Core data persistence |
| Handler Types | 13 | Command handlers, chat, etc. |
| AI Types | 13 | Different AI behaviors |
| NPC Subtypes | 40+ | Merchant, trainer, teleporter, etc. |
| Client Packets | 400+ | Commands from client |
| Server Packets | 350+ | Updates to client |

---

**Source of Truth**: Verified against physical file structure  
**Verified**: 2026-08-23  
**Status**: VERIFIED
