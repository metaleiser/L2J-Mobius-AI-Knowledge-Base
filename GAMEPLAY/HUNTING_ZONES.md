# HUNTING ZONES

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: GAMEPLAY — Zonas de caza y territorios  
**Source of Truth**: `data/SpawnTable.java`, `data/spawns/` (SOURCE + RUNTIME), `data/mapregion/*.xml`  
**Evidence Date**: 2026-08-27 (Sprint 0.7 — Checkpoint 2)  
**Status**: PARTIAL (arquitectura VERIFIED; rangos de nivel INFERENCE)

---

## 1. SPAWN ARCHITECTURE

| Componente | Path | Función |
|---|---|---|
| `SpawnTable` | `data/SpawnTable.java` | Mapa `_npcSpawns` (npcId → Set<Spawn>) |
| `Spawn` | `entity/spawns/Spawn.java` | Datos de spawn: locación, respawn, template |
| `SpawnGroup` | `entity/spawns/SpawnGroup.java` | Grupo de spawns |
| `RespawnTaskManager` | `taskmanagers/RespawnTaskManager.java` | Reprogramación de respawn |
| `DecayTaskManager` | `taskmanagers/DecayTaskManager.java` | Decaimiento de cuerpos |

[FACT]

---

## 2. DATA SOURCES

### SOURCE: `dist/game/data/spawns/`
Archivos organizados por carpeta de territorio. [FACT]

### RUNTIME: `game/data/spawns/`
Territorios en carpetas. **Diferencia estructural**: SOURCE tiene archivos multi-XML, RUNTIME tiene `HighFiveSpawns.xml` consolidado (además de carpetas).

### Map Regions: `game/data/mapregion/`
28 archivos XML definiendo regiones geográficas del mundo. [FACT]

---

## 3. TERRITORIES

| Carpeta de spawns | Región de mapa (mapregion) | NPCs típicos |
|---|---|---|
| TalkingIsland/ | talking_island_town.xml | Roxxy, Guard Leon, Captain Gilbert |
| ElvenTerritory/ | elf_town.xml | Mirabel, Sentinel Gartrandell |
| DarkElfTerritory/ | darkelf_town.xml | Jasmine, Sentry Knight Rayla |
| OrcTerritory/ | orc_town.xml | Tamil, Praetorian Rukain |
| DwarvenTerritory/ | dwarf_town.xml | Wirphy, Protector Paion |
| Gludio/ | gludio_castle_town.xml | Guardias de Gludio |
| Gludin/ | gludin_town.xml | Guardias de Gludin |
| Dion/ | dion_castle_town.xml | Guardias de Dion |
| Giran/ | giran_castle_town.xml | Guardias de Giran |
| Oren/ | oren_castle_town.xml | Guardias de Oren |
| Aden/ | aden_town.xml | Guardias de Aden |
| Goddard/ | godard_town.xml | Guardias de Goddard |
| Rune/ | rune_town.xml | Guardias de Rune |
| Innadril/ | floran_town.xml | Guardias de Heine |
| Hellbound/ | — | NPCs de Hellbound |
| Gracia/ | — | NPCs de Gracia (Seed of Destruction) |
| Hunters/ | hunter_town.xml | Cazadores, NPCs de hunting ground |
| Kamael/ | kamael_town.xml | Ragara, Zerstorer Marcela |
| Castles/ | castle zones | NPCs de castillos |
| Others/ | miscs.xml | Varios |

[FACT] — las carpetas y archivos existen en SOURCE y RUNTIME.

---

## 4. LEVEL RANGES (INFERENCE)

> ⚠️ Los siguientes rangos son [INFERENCE] basados en:
> - Niveles de quests conocidas (Q00005: lvl2, Q00003: lvl16, Q00039: lvl20)
> - Rangos de instancias (Kamaloka: lvl20-50, Pailaka: lvl20-60, etc.)
> - Ubicación geográfica relativa
>
> **No se ha verificado el nivel exacto de cada mob en los XML de NpcTemplate.**

| Zona | Nivel estimado |
|---|---|
| Talking Island | 1-20 |
| Elven Forest/Elven Fortress | 10-25 |
| Dark Elf Forest | 10-25 |
| Orc Village area | 5-20 |
| Dwarven area | 1-10 |
| Kamael area | 1-85+ |
| Gludio/Gludin | 10-30 |
| Dion area | 20-35 |
| Giran area | 25-40 |
| Oren area | 30-50 |
| Aden area | 40-60 |
| Goddard area | 50-70 |
| Rune area | 60-75 |
| Innadril area | 55-70 |
| Hellbound | 70-80 |
| Gracia (Seed of Destruction) | 75-85 |
| Hunters Village area | 70-85 |

---

## 5. HUNTING ZONE GAMEPLAY

Cada zona de caza típicamente contiene:
- **Mobs de varios niveles** dentro del rango de la zona
- **Ocasionalmente**: un raid boss o mini-boss
- **NPCs de servicio**: Gatekeeper, Merchant, Warehouse
- **Puntos de teleport** vía NPC Gatekeeper o Community Board

---

## 6. KNOWN UNKNOWNS

- **Niveles exactos de mobs por zona**: UNKNOWN — requiere leer NpcTemplate XMLs
- **Monsters por zona**: Catálogo completo de IDs/mobs por zona no existe
- **Drops asociados por zona**: No investigado
- **Quests asociadas por zona**: Solo las 3 quests documentadas tienen mapeo zona↔quest
- **RUNTIME HighFiveSpawns.xml**: No se leyó su contenido completo

## Cross-links

- [PVE_CONTENT_MODEL.md](PVE_CONTENT_MODEL.md) — Mapa de contenido por nivel
- [LEVELING_AND_PROGRESSION.md](LEVELING_AND_PROGRESSION.md) — Progresión por nivel
- [TELEPORT_SYSTEM.md](TELEPORT_SYSTEM.md) — Acceso a zonas
- [ZONE_SYSTEM.md](ZONE_SYSTEM.md) — Tipos de zona
- [../WORLD/SPAWN_QUERY_GUIDE.md](../WORLD/SPAWN_QUERY_GUIDE.md) — Consulta de spawns

---

**Status**: PARTIAL (arquitectura VERIFIED; rangos de nivel INFERENCE)  
**Evidence Date**: 2026-08-27