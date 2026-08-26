# GAPS MAP — Cobertura de la AI_KNOWLEDGE_BASE

**Última actualización**: 2026-08-26 (KB v2.2 / SKILL SEMANTIC FOUNDATION 2.5)

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
| SKILLS | COVERED* | SKILLS/ (12 docs): ARCHITECTURE, DATA_MODEL, TARGETING, CONDITIONS, LOADING, LEARNING, CAST_FLOW, EFFECT_SYSTEM, MAGIC_DAMAGE, HANDLERS_SCRIPTS + SKILL_SEMANTIC_REFERENCE + ABNORMAL_TYPE_REFERENCE | Semántica cross-AbnormalType y augmentation → SOURCE_REQUIRED | `skills/**`, XML | ALTA (2.5) |
| BUFFS / EFFECTS | COVERED* | SKILL_SEMANTIC_REFERENCE, EFFECT_SYSTEM extendido, ABNORMAL_TYPE_REFERENCE | Comportamiento por skill XML específica → SOURCE_REQUIRED | `EffectList.java`, `AbstractEffect` | ALTA (2.5) |
| TARGETS | COVERED | SKILL_TARGETING extendido con AffectScope/AffectObject | Semántica específica de cada handler script → SOURCE_REQUIRED | `TargetHandler`, scripts targets | — |
| SCHEME VALIDATION | PARTIAL | SKILL_SEMANTIC_REFERENCE evidence matrix | Reglas ejecutables del validator; validación por skill XML → SOURCE_REQUIRED | `mechanics/skill/**`, XML | ALTA (2.6) |
| ITEMS / EQUIPMENT | PARTIAL | SYSTEMS/ITEM_SYSTEM.md (59 líneas), INVENTORY_SYSTEM.md | enchant/attributes/equip/sets/crystalización | `items/**`, Item/Inventory | MEDIA |
| DROPS / LOOT | PARTIAL | DEATH_FLOW menciona `doItemDrop`; SPAWN_QUERY_GUIDE no cubre tablas | Tablas normales vs quest vs evento; rates | `DropListDAO`/XML drops, Attackable | ALTA |
| LEVELING | MISSING | — | XP tables, rates, penalty party-level | `PcStat`, config rates | MEDIA |
| PARTY GENERAL | PARTIAL | Solo crédito quest ([QUEST_PARTY_CREDIT.md](QUESTS/QUEST_PARTY_CREDIT.md)) | Party general: formación, share EXP/drop, comandos | `Party.java`, `PartyMemberPosition` | MEDIA |
| INSTANCES | MISSING (planificado ⧗) | mencionadas en WORLD_SYSTEM §9 | InstanceManager, InstanceWorld, reenter | `entity/instancezone/`, managers | MEDIA |
| RAID / BOSS | MISSING | Monster System lista RaidBoss/GrandBoss como clases; sin doc de mecánica | respawn timers, curse, minions, raid parties | `RaidBoss*`, `GrandBoss*`, DB | ALTA |
| ZONES | MISSING (planificado ⧗) | — | ZoneType, peace/combat/water/etc. | `zones/*.xml`, ZoneManager | MEDIA |
| TELEPORT | MISSING | — | Teleporters NPC, Community Board teleports, scrolls, locId | `TeleportLocation*`, SQL, CB | **ALTA (CB)** |
| COMMUNITY BOARD | MISSING | — | Prioridad post-2.4 (ROADMAP 2.7) | `CommunityBoard*`, HTML/bypass | **ALTA (2.7)** |
| SCHEME BUFFER | MISSING | — | Prioridad post-2.4 (ROADMAP 2.6); requiere Skill Semantic Foundation (2.5) | scripts buffers, skills | **ALTA (2.6)** |
| CLAN | MISSING (planificado ⧗) | — | Clan, skills de clan, reputación, wars | `clan/**` | BAJA (ahora) |
| SIEGE | MISSING (planificado ⧗) | — | castles, artifacts, mercs | `siege/**` | BAJA (ahora) |
| CRAFTING | MISSING | — | recipes, dwarven craft, manufacture | `recipe/**`, RequestRecipe | MEDIA |
| ECONOMY | MISSING | — | shops, multsell, taxes, warehouse, trade | `multisell/**`, merchants | MEDIA |
| OLYMPIAD | MISSING | — | matches, points, heroes | `olympiad/**` | BAJA |
| PVP / PK / KARMA | PARTIAL | mencionado en configs (karma) | cálculo karma, PK protect, drop por karma | `Player`, Config | MEDIA |
| PETS | MISSING | SUMMON_SYSTEM cubre summons genéricos | pet evolution, feeding, inventory | `pets/**` | BAJA |
| FISHING | MISSING | — | baits, zones de pesca, minigame | `fishing/**` | BAJA |
| SPAWNS (consulta práctica) | **COVERED** | [WORLD/SPAWN_QUERY_GUIDE.md](WORLD/SPAWN_QUERY_GUIDE.md) | doc formal del sistema (`SYSTEMS/SPAWN_SYSTEM.md`) sigue ⧗ | `spawns/**`, SpawnData | — |

---

## Nota sobre documentación histórica

No existe un directorio `FASE1_OBSOLETO/`. Los hallazgos y correcciones de Fase 1 están registrados en [VERSIONING/AUDIT_HISTORY.md](VERSIONING/AUDIT_HISTORY.md) (AUDIT-001, H1–H18) y en [CHANGELOG.md](VERSIONING/CHANGELOG.md). Los documentos actuales de Fase 2+ son la referencia vigente; el historial **NO debe usarse como Source of Truth** cuando exista una corrección posterior.

## Regla de uso

Antes de responder una pregunta sobre un dominio marcado MISSING/PARTIAL:
1. Consultar el doc existente (si hay).
2. Verificar la afirmación concreta en SOURCE.
3. Marcar el resultado como VERIFIED solo con evidencia `ruta:línea`.
4. Si se documenta conocimiento nuevo reusable → actualizar este mapa.
