# ZONE SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: GAMEPLAY — Sistema de zonas y tipos de zona  
**Source of Truth**: `managers/ZoneManager.java`, `entity/zone/ZoneId.java`, `entity/zone/type/*.java`, `data/zones/*.xml`  
**Evidence Date**: 2026-08-27 (Sprint 0.7 — Checkpoint 2)  
**Status**: VERIFIED (server-side SOURCE + RUNTIME)

---

## 1. ZONE ARCHITECTURE

| Componente | Path | Función |
|---|---|---|
| `ZoneManager` | `managers/ZoneManager.java` | Carga y gestión de zonas |
| `ZoneId` | `entity/zone/ZoneId.java` | Enum con 24 tipos de zona |
| `ZoneType` | `entity/zone/ZoneType.java` | Clase base de zona |
| `ZoneForm` | `entity/zone/ZoneForm.java` | Forma geométrica (cuboid, cylinder, n-poly) |
| `ZoneRegion` | `entity/zone/ZoneRegion.java` | Región de zona |
| `ZoneRespawn` | `entity/zone/ZoneRespawn.java` | Punto de respawn |
| `AbstractZoneSettings` | `entity/zone/AbstractZoneSettings.java` | Configuración de zona |

[FACT]

---

## 2. ZONE ID TYPES (24 valores) [FACT]

| ZoneId | Efecto PvE | Clase |
|---|---|---|
| PVP | PvP permitido siempre | — |
| PEACE | No PvP, no debuffs hostiles | `PeaceZone` |
| SIEGE | Zona de asedio | `SiegeZone` |
| TOWN | Zona segura (peace + servicios) | `TownZone` |
| WATER | Efecto de agua, pesca | `WaterZone` |
| SWAMP | Movimiento lento | `SwampZone` |
| DANGER_AREA | Zona peligrosa | — |
| NO_STORE | No se puede abrir tienda | `NoStoreZone` |
| NO_PVP | PvP desactivado | `NoPvPZone` |
| NO_SUMMON_FRIEND | No invocar amigo | `NoSummonFriendZone` |
| NO_BOOKMARK | No usar bookmark | — |
| NO_ITEM_DROP | No dropear items | — |
| NO_RESTART | No reiniciar | `NoRestartZone` |
| JAIL | Zona de castigo | `JailZone` |
| CLAN_HALL | Clan hall | `ClanHallZone` |
| CASTLE | Castillo | `CastleZone` |
| FORT | Fuerte | `FortZone` |
| MOTHER_TREE | Regeneración (elfos) | `MotherTreeZone` |
| LANDING | Aterrizaje permitido | `LandingZone` |
| NO_LANDING | No aterrizar | `NoLandingZone` |
| HQ | Headquarters (asedio) | `HqZone` |
| SCRIPT | Zona scripteable | `ScriptZone` |
| ALTERED | Zona alterada (Seven Signs) | `ConditionZone` |
| MONSTER_TRACK | Rastreo de monstruos | — |

**Source**: `entity/zone/ZoneId.java L23-54`

## 3. ZONE CLASS HIERARCHY

**33 clases de zona** en `entity/zone/type/` [FACT]:

| Clase | Extiende | Propósito |
|---|---|---|
| `ArenaZone` | ZoneType | Zona de arena PvP |
| `BossZone` | ZoneType | Zona de raid boss |
| `CastleZone` | ResidenceZone | Zona de castillo |
| `ClanHallZone` | ResidenceZone | Zona de clan hall |
| `ConditionZone` | ZoneType | Zona condicional (Altered) |
| `DamageZone` | ZoneType | Daño periódico al jugador |
| `DerbyTrackZone` | ZoneType | Pista de carreras |
| `EffectZone` | ZoneType | Efecto personalizado |
| `FishingZone` | ZoneType | Pesca |
| `FlagPvPZone` | ZoneType | PvP con flag |
| `FortZone` | ResidenceZone | Zona de fuerte |
| `HqZone` | ZoneType | Headquarters |
| `JailZone` | ZoneType | Cárcel |
| `LandingZone` | ZoneType | Aterrizaje |
| `MotherTreeZone` | ZoneType | Mother Tree (regeneración) |
| `NoLandingZone` | ZoneType | No aterrizar |
| `NoPvPZone` | ZoneType | No PvP |
| `NoRestartZone` | ZoneType | No reiniciar |
| `NoStoreZone` | ZoneType | No tienda |
| `NoSummonFriendZone` | ZoneType | No summon friend |
| `NpcSpawnTerritory` | ZoneType | Territorio de spawn |
| `OlympiadStadiumZone` | ZoneType | Olympiad |
| `PeaceZone` | ZoneType | Zona pacífica |
| `ResidenceHallTeleportZone` | ResidenceZone | Teleport residencia |
| `ResidenceTeleportZone` | ResidenceZone | Teleport |
| `ResidenceZone` | ZoneType | Base residencia |
| `RespawnZone` | ZoneType | Punto de respawn |
| `ScriptZone` | ZoneType | Zona scripteable |
| `SiegableHallZone` | SiegeZone | Clan hall siegeable |
| `SiegeZone` | ZoneType | Zona de asedio |
| `SwampZone` | ZoneType | Pantano |
| `TownZone` | ZoneType | Ciudad |
| `WaterZone` | ZoneType | Agua |

---

## 4. RUNTIME ZONE XMLS

**47+ archivos** en `game/data/zones/` [FACT]:

- `peace.xml`, `custom_town.xml`, `custom_boss.xml`
- `damage.xml`, `effect.xml`, `swamp.xml`, `water.xml`, `fishing.xml`
- `pvp.xml`, `no_bookmark.xml`, `no_drop_item.xml`, `no_landing.xml`
- `no_restart.xml`, `no_summon_friend.xml`, `respawn.xml`
- `olympiad_stadium.xml`, `krateis_cube.xml`
- `castle_*.xml`, `clanhall_*.xml`, `fortress_*.xml`
- `ssq.xml`, `plains_of_the_lizardmen.xml`

---

## 5. MAP REGIONS (28 archivos)

`game/data/mapregion/` contiene regiones geográficas como:
`talking_island_town.xml`, `elf_town.xml`, `darkelf_town.xml`, `orc_town.xml`, `dwarf_town.xml`, `kamael_town.xml`, `gludio_castle_town.xml`, `gludin_town.xml`, `dion_castle_town.xml`, `giran_castle_town.xml`, `oren_castle_town.xml`, `aden_town.xml`, `godard_town.xml`, `rune_town.xml`, `hunter_town.xml`, `floran_town.xml`, `town_of_schuttgart.xml`, etc.

## Cross-links

- [HUNTING_ZONES.md](HUNTING_ZONES.md) — Zonas de caza
- [PVE_CONTENT_MODEL.md](PVE_CONTENT_MODEL.md) — Mapa de contenido
- [INSTANCE_SYSTEM.md](INSTANCE_SYSTEM.md) — BossZone
- [RAID_BOSS_SYSTEM.md](RAID_BOSS_SYSTEM.md) — Boss zones
- [../WORLD/WORLD_SYSTEM.md](../WORLD/WORLD_SYSTEM.md) — World grid

---

**Status**: VERIFIED (server-side SOURCE + RUNTIME). 24 ZoneId, 33 clases, 47+ XMLs, 28 mapregions documentados.  
**Evidence Date**: 2026-08-27