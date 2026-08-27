# RAID BOSS & GRAND BOSS SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: GAMEPLAY — Sistema de raid bosses y grand bosses  
**Source of Truth**: `entity/actor/instance/RaidBoss.java`, `entity/actor/instance/GrandBoss.java`, `managers/RaidBossSpawnManager.java`, `managers/RaidBossPointsManager.java`  
**Evidence Date**: 2026-08-27 (Sprint 0.7 — Checkpoint 2)  
**Status**: VERIFIED (server-side SOURCE)

---

## 1. ARCHITECTURE OVERVIEW

**Aclaración importante**: RaidBoss y GrandBoss son clases **separadas**. GrandBoss NO extiende RaidBoss. Ambos extienden `Monster` directamente. [FACT]

```
Monster
├── RaidBoss    (entity/actor/instance/RaidBoss.java)
└── GrandBoss   (entity/actor/instance/GrandBoss.java)
```

| Componente | Path | Función |
|---|---|---|
| `RaidBoss` | `entity/actor/instance/RaidBoss.java` ~137L | Raid boss con spawn manager |
| `GrandBoss` | `entity/actor/instance/GrandBoss.java` ~122L | Grand boss (epic bosses) |
| `RaidBossSpawnManager` | `managers/RaidBossSpawnManager.java` ~507L | Singleton: spawn, respawn, persistencia |
| `RaidBossPointsManager` | `managers/RaidBossPointsManager.java` | Puntos de raid por boss kill |
| `RaidBossStatus` | `entity/actor/enums/npc/RaidBossStatus.java` | Enum: ALIVE, DEAD, UNDEFINED |
| `BossZone` | `entity/zone/type/BossZone.java` | Zona de boss (raid lock) |

---

## 2. RAIDBOSS vs GRANDBOSS DIFFERENCES [FACT]

| Aspecto | RaidBoss | GrandBoss |
|---|---|---|
| **Extends** | Monster | Monster (NO RaidBoss) |
| **RaidBossSpawnManager** | ✅ Usado (DB-driven) | ❌ No usa |
| **Vitality** | Configurable `RAIDBOSS_USE_VITALITY` | Siempre false |
| **Raid curse** | Configurable `_useRaidCurse` | Configurable `_useRaidCurse` |
| **Death status update** | `RaidBossSpawnManager.updateStatus(true)` | Solo broadcast + points |
| **Respawn** | Via RaidBossSpawnManager (DB) | Propio (no manager central) |
| **InstanceType** | RaidBoss | GrandBoss |
| **Lethalable** | false | false |
| **RandomWalk on spawn** | false | false |
| **isRaid** | true | true |

---

## 3. RAIDBOSS DETAILS

### Class: `RaidBoss` [FACT]

```
Constructor:
  setInstanceType(InstanceType.RaidBoss)
  setIsRaid(true)
  setLethalable(false)

onSpawn():
  setRandomWalking(false)
  super.onSpawn()

doDie(killer):
  → super.doDie(killer)
  → if killer is Playable:
      → broadcast CONGRATULATIONS_YOUR_RAID_WAS_SUCCESSFUL
      → If killer in party:
          → For each member:
              → RaidBossPointsManager.addPoints(member, id, (level/2) + Rnd(-5,5))
              → If noble: Hero.getInstance().setRBkilled(member.getObjectId(), id)
      → If solo: same for player
  → RaidBossSpawnManager.getInstance().updateStatus(this, true)
```

### Raid Curse [FACT]

- `_useRaidCurse` = true (default), configurable vía `setUseRaidCurse(boolean)`
---

## 4. GRANDBOSS DETAILS

### Class: `GrandBoss` [FACT]

```
Constructor:
  setInstanceType(InstanceType.GrandBoss)
  setIsRaid(true)
  setLethalable(false)

doDie(killer):
  → Same as RaidBoss: broadcast, points, Hero tracking
  → NO updateStatus to any spawn manager
  → return true
```

### Differences from RaidBoss [FACT]

- `useVitalityRate()` → **false** (Siempre — GrandBoss nunca usa vitality)
- `getVitalityPoints()` → same formula as RaidBoss (`-super/100`)
- No tiene `_raidStatus` field
- No se registra en `RaidBossSpawnManager`

---

## 5. RAIDBOSS SPAWN MANAGER

### Class: `RaidBossSpawnManager` [FACT]

**Datos desde DB**: Tabla `raidboss_spawnlist`

