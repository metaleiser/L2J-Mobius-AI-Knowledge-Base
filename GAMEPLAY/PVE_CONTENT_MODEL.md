# PvE CONTENT MODEL

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: GAMEPLAY — Modelo de contenido PvE, navegación y relaciones  
**Evidence Date**: 2026-08-27 (Sprint 0.7 — Checkpoint 2)  
**Status**: VERIFIED / PARTIAL (síntesis de sistemas)

---

## 1. PURPOSE

Este documento modela el contenido PvE completo del servidor High Five: qué existe, dónde ocurre, cómo se accede, y cómo se relacionan los sistemas. Sirve como mapa de navegación para la documentación GAMEPLAY.

---

## 2. PLAYER PROGRESSION PATH

```
Character Creation (raza/clase)
  → Level 1-20: Zonas iniciales + Quests iniciales (Q00005, Q00001)
  → Level 20-40: Zonas intermedias + Quests de clase
  → Level 40:   Subclass desbloqueable (BASE_SUBCLASS_LEVEL=40)
  → Level 20-50: Instancias Kamaloka / Pailaka
  → Level 40-60: Zonas avanzadas
  → Level 50+:   Raid Bosses (varios)
  → Level 60+:   Instancias avanzadas (Crystal Caverns, Ice Queen, etc.)
  → Level 70+:   Hellbound, Gracia, Seed of Destruction
  → Level 75+:   Grand Bosses
  → Level 85:    Nivel máximo alcanzable
```

> Este path es [INFERENCE] basado en niveles de quests conocidos, rangos de instancias, y distribución de spawns por territorio. No se ha verificado cada nivel de mob individualmente.

---

## 3. HUNTING ZONES (por nivel aproximado)

| Zona/Race Start | Nivel aprox | Región | Spawns |
|---|---|---|---|
| Talking Island | 1-20 | talking_island_town.xml | TalkingIsland/ |
| Elven Village | 10-25 | elf_town.xml | ElvenTerritory/ |
| Dark Elf Village | 10-25 | darkelf_town.xml | DarkElfTerritory/ |
| Orc Village | 5-20 | orc_town.xml | OrcTerritory/ |
| Dwarven Village | 1-10 | dwarf_town.xml | DwarvenTerritory/ |
| Kamael Village | 1-85+ | kamael_town.xml | Kamael/ |
| Gludio | 10-25 | gludio_castle_town.xml | Gludio/ |
| Gludin | 15-30 | gludin_town.xml | Gludin/ |

| Dion | 20-35 | dion_castle_town.xml | Dion/ |
| Giran | 25-40 | giran_castle_town.xml | Giran/ |
| Oren | 30-50 | oren_castle_town.xml | Oren/ |
| Aden | 40-60 | aden_town.xml | Aden/ |
| Goddard | 50-70 | godard_town.xml | Goddard/ |
| Rune | 60-75 | rune_town.xml | Rune/ |
| Innadril (Heine) | 55-70 | florian_town.xml | Innadril/ |
| Hellbound | 70-80 | — | Hellbound/ |
| Gracia | 75-85 | — | Gracia/ |
| Hunters Village | 70-85 | hunter_town.xml | Hunters/ |

> Los niveles por zona son [INFERENCE] basados en ubicación geográfica, quests conocidas, y rangos de instancias. Los niveles exactos de mobs requieren leer NpcTemplate XML de cada NPC.

---

## 4. INSTANCES (por nivel aproximado)

| Instancia | Nivel min | Party | Archivo template |
|---|---|---|---|
| Kamaloka (varios) | 20-50 | Party | `Kamaloka/` |
| Pailaka | 20-60 | Party | `Pailaka/`, `PailakaRuneCastle/` |
| Dark Cloud Mansion | 40+ | Party | `DarkCloudMansion.xml` |
| Cavern of the Pirate Captain | 60+ | Party | `CavernOfThePirateCaptain*.xml` |
| Crystal Caverns | 65+ | Party | `CrystalCaverns.xml` |
| Ice Queen's Castle | 70+ | Party | `IceQueensCastle.xml` (+ Battle) |
| Jinia Guild Hideout | 70+ | Party | `JiniaGuildHideout1-4.xml` |
| Hall of Suffering/Erosion | 75+ | Party | `HallOfSuffering/Erosion*.xml` |
| Heart Infinity | 78+ | Party | `HeartInfinity*.xml` |
| Seed of Destruction | 80+ | Raid Party | `SeedOfDestruction*.xml` |
| Mithril Mine | 80+ | Party | `MithrilMine.xml` |
| Nornil's Garden | 80+ | Party | `NornilsGarden.xml` |
| Demon Prince | 80+ | Party | `DemonPrince.xml` |
| Ranku | 80+ | Party | `Ranku.xml` |

