# GAPS MAP — Cobertura de la AI_KNOWLEDGE_BASE

**Última actualización**: 2026-08-27 (Sprint 0.7 — PvE Gameplay Knowledge Expansion)

> **Nota `*`**: dominio COVERED con base semántica documentada en Micro-Sprint 2.5. Sub-temas cross-AbnormalType y augmentation siguen como SOURCE_REQUIRED.
**Propósito**: mapa central de cobertura documental. Protege contra alucinaciones: si un dominio figura como MISSING, la KB **no lo documenta** y cualquier afirmación sobre él debe verificarse directamente contra SOURCE.

> ⚠️ **"MISSING" significa "La KB todavía no documenta este dominio."**
> **NO significa "el sistema no existe".**

Estados permitidos:
- **COVERED** — documentado con evidencia.
- **PARTIAL** — documentado parcialmente; faltan sub-temas.
- **MISSING** — sin documento propio; consultar SOURCE directamente.
- **SOURCE_REQUIRED** — existe orientación, pero toda afirmación concreta exige verificación en SOURCE.

---

## Matriz de cobertura

| Dominio | Estado | Existe hoy en la KB | Falta / siguiente paso | Fuente de investigación | Prioridad futura |
|---|---|---|---|---|---|
| QUESTING (motor) | **COVERED** | QUESTS/ (14 docs): ARCHITECTURE, LIFECYCLE, EVENTS, STATES_VARIABLES, REWARDS, PARTY_CREDIT, TIMERS, PLAYER_NPC_DIALOG, ENGINE_REFERENCE, RESEARCH_FRAMEWORK, VERTICAL_SLICE_TEMPLATE + slices Q00003/Q00005 + análisis Q00039/Q00005 | Mantener al día; promover patrones nuevos a ENGINE_REFERENCE/REWARDS | `mechanics/script/*`, datapack quests | — |
| COMBAT | PARTIAL | COMBAT/ (7 docs): architecture, attack flow, damage, defense, criticals, death flow, tasks | Drop/reward normal del mob (cruce con DEATH_FLOW); efectos por elemento | `Attackable`, `Creature`, formulas | MEDIA |
| SKILLS | COVERED* | SKILLS/ (14 docs): ARCHITECTURE, DATA_MODEL, TARGETING, CONDITIONS, LOADING, LEARNING, CAST_FLOW, EFFECT_SYSTEM, MAGIC_DAMAGE, HANDLERS_SCRIPTS + SKILL_SEMANTIC_REFERENCE + REFERENCE/ABNORMAL_TYPE_CATALOG.md + SCHEME_VALIDATION.md (consolidated) | Semántica cross-AbnormalType y augmentation → SOURCE_REQUIRED | `skills/**`, XML | ALTA (2.5) |
| BUFFS / EFFECTS | COVERED* | SKILL_SEMANTIC_REFERENCE, EFFECT_SYSTEM extendido, REFERENCE/ABNORMAL_TYPE_CATALOG.md | Comportamiento por skill XML específica → SOURCE_REQUIRED | `EffectList.java`, `AbstractEffect` | ALTA (2.5) |
| TARGETS | COVERED | SKILL_TARGETING extendido con AffectScope/AffectObject | Semántica específica de cada handler script → SOURCE_REQUIRED | `TargetHandler`, scripts targets | — |
| SCHEME VALIDATION | **COVERED** | `SKILL_SEMANTIC_REFERENCE` evidence matrix, `SKILLS/SCHEME_VALIDATION.md` (consolidated from design + lifecycle) | Cross-AbnormalType y augmentation siguen como SOURCE_REQUIRED | `mechanics/skill/**`, XML | **2.7 (implementación)** |
| ITEMS / EQUIPMENT | PARTIAL | SYSTEMS/ITEM_SYSTEM.md (59 líneas), INVENTORY_SYSTEM.md | enchant/attributes/equip/sets/crystalización | `items/**`, Item/Inventory | MEDIA |
| DROPS / LOOT | **COVERED** | [GAMEPLAY/PVE_REWARDS_AND_LOOT.md](GAMEPLAY/PVE_REWARDS_AND_LOOT.md): doItemDrop flow, spoil, auto-loot configs, DropTypes (DROP/SPOIL/CHAMPION/EVENT), rates. [COMBAT/DEATH_FLOW.md](../COMBAT/DEATH_FLOW.md): muerte y rewards. | DropData XML format, spoil skill exacto | `Attackable`, `NpcTemplate`, `RatesConfig` | — |
| LEVELING | **COVERED** | [GAMEPLAY/LEVELING_AND_PROGRESSION.md](GAMEPLAY/LEVELING_AND_PROGRESSION.md): XP table, death loss, rates, subclass, vitality, dynamic rates, config | Vitality exact values, dynamic rates XML | `ExperienceData`, `ExperienceLossData`, `PlayerStat`, `DynamicExpRateData` | — |
| PARTY GENERAL | **COVERED** | [GAMEPLAY/PARTY_PVE.md](GAMEPLAY/PARTY_PVE.md): party architecture, 5 loot modes, 5 EXP modes, CommandChannel, spoil integration, damaging party logic, quest credit relationship | EXP distribution formula exacta, loot mechanics detail | `Party.java`, `PartyDistributionType`, `PartyExpType`, `CommandChannel` | — |
| INSTANCES | **COVERED** | [GAMEPLAY/INSTANCE_SYSTEM.md](GAMEPLAY/INSTANCE_SYSTEM.md): lifecycle, InstanceManager, InstanceWorld, reentry, timers, doors, death/eject, buff removal. [GAMEPLAY/PVE_CONTENT_MODEL.md](GAMEPLAY/PVE_CONTENT_MODEL.md): instancias por nivel. | SOURCE/RUNTIME template divergence (7 SOURCE_ONLY, 1 RUNTIME_ONLY). LastImperialTomb vs FinalEmperialTomb. RimKamaloka. | `entity/instancezone/`, `managers/InstanceManager.java` | — |
| RAID / BOSS | **COVERED** | [GAMEPLAY/RAID_BOSS_SYSTEM.md](GAMEPLAY/RAID_BOSS_SYSTEM.md): RaidBoss vs GrandBoss differences, RaidBossSpawnManager, raid curse, points, hero tracking, boss zones. [GAMEPLAY/PVE_CONTENT_MODEL.md](GAMEPLAY/PVE_CONTENT_MODEL.md): raids por nivel. | Lista completa de RaidBoss IDs, GrandBoss IDs, minion mechanics | `RaidBoss*`, `GrandBoss*`, DB | — |
| ZONES | **COVERED** | [GAMEPLAY/ZONE_SYSTEM.md](GAMEPLAY/ZONE_SYSTEM.md): 24 ZoneId types, 33 zone classes, 47+ runtime XMLs, 28 mapregions, gameplay effects. [GAMEPLAY/HUNTING_ZONES.md](GAMEPLAY/HUNTING_ZONES.md): hunting territories. | Niveles exactos de mobs por zona (INFERENCE actualmente) | `zones/*.xml`, `ZoneManager`, mapregion XMLs | — |
| TELEPORT | **COVERED** | [GAMEPLAY/TELEPORT_SYSTEM.md](GAMEPLAY/TELEPORT_SYSTEM.md): ⚠️ CRITICAL DIVERGENCE documentada. SOURCE (198 XMLs, NPC-based) vs RUNTIME (CB _bbsteleport, SQL). TeleporterData, TeleportHolder, categorías, types, fees, noble teleport, CB teleport flow, config. | Cómo carga RUNTIME teleport data (SQL o XML?). Si los NPC Gatekeeper siguen funcionales sin XML. | `TeleporterData`, `teleporters/` (SOURCE), `teleport.sql`, `CommunityBoard.ini`, `HomeBoard.java` | **ALTA (CB)** |
| COMMUNITY BOARD | PARTIAL | `BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md` (subsistema scheme; análisis **SERVER_RUNTIME**, divergencia SOURCE/RUNTIME PATH_DIVERGENCE documentada), `BUFFS/SCHEME_SYSTEM_COMPARISON.md`. Arquitectura verificada (AUDIT-007): MasterHandler registration · routing `RequestBypassToServer` · dispatch `CommunityBoardHandler` · entrega HTML `HtmlUtil.sendCBHtml` / `CommunityBoardHandler.separateAndSend` (runtime L633-640) | UI general y otros servicios (teleport, multisell) siguen MISSING/planificados; paridad de interfaz con `IParseBoardHandler` del baseline SOURCE: PARTIAL; attribution de la customización runtime: PARTIAL | `CommunityBoard*`, HTML/bypass | **ALTA (2.7)** |
| SCHEME BUFFER | **COVERED** | `BUFFS/SCHEME_BUFFER_ANALYSIS.md`, `BUFFS/SCHEME_SYSTEM_COMPARISON.md` | Mantener al día si cambia el NPC | `SchemeBuffer.java`, `SchemeBufferTable.java` | — |
| CLAN | MISSING (planificado ⧗) | — | Clan, skills de clan, reputación, wars | `clan/**` | BAJA (ahora) |
| SIEGE | MISSING (planificado ⧗) | — | castles, artifacts, mercs | `siege/**` | BAJA (ahora) |
| CRAFTING | MISSING | — | recipes, dwarven craft, manufacture | `recipe/**`, RequestRecipe | MEDIA |
| ECONOMY | MISSING | — | shops, multsell, taxes, warehouse, trade | `multisell/**`, merchants | MEDIA |
| OLYMPIAD | MISSING | — | matches, points, heroes | `olympiad/**` | BAJA |
| PVP / PK / KARMA | PARTIAL | mencionado en configs (karma) | cálculo karma, PK protect, drop por karma | `Player`, Config | MEDIA |
| PETS | MISSING | SUMMON_SYSTEM cubre summons genéricos | pet evolution, feeding, inventory | `pets/**` | BAJA |
| FISHING | MISSING | — | baits, zones de pesca, minigame | `fishing/**` | BAJA |
| GAMEPLAY (PvE knowledge) | **COVERED** | [GAMEPLAY/](GAMEPLAY/): LEVELING_AND_PROGRESSION, PARTY_PVE, PVE_CONTENT_MODEL, INSTANCE_SYSTEM, RAID_BOSS_SYSTEM, HUNTING_ZONES, TELEPORT_SYSTEM, PVE_REWARDS_AND_LOOT, ZONE_SYSTEM, GAMEPLAY_RELATION_GRAPH (10 documentos) | Niveles exactos de mobs por zona, lista completa de RaidBoss IDs, spoil skill exacto, dynamic rates XML | Síntesis de sistemas | — |\n| SPAWNS (consulta práctica) | **COVERED** | [WORLD/SPAWN_QUERY_GUIDE.md](WORLD/SPAWN_QUERY_GUIDE.md) | doc formal del sistema (`SYSTEMS/SPAWN_SYSTEM.md`) sigue ⧗ | `spawns/**`, SpawnData | — |

---

## Nota sobre documentación histórica

No existe un directorio `FASE1_OBSOLETO/`. Los hallazgos y correcciones de Fase 1 están registrados en [VERSIONING/AUDIT_HISTORY.md](VERSIONING/AUDIT_HISTORY.md) (AUDIT-001, H1–H18) y en [CHANGELOG.md](VERSIONING/CHANGELOG.md). Los documentos actuales de Fase 2+ son la referencia vigente; el historial **NO debe usarse como Source of Truth** cuando exista una corrección posterior.

## Regla de uso

Antes de responder una pregunta sobre un dominio marcado MISSING/PARTIAL:
1. Consultar el doc existente (si hay).
2. Verificar la afirmación concreta en SOURCE.
3. Marcar el resultado como VERIFIED solo con evidencia `ruta:línea`.
4. Si se documenta conocimiento nuevo reusable → actualizar este mapa.