```sql
boss_id, loc_x, loc_y, loc_z, heading, amount,
respawn_delay, respawn_random, respawn_time,
currentHP, currentMP
```

**Estructura interna**:
```
_bosses:   Map<Integer, RaidBoss>       — bosses vivos
## 6. RAID POINTS & HERO TRACKING

### RaidBossPointsManager [FACT]

- `addPoints(Player, bossId, points)` — añade puntos al matar un raid/grand boss
- Cálculo de puntos: `(bossLevel / 2) + Rnd.get(-5, 5)` [FACT]
- **Source**: `managers/RaidBossPointsManager.java`

### Hero Tracking [FACT]

- `Hero.getInstance().setRBkilled(playerId, bossId)` — registra kill de raid para nobles
- Requiere `player.isNoble()` — si el jugador es noble
- **Source**: `RaidBoss.java L85-87`, `GrandBoss.java L81-83`

---

## 7. BOSS ZONES

`BossZone` (`entity/zone/type/BossZone.java`) — zona específica para raid/grand bosses.  
Archivos en runtime: `data/zones/custom_boss.xml` [FACT]

---

## 8. RAID DROP BROADCAST

Cuando un RaidBoss (no minion) muere, se broadcasta un mensaje con el item drop: [FACT]

```
doItemDrop(...):
  if (_isRaid && !_isRaidMinion && (drop.getCount() > 0)):
      broadcast packet C1_DIED_AND_DROPPED_S3_S2
```

**Source**: `entity/actor/Attackable.java L1194-1200`

---

## 9. KNOWN UNKNOWNS

- **Lista completa de RaidBoss IDs**: No verificada — requiere leer DB raidboss_spawnlist
- **Lista completa de GrandBoss IDs**: No verificada — requiere leer scripts/XML
- **Raid zone mechanics**: BossZone comportamiento exacto no investigado
- **Minion mechanics**: Cómo se generan minions para raids — no investigado
- **RUNTIME raidboss_spawnlist data**: Contenido de la tabla en runtime no verificado

## Cross-links

- [PVE_CONTENT_MODEL.md](PVE_CONTENT_MODEL.md) — Raids por nivel
- [PARTY_PVE.md](PARTY_PVE.md) — Party en raids, CommandChannel
- [PVE_REWARDS_AND_LOOT.md](PVE_REWARDS_AND_LOOT.md) — Drop de raids
- [ZONE_SYSTEM.md](ZONE_SYSTEM.md) — BossZone
- [../COMBAT/DEATH_FLOW.md](../COMBAT/DEATH_FLOW.md) — Raid curse
- [../SYSTEMS/MONSTER_SYSTEM.md](../SYSTEMS/MONSTER_SYSTEM.md) — Monstruos y minions
- [../SOURCE_VS_RUNTIME.md](../SOURCE_VS_RUNTIME.md)

---

**Status**: VERIFIED (server-side SOURCE). RaidBoss ≠ GrandBoss documentado explícitamente.  
**Evidence Date**: 2026-08-27
_spawns:   Map<Integer, Spawn>          — spawn data
_storedInfo: Map<Integer, StatSet>      — HP/MP/respawnTime persistido
_schedules: Map<Integer, ScheduledFuture> — respawn scheduling
```

**Métodos clave**:
- `load()` → Carga desde DB, inicializa bosses y schedules
- `addNewSpawn(Spawn, respawnTime, currentHP, currentMP, isPersistent)`
- `updateStatus(RaidBoss, boolean isDead)` → Actualiza estado, programa respawn
- `reSpawnBoss(RaidBoss)` → Respawnea el boss
- `notifySpawnNightBoss(RaidBoss)` → Boss nocturno especial
- `getAllRaidBossStatus()` → Array con estado de todos los bosses
- `updateDb()` → Persiste HP/MP/respawnTime de todos los bosses
- `cleanUp()` → Limpia schedules y salva estado

**Respawn scheduling**: Via `ThreadPool.schedule()` con `respawn_delay + random` [FACT]
- `giveRaidCurse()` → usado en `Creature.onHitTimer()`: si atacante > nivel+8 del boss, damage=0 y petrifica al atacante
- `setLethalable(false)` — no puede recibir muerte letal

### Vitality [FACT]

- `getVitalityPoints()` → `-super.getVitalityPoints() / 100` (invierte vitality)
- `useVitalityRate()` → `PlayerConfig.RAIDBOSS_USE_VITALITY` (default true)