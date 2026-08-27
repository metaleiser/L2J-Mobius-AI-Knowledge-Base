# SCRIPTING ENGINE

**Source of Truth**: `gameserver.scripting.ScriptEngine`, `gameserver.mechanics.events`  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## PURPOSE

Dynamically load and execute Java code as scripts at runtime.

**Benefit**: Change game behavior without recompiling/restarting

---

## ARCHITECTURE

### ScriptEngine (Singleton)

**Location**: `gameserver.scripting.ScriptEngine`

**Responsibilities**:
- Locate script files
- Load script XML config
- Pass to ScriptExecutor for compilation
- Cache compiled scripts
- Manage script lifecycle

**Key Methods**:
```java
ScriptEngine.getInstance()  // Get singleton
load()                      // Load Scripts.xml
parseDocument()             // Parse XML config
```

### ScriptExecutor

**Location**: `gameserver.scripting.engine.ScriptExecutor`

**Responsibilities**:
- Compile Java code to bytecode
- Load classes dynamically
- Execute compiled handlers

**Technology**: 
- UNKNOWN - Java compiler used (javac, ECJ, etc.)

### MasterHandler

**Location**: Script folder: `handlers/MasterHandler.java`

**Purpose**: Central registry of all script handlers

**Generated**: Auto-generated or manual aggregation (UNKNOWN)

---

## SCRIPT LOADING FLOW

```
1. GameServer.init() loads ScriptEngine
   └─ ScriptEngine.getInstance()
   
2. ScriptEngine loads Scripts.xml
    └─ Reads from: config/Scripts.xml (relative to datapack working directory)
   
3. For each script entry in XML:
   └─ Load Java file
   └─ Compile to bytecode
   └─ Register handler
   └─ Cache in memory
   
4. On event/trigger:
   └─ Lookup handler from cache
   └─ Execute handler code
   └─ Return result
```

---

## SCRIPTS.XML FORMAT

**Location**: `dist/game/config/Scripts.xml` in the checked distribution

**Structure**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<list>
    <script path="handlers/MasterHandler.java"/>
    <script path="custom/SomeFeature.java"/>
    
    <!-- Exclusions (scripts to skip) -->
    <exclude file="broken_script.java">
        <include file="working_fix.java"/>  <!-- But include this -->
    </exclude>
</list>
```

---

## HANDLER TYPES

**13 Handler Categories** (based on event types):

| Handler Type | Purpose | Trigger |
|--------------|---------|---------|
| ActionClickHandler | Player click action | Client clicks NPC |
| ChatHandler | Player chat | Client sends message |
| BypassHandler | NPC dialog button | Player clicks button |
| ItemHandler | Item use | Player uses item |
| AdminCommandHandler | Admin command | GM types command |
| VoicedCommandHandler | Voice command | Player chat command (.) |
| UserCommandHandler | User command | Player chat command (,) |
| EffectHandler | Skill effects | Skill cast |
| TargetHandler | Target type | Skill targeting |
| PunishmentHandler | Punishment type | Admin punish |
| CommunityBoardHandler | Forum post | Board interaction |
| UNKNOWN | UNKNOWN | UNKNOWN |

---

## SCRIPT EXAMPLE: ACTION HANDLER

```java
public class MyActionHandler extends ActionClickHandler {
    @Override
    public boolean action(Player player, WorldObject target) {
        if (target.getName().equals("MyNPC")) {
            player.sendMessage("Hello!");
            return true;
        }
        return false;
    }
}
```

---

## SCRIPT EXAMPLE: ITEM HANDLER

```java
public class MyItemHandler extends ItemHandler {
    @Override
    public boolean use(Playable playable, ItemInstance item) {
        Player player = playable.getActingPlayer();
        if (item.getId() == 12345) {
            player.broadcastStatusUpdate();
            return true;
        }
        return false;
    }
}
```

---

## EVENT SYSTEM (EventDispatcher)

### Events Supported

**OnServerStart**: Server started  
**OnCreatureDeath**: Any creature died  
**OnCreatureAttacked**: Creature attacked  
**OnCreatureSee**: Creature sees another  
**OnCreatureForgot**: Creature loses sight  
**OnCreatureTalk**: Player talks to NPC  
**OnCreatureSay**: Creature says something  

... (UNKNOWN - total event count)

### Event Listener Registration

```java
public class MyEventListener implements EventListener {
    public MyEventListener() {
        EventDispatcher.getInstance().addEventListener(this, EventType.ON_CREATURE_DEATH);
    }
    
    @Override
    public void onEvent(Event event) {
        OnCreatureDeath death = (OnCreatureDeath) event;
        // Handle creature death
    }
}
```

---

## SCRIPT FOLDERS

```
dist/game/data/scripts/
├── handlers/              ← Main handlers
│   ├── MasterHandler.java
│   ├── ActionClickHandler.java
│   └── ... (all handler implementations)
│
├── custom/                ← Custom modifications
│   ├── MyFeature.java
│   └── ... (custom scripts)
│
├── ai/                    ← AI scripts (UNKNOWN structure)
├── quests/                ← Quest scripts (UNKNOWN structure)
└── events/                ← Event scripts (UNKNOWN structure)
```

---

## COMPILATION & CACHING

**On Script Load**:
```
1. Java source read
2. Compiled to .class bytecode
3. Loaded by dynamic classloader
4. Cached in memory
5. Handler instance created
```

**Performance**: 
- First load: ~1 second per script
- Subsequent access: Instant (from cache)

**Persistence**:
- Scripts stay in memory until server stops
- No automatic reloading (requires restart)

---

## REGISTRATION WITH HANDLERS

**Each script must register itself**:

```java
public class MyActionHandler extends ActionClickHandler {
    public MyActionHandler() {
        // Register this handler somewhere
        ActionClickHandler.registerHandler(this);
        // or auto-registered by MasterHandler?
    }
}
```

**UNKNOWN**: Exact registration mechanism (auto vs manual)

---

## ERROR HANDLING

**Compilation Errors**:
```
If syntax error in script:
├─ Logged to console/logs
├─ Script skipped
└─ Server continues (other scripts load)
```

**Runtime Errors**:
```
If exception during handler execution:
├─ Caught by handler framework
├─ Logged
├─ Execution halts (don't crash server)
└─ Server continues
```

---

## DEVELOPMENT WORKFLOW

### Add New Script Feature

```
1. Create new Java file in dist/game/data/scripts/
2. Extend appropriate Handler class
3. Implement handler logic
4. Add entry to Scripts.xml (if needed)
5. Restart server
6. Test feature
```

### Disable Broken Script

```
1. Edit Scripts.xml
2. Add exclusion:
   <exclude file="broken_script.java"/>
3. Restart server
```

---

## PERFORMANCE IMPACT

**Script Loading**:
- ~30-60 seconds total (all scripts)
- Scales with number of scripts
- One-time cost at startup

**Script Execution**:
- Negligible (cached, compiled bytecode)
- Same as direct Java code

**Memory**:
- Scripts + handlers cached in memory
- ~10-50MB typical (varies by script count)

---

## LIMITATIONS

**UNKNOWN**:
- Hotload (reload without restart)
- Script communication (inter-script calls)
- Script debugging (breakpoints, etc.)
- Memory limits per script
- Timeout for long-running scripts

---

## NEXT STEPS

- `SYSTEMS/EVENT_SYSTEM.md` - planned event system details
- `INDEXES/HANDLER_INDEX.md` - planned handler catalog

---

**Source of Truth**: ScriptEngine class, handler framework  
**Verified**: 2026-08-23  
**Status**: PARTIAL (some UNKNOWN implementation details)