> Los niveles mínimos son [INFERENCE] basados en documentación de Lineage 2. No se verificaron level requirements en los XML templates.

**Detalle completo**: [INSTANCE_SYSTEM.md](INSTANCE_SYSTEM.md)

---

## 5. RAID BOSSES / GRAND BOSSES

| Tipo | Sistema | Spawn | Respawn | Puntos |
|---|---|---|---|---|
| **RaidBoss** | `RaidBoss.java` | DB `raidboss_spawnlist` | Via `RaidBossSpawnManager` | `RaidBossPointsManager` |
| **GrandBoss** | `GrandBoss.java` | Propio (no manager) | Propio | `RaidBossPointsManager` |

**Diferencia clave**: GrandBoss NO extiende RaidBoss. Ambos extienden Monster directamente. [FACT]

## 6. TELEPORT / TRAVEL

⚠️ **CRITICAL DIVERGENCE**: SOURCE y RUNTIME tienen sistemas de teleport arquitectónicamente diferentes.

| Aspecto | SOURCE | RUNTIME |
|---|---|---|
| Sistema | `TeleporterData` (198 XMLs) | Community Board `_bbsteleport` |
| Datos | `data/teleporters/*.xml` | `CommunityBoard.ini` + `teleport.sql` |
| NPCs | 100+ NPC teleporter | Viajes directos desde CB |

**Detalle completo**: [TELEPORT_SYSTEM.md](TELEPORT_SYSTEM.md)

---

## 7. ZONES

El servidor tiene **24 tipos de zona** (`ZoneId` enum) y **33 clases de zona** en `entity/zone/type/`. Efectos PvE clave: [ZONE_SYSTEM.md](ZONE_SYSTEM.md)

---

## 8. NPC SERVICES (PvE-relevantes)

| NPC | Función PvE | Documento relacionado |
|---|---|---|
| Teleporter (NPC) | Transporte entre ciudades | [TELEPORT_SYSTEM.md](TELEPORT_SYSTEM.md) |
| DungeonGatekeeper | Acceso a mazmorras/instancias | [INSTANCE_SYSTEM.md](INSTANCE_SYSTEM.md) |
| Merchant | Compra/venta de items | (no documentado aún) |
| Trainer | Aprendizaje skills | [../SKILLS/SKILL_LEARNING.md](../SKILLS/SKILL_LEARNING.md) |
| Warehouse | Almacenamiento | (no documentado aún) |
| SchemeBuffer | Buff por donation | [../BUFFS/SCHEME_BUFFER_ANALYSIS.md](../BUFFS/SCHEME_BUFFER_ANALYSIS.md) |

---

## 9. CROSS-SYSTEM RELATIONSHIPS

```
LEVEL → DETERMINES → Zone access, instance entry, skill learning, equipment
PARTY → ENABLES → Instance entry, raid participation, efficient hunting
COMBAT → PRODUCES → EXP, SP, drops, spoil
ZONE → AFFECTS → Combat rules, death handling, buff restrictions
TELEPORT → ENABLES → Zone access, instance entry, quest completion
QUEST → GUIDES → Leveling path, zone exploration, equipment progression
INSTANCE → PROVIDES → High-value rewards, boss encounters, party content
RAID BOSS → PROVIDES → Endgame points, hero progression, rare drops
```

## Cross-links

- [LEVELING_AND_PROGRESSION.md](LEVELING_AND_PROGRESSION.md)
- [PARTY_PVE.md](PARTY_PVE.md)
- [INSTANCE_SYSTEM.md](INSTANCE_SYSTEM.md)
- [RAID_BOSS_SYSTEM.md](RAID_BOSS_SYSTEM.md)
- [HUNTING_ZONES.md](HUNTING_ZONES.md)
- [TELEPORT_SYSTEM.md](TELEPORT_SYSTEM.md)
- [ZONE_SYSTEM.md](ZONE_SYSTEM.md)
- [PVE_REWARDS_AND_LOOT.md](PVE_REWARDS_AND_LOOT.md)

---

**Status**: PARTIAL (síntesis basada en investigación — algunos datos son INFERENCE)  
**Evidence Date**: 2026-08-27
**Detalle completo**: [RAID_BOSS_SYSTEM.md](RAID_BOSS_SYSTEM.md)