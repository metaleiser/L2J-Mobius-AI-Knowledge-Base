# INSTANCE SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: GAMEPLAY — Sistema de instancias  
**Source of Truth**: `entity/instancezone/Instance.java`, `entity/instancezone/InstanceWorld.java`, `managers/InstanceManager.java`  
**Evidence Date**: 2026-08-27 (Sprint 0.7 — Checkpoint 2)  
**Status**: VERIFIED (server-side SOURCE)

---

## 1. CORE ARCHITECTURE

| Componente | Path | Líneas | Función |
|---|---|---|---|
| `Instance` | `entity/instancezone/Instance.java` | ~1000 | Clase principal: jugadores, NPCs, puertas, timers |
| `InstanceWorld` | `entity/instancezone/InstanceWorld.java` | ~420 | DTO con allowed list, helpers NPC, broadcast |
| `InstanceManager` | `managers/InstanceManager.java` | ~440 | Singleton: creación, destrucción, persistencia |
| `InstanceReenterType` | `entity/instancezone/InstanceReenterType.java` | Enum | NONE, ON_ENTER, ON_FINISH |
| `InstanceRemoveBuffType` | `entity/instancezone/InstanceRemoveBuffType.java` | Enum | NONE, ALL, WHITELIST, BLACKLIST |
| `InstanceReenterTimeHolder` | `data/holders/InstanceReenterTimeHolder.java` | Holder | Tiempo de reentrada |

[FACT]

---

## 2. INSTANCE LIFECYCLE

### Creation [FACT]

```
InstanceManager:
  createDynamicInstance(templateId):
    → Incrementa _dynamic (desde 300000+)
    → new Instance(_dynamic)
    → INSTANCES.put(_dynamic, instance)
    → if (templateId > 0):
        → loadInstanceTemplate(templateId)  // XML
        → spawnDoors()
        → spawnGroup("general")
    → return instance

  createInstanceFromTemplate(id, templateId):
    → if instance already exists: return false
    → new Instance(id)
    → loadInstanceTemplate(templateId)
    → spawnDoors()
    → spawnGroup("general")
    → return true
```

**Source**: `managers/InstanceManager.java L377-419`

### Instance In-Game [FACT]

```
Instance state:
  _players (Collection<Integer>) — jugadores en la instancia
  _npcs (Collection<Npc>) — NPCs vivos
  _doors (Map<Integer, Door>) — puertas
  _spawnTemplates — templates de spawn desde XML
  _enterLocations / _exitLocation — puntos de entrada/salida
  _duration — duración en ms
  _emptyDestroyTime — tiempo para destruir si vacío
  _allowRandomWalk — NPC random walk permitido
  _allowSummon — summon permitido
  _ejectTime — tiempo para expulsar jugador muerto
```

### Destruction [FACT]

```
InstanceManager.destroyInstance(instanceid):
  → if instanceid <= 0: return
  → temp.removeNpcs()
## 3. KEY CAPABILITIES

| Capacidad | Método | Comportamiento |
|---|---|---|
| **Duration** | `setDuration(long)` | Tiempo máximo en ms. `TimeUp` destruye al expirar |
| **Empty destroy** | `setEmptyDestroyTime(long)` | Destruye si vacío tras X ms |
| **Player entry** | `addPlayer(Player)` | Añade jugador, check reenter data |
| **Player exit** | `removePlayer(Player)` | Quita jugador, check empty destroy |
| **Spawn doors** | `spawnDoors()` | Spawnea puertas desde template XML |
| **Spawn groups** | `spawnGroup(String)` | Spawnea NPCs desde template |
| **Death handling** | `notifyDeath(Player)` | Programación de eject si no revive |
| **Eject player** | `ejectPlayer(Player)` | Teletransporta a exitLocation o town |
| **Reenter check** | `checkReenterData(Player)` | Verifica cooldown por día/hora |
| **Reenter set** | `setReenterTime(Player)` | Registra en DB `character_instance_time` |
| **Buff removal** | `getRemoveBuffType()` | ALL, WHITELIST o BLACKLIST |



## 4. INSTANCE WORLD (InstanceWorld)

`InstanceWorld` es el DTO que las quests/scripts usan para interactuar con la instancia. [FACT]

| Método | Función |
|---|---|
| `addAllowed(Player)` / `isAllowed(Player)` | Control de acceso |
| `getAliveNpcs(int... ids)` | NPCs vivos por ID |
| `getAliveNpcs(Class<T>, int... ids)` | NPCs vivos filtrados por clase |
| `getNpc(int id)` | Primer NPC con ese ID |
| `spawnGroup(String groupName)` | Spawnea grupo desde template |
| `openDoor(int doorId)` / `closeDoor(int doorId)` | Puertas |
| `broadcastPacket(ServerPacket...)` | Broadcast a jugadores |
| `destroy()` | Marca para destrucción (setEmptyDestroyTime=0, setDuration 1000) |

## 5. REENTRY SYSTEM

`InstanceReenterType`: NONE (sin reentrada), ON_ENTER (al entrar), ON_FINISH (al terminar) [FACT]

`InstanceReenterTimeHolder`: Almacena reentrada como:
- `(DayOfWeek day, int hour, int minute)` — reinicio semanal programado
- `(long time)` — reinicio por timestamp absoluto

**Persistencia**: Tabla SQL `character_instance_time` (charId, instanceId, time) [FACT]

**Source**: `managers/InstanceManager.java L65-67`

## 6. BUFF REMOVAL

`InstanceRemoveBuffType`: NONE (sin remover), ALL (todos), WHITELIST (solo lista), BLACKLIST (todo excepto lista) [FACT]

## 7. DEATH & EJECT

```
Instance.notifyDeath(Player):
  → if ejectTime > 0 && !player.isOnEvent():
      → Send message "IF_YOU_ARE_NOT_RESURRECTED_WITHIN_S1_MINUTES..."
      → Schedule eject task:
          → if player still dead in instance:
              → setInstanceId(0)
              → teleToLocation(exitLocation or TeleportWhereType.TOWN)
```
## 9. INSTANCE TEMPLATES (SOURCE vs RUNTIME)

### Template Inventory [FACT]

| Template | SOURCE | RUNTIME | Estado |
|---|---|---|---|
| CavernOfThePirateCaptainWorldDay60 | ✅ | ✅ | COMMON |
| CavernOfThePirateCaptainWorldDay83 | ✅ | ✅ | COMMON |
| CavernOfThePirateCaptainWorldNight60 | ✅ | ✅ | COMMON |
| Coliseum | ✅ | ✅ | COMMON |
| CrystalCaverns | ✅ | ✅ | COMMON |
| DarkCloudMansion | ✅ | ✅ | COMMON |
| DemonPrince | ✅ | ✅ | COMMON |
| HallOfErosionAttack/Defence | ✅ | ✅ | COMMON |
| HallOfSufferingAttack/Defence | ✅ | ✅ | COMMON |
| HeartInfinityAttack/Defence | ✅ | ✅ | COMMON |
| IceQueensCastle (+ BattleEasy, BattleHardcore) | ✅ | ✅ | COMMON |
| JiniaGuildHideout (1-4) | ✅ | ✅ | COMMON |
| MithrilMine | ✅ | ✅ | COMMON |
| NornilsGarden (+ Quest) | ✅ | ✅ | COMMON |
| Ranku | ✅ | ✅ | COMMON |
| SecretArea | ✅ | ✅ | COMMON |
| SeedOfDestruction | ✅ | ✅ | COMMON |
| SSQ (5 templates) | ✅ | ✅ | COMMON |
| UrbanArea | ✅ | ✅ | COMMON |
| CastleDungeon (dir) | ✅ | ✅ | COMMON |
| ChamberOfDelusion (dir) | ✅ | ✅ | COMMON |
| FortressDungeon (dir) | ✅ | ✅ | COMMON |
| Kamaloka (dir) | ✅ | ✅ | COMMON |
| Olympiad (dir) | ✅ | ✅ | COMMON |
| Pailaka (dir) | ✅ | ✅ | COMMON |
| PailakaRuneCastle (dir) | ✅ | ✅ | COMMON |

### Divergences [FACT]

| Template | Estado | Nota |
|---|---|---|
| **LastImperialTomb** | **SOURCE_ONLY** | Presente en SOURCE; ausente en RUNTIME |
| **FinalEmperialTomb** | **RUNTIME_ONLY** | Presente en RUNTIME; ausente en SOURCE. Probable renombre |
| **RimKamaloka** | **SOURCE_ONLY** | En directorio SOURCE; no en RUNTIME |
| **SeedOfDestruction*MountedTroop** (4) | **SOURCE_ONLY** | 4 variantes en SOURCE; ausentes en RUNTIME |
| **custom/** | **SOURCE_ONLY** | Directorio de templates custom en SOURCE |

> `LastImperialTomb.xml` y `FinalEmperialTomb.xml` son probablemente el mismo contenido con nombre diferente — **no verificado** (UNKNOWN).

## 10. KNOWN UNKNOWNS

- **Si `LastImperialTomb` ≡ `FinalEmperialTomb`**: UNKNOWN — requiere comparación de archivos
- **RimKamaloka funcionalidad**: UNKNOWN
- **SeedOfDestruction MountedTroop**: 4 variantes en SOURCE no verificadas en RUNTIME
- **Level/party requirements exactos**: No verificados en templates XML

## Cross-links

- [PVE_CONTENT_MODEL.md](PVE_CONTENT_MODEL.md) — Instancias por nivel
- [PARTY_PVE.md](PARTY_PVE.md) — Party en instancias
- [ZONE_SYSTEM.md](ZONE_SYSTEM.md) — BossZone
- [../WORLD/WORLD_SYSTEM.md](../WORLD/WORLD_SYSTEM.md) — World e instancias
- [../MANAGERS_INDEX.md](../INDEXES/MANAGERS_INDEX.md) — InstanceManager

---

**Status**: VERIFIED (server-side SOURCE). Divergencias con RUNTIME documentadas como UNKNOWN pendiente de verificación.  
**Evidence Date**: 2026-08-27

**Source**: `entity/instancezone/Instance.java L921-947`

## 8. TIMER & BROADCAST

```
Instance.doCheckTimeUp(int remaining):
  → remaining > 60000: broadcast each 60s
  → remaining > 30000: broadcast each 30s
  → remaining ≤ 30000: broadcast each 10s
  → Cancel old timer
  → Schedule CheckTimeUp or TimeUp (which calls destroyInstance)
```

**Source**: `entity/instancezone/Instance.java L840-898`
  → temp.removePlayers()
  → temp.removeDoors()
  → temp.cancelTimer()
  → INSTANCES.remove(instanceid)
  → _instanceWorlds.remove(instanceid)
```

**Source**: `managers/InstanceManager.java L312-329`