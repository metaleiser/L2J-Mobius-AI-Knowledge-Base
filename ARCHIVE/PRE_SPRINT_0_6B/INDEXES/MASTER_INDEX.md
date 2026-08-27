# MASTER INDEX

**Purpose**: Navigation guide for all AI queries about L2J Mobius CT 2.6 HighFive.

**How to Use**:
1. Identify your question topic below
2. Follow the arrow to the relevant document
3. Read the document for detailed information
4. Use internal links to navigate further

> ✅ = documento existente ([Fase 2A](../FASE1_SUMMARY.md)) · ⧗ = planificado (no creado aún)

---

## SYSTEM & MAINTENANCE

| Question | Document |
|----------|----------|
| Guía de entrada para IAs | [AI_INSTRUCTIONS/AI_README.md](../AI_INSTRUCTIONS/AI_README.md) |
| Reglas de verificación | [AI_INSTRUCTIONS/VERIFICATION_RULES.md](../AI_INSTRUCTIONS/VERIFICATION_RULES.md) |
| Protocolo de auditoría | [AI_INSTRUCTIONS/AUDIT_PROTOCOL.md](../AI_INSTRUCTIONS/AUDIT_PROTOCOL.md) |
| Protocolo de actualización | [AI_INSTRUCTIONS/UPDATE_PROTOCOL.md](../AI_INSTRUCTIONS/UPDATE_PROTOCOL.md) |
| Detección de cambios upstream | [AI_INSTRUCTIONS/CHANGE_DETECTION.md](../AI_INSTRUCTIONS/CHANGE_DETECTION.md) |
| Versión de la KB | [VERSIONING/KB_VERSION.md](../VERSIONING/KB_VERSION.md) |
| Baseline upstream | [VERSIONING/UPSTREAM_BASELINE.md](../VERSIONING/UPSTREAM_BASELINE.md) |
| Changelog | [VERSIONING/CHANGELOG.md](../VERSIONING/CHANGELOG.md) |
| Historial de auditorías | [VERSIONING/AUDIT_HISTORY.md](../VERSIONING/AUDIT_HISTORY.md) |

---

## PROJECT & WORKSPACE (KB v2.0)

| Pregunta / tema | Documento |
|-----------------|-----------|
| ¿Qué es el proyecto y sus 4 entidades? | [00_PROJECT/PROJECT_CONTEXT.md](../00_PROJECT/PROJECT_CONTEXT.md) ✅ |
| Fuentes de referencia (repos, wiki, cliente) | [00_PROJECT/REFERENCE_SOURCES.md](../00_PROJECT/REFERENCE_SOURCES.md) ✅ |
| Decisiones de proyecto | [00_PROJECT/DECISIONS.md](../00_PROJECT/DECISIONS.md) ✅ |
| Hoja de ruta | [00_PROJECT/ROADMAP.md](../00_PROJECT/ROADMAP.md) ✅ |
| Ideas futuras (Quest Assistant, bots, crónicas) | [00_PROJECT/IDEAS.md](../00_PROJECT/IDEAS.md) ✅ |

### Entidades del workspace

| Tema | Documento |
|------|-----------|
| SERVER_SOURCE (UPSTREAM, code Git) | [00_PROJECT/PROJECT_CONTEXT.md](../00_PROJECT/PROJECT_CONTEXT.md) + [VERSIONING/UPSTREAM_BASELINE.md](../VERSIONING/UPSTREAM_BASELINE.md) |
| SERVER_RUNTIME (desplegado/ejecutable) | [00_PROJECT/PROJECT_CONTEXT.md](../00_PROJECT/PROJECT_CONTEXT.md) + [VERSIONING/UPSTREAM_BASELINE.md](../VERSIONING/UPSTREAM_BASELINE.md) |
| CLIENT (cliente H5) | [00_PROJECT/PROJECT_CONTEXT.md](../00_PROJECT/PROJECT_CONTEXT.md) (investigación futura) |
| KNOWLEDGE_BASE (esta KB) | [README.md](../README.md) |
| Build (Apache Ant) y deployment | [BUILD_AND_DEPLOYMENT.md](../BUILD_AND_DEPLOYMENT.md) ✅ |

### Investigación del cliente (CLIENT_RESEARCH)

| Tema | Documento |
|------|-----------|
| Estructura del cliente H5 (carpetas/formatos) | [CLIENT_RESEARCH/CLIENT_STRUCTURE.md](../CLIENT_RESEARCH/CLIENT_STRUCTURE.md) ✅ |
| Cifrado de archivos de texto del cliente | [CLIENT_RESEARCH/CLIENT_ENCRYPTION.md](../CLIENT_RESEARCH/CLIENT_ENCRYPTION.md) ✅ |
| Caso piloto Q00001 Letters of Love | [CLIENT_RESEARCH/QUEST_PILOT_Q00001.md](../CLIENT_RESEARCH/QUEST_PILOT_Q00001.md) ✅ |
| Mapa autoridad CLIENT ↔ SERVER por entidad | [CLIENT_RESEARCH/CLIENT_SERVER_MAPPING.md](../CLIENT_RESEARCH/CLIENT_SERVER_MAPPING.md) ✅ |

### Investigación futura / experimentos

| Tema | Documento |
|------|-----------|
| Investigación del cliente (QUESTS/LORE/HTML/NPC/ITEMS/...) | [00_PROJECT/ROADMAP.md](../00_PROJECT/ROADMAP.md) ⧗ (en curso) |
| Quest Knowledge multi-perspectiva | [00_PROJECT/ROADMAP.md](../00_PROJECT/ROADMAP.md) ⧗ (futuro) |
| Experimentos / observaciones del SERVER_RUNTIME | KB: marcar OBSERVED/VERIFIED (ver [AI_INSTRUCTIONS/VERIFICATION_RULES.md](../AI_INSTRUCTIONS/VERIFICATION_RULES.md)) |

---

## CORE PROJECT DOCUMENTATION

| Question | Document |
|----------|----------|
| What is this project? | [README.md](../README.md) ✅ |
| Project overview | [PROJECT_OVERVIEW.md](../PROJECT_OVERVIEW.md) ✅ |
| Project directory structure | [PROJECT_STRUCTURE.md](../PROJECT_STRUCTURE.md) ✅ |
| How do components interact? | [ARCHITECTURE_OVERVIEW.md](../ARCHITECTURE_OVERVIEW.md) ✅ |
| How do I build and deploy? | [BUILD_AND_DEPLOYMENT.md](../BUILD_AND_DEPLOYMENT.md) ✅ |
| Phase 1 summary | [FASE1_SUMMARY.md](../FASE1_SUMMARY.md) ✅ |

---

## SERVER ARCHITECTURE

| Question | Document |
|----------|----------|
| How does Game Server start up? | [GAMESERVER_ARCHITECTURE.md](../GAMESERVER_ARCHITECTURE.md) ✅ |
| What is the initialization order? | [GAMESERVER_ARCHITECTURE.md](../GAMESERVER_ARCHITECTURE.md#startup-sequence) ✅ |
| How does Login Server work? | [LOGINSERVER_ARCHITECTURE.md](../LOGINSERVER_ARCHITECTURE.md) ✅ |
| How do client logins work? | [LOGINSERVER_ARCHITECTURE.md](../LOGINSERVER_ARCHITECTURE.md#authentication-flow) ✅ |
| What does Commons library provide? | [COMMONS_ARCHITECTURE.md](../COMMONS_ARCHITECTURE.md) ✅ |

---

## GAME SYSTEMS (Fase 2A: entidades y mundo)

| Question | Document |
|----------|----------|
| How are entities structured? | [SYSTEMS/ENTITY_SYSTEM.md](../SYSTEMS/ENTITY_SYSTEM.md) ✅ |
| How does the world work? | [SYSTEMS/WORLD_SYSTEM.md](../SYSTEMS/WORLD_SYSTEM.md) ✅ |
| How do players work? | [SYSTEMS/PLAYER_SYSTEM.md](../SYSTEMS/PLAYER_SYSTEM.md) ✅ |
| How do NPCs work? | [SYSTEMS/NPC_SYSTEM.md](../SYSTEMS/NPC_SYSTEM.md) ✅ |
| How do monsters work? | [SYSTEMS/MONSTER_SYSTEM.md](../SYSTEMS/MONSTER_SYSTEM.md) ✅ |
| How do summons work? | [SYSTEMS/SUMMON_SYSTEM.md](../SYSTEMS/SUMMON_SYSTEM.md) ✅ |
| How do items work? | [SYSTEMS/ITEM_SYSTEM.md](../SYSTEMS/ITEM_SYSTEM.md) ✅ |
| How does inventory work? | [SYSTEMS/INVENTORY_SYSTEM.md](../SYSTEMS/INVENTORY_SYSTEM.md) ✅ |

---

## WORLD / SPAWNS

| Tema | Documento |
|------|-----------|
| Localizar/contar spawns de un NPC/mob (layout SOURCE por celdas vs RUNTIME consolidado) | [WORLD/SPAWN_QUERY_GUIDE.md](../WORLD/SPAWN_QUERY_GUIDE.md) ✅ |

---

## AI & COMBAT (Fase 2B)

| Tema | Documento |
|------|-----------|
| Arquitectura de IA | [AI/AI_ARCHITECTURE.md](../AI/AI_ARCHITECTURE.md) ✅ |
| Intención / Acción / NextAction | [AI/INTENTION_ACTION.md](../AI/INTENTION_ACTION.md) ✅ |
| Sistema de target | [AI/TARGET_SYSTEM.md](../AI/TARGET_SYSTEM.md) ✅ |
| Aggro / amenaza | [AI/AGGRO_SYSTEM.md](../AI/AGGRO_SYSTEM.md) ✅ |
| Tareas de IA | [AI/AI_TASKS.md](../AI/AI_TASKS.md) ✅ |
| Arquitectura de combate | [COMBAT/COMBAT_ARCHITECTURE.md](../COMBAT/COMBAT_ARCHITECTURE.md) ✅ |
| Flujo de ataque | [COMBAT/ATTACK_FLOW.md](../COMBAT/ATTACK_FLOW.md) ✅ |
| Cálculo de daño | [COMBAT/DAMAGE_CALCULATION.md](../COMBAT/DAMAGE_CALCULATION.md) ✅ |
| Defensa / mitigación | [COMBAT/DEFENSE.md](../COMBAT/DEFENSE.md) ✅ |
| Críticos | [COMBAT/CRITICALS.md](../COMBAT/CRITICALS.md) ✅ |
| Flujo de muerte | [COMBAT/DEATH_FLOW.md](../COMBAT/DEATH_FLOW.md) ✅ |
| Tareas de combate | [COMBAT/COMBAT_TASKS.md](../COMBAT/COMBAT_TASKS.md) ✅ |

---

## SKILLS (Fase 2C)

| Tema | Documento |
|------|-----------|
| Arquitectura (3 capas: core/scripts/xml) | [SKILLS/SKILL_ARCHITECTURE.md](../SKILLS/SKILL_ARCHITECTURE.md) ✅ |
| Modelo de datos (XML ↔ Skill) | [SKILLS/SKILL_DATA_MODEL.md](../SKILLS/SKILL_DATA_MODEL.md) ✅ |
| Referencia semántica (Scheme Validator) | [SKILLS/SKILL_SEMANTIC_REFERENCE.md](../SKILLS/SKILL_SEMANTIC_REFERENCE.md) ✅ |
| Catálogo AbnormalType | [SKILLS/ABNORMAL_TYPE_REFERENCE.md](../SKILLS/ABNORMAL_TYPE_REFERENCE.md) ✅ |
| Carga y reload | [SKILLS/SKILL_LOADING.md](../SKILLS/SKILL_LOADING.md) ✅ |
| Aprendizaje / adquisición | [SKILLS/SKILL_LEARNING.md](../SKILLS/SKILL_LEARNING.md) ✅ |
| Flujo de casteo | [SKILLS/CAST_FLOW.md](../SKILLS/CAST_FLOW.md) ✅ |
| Targeting de skills | [SKILLS/SKILL_TARGETING.md](../SKILLS/SKILL_TARGETING.md) ✅ |
| Condiciones | [SKILLS/SKILL_CONDITIONS.md](../SKILLS/SKILL_CONDITIONS.md) ✅ |
| Sistema de efectos | [SKILLS/EFFECT_SYSTEM.md](../SKILLS/EFFECT_SYSTEM.md) ✅ |
| Daño mágico / resistencias | [SKILLS/MAGIC_DAMAGE.md](../SKILLS/MAGIC_DAMAGE.md) ✅ |
| Handlers como scripts | [SKILLS/SKILL_HANDLERS_SCRIPTS.md](../SKILLS/SKILL_HANDLERS_SCRIPTS.md) ✅ |
| Diseño del Scheme Validator | [SKILLS/SCHEME_VALIDATOR_DESIGN.md](../SKILLS/SCHEME_VALIDATOR_DESIGN.md) ✅ |
| Ciclo de vida de validación de schemes | [SKILLS/SCHEME_VALIDATION_LIFECYCLE.md](../SKILLS/SCHEME_VALIDATION_LIFECYCLE.md) ✅ |

---

## BUFFS (Micro-Sprint 2.6)

| Tema | Documento |
|------|-----------|
| SchemeBuffer NPC: persistencia DB, handlers, storage CSV `"id,level"`, slots reales | [BUFFS/SCHEME_BUFFER_ANALYSIS.md](../BUFFS/SCHEME_BUFFER_ANALYSIS.md) ✅ |
| Community Board schemes: bypass/HTML, add/remove buff, cast flow | [BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md](../BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md) ✅ |
| Comparación NPC buffer ↔ Community Board (veredictos y gaps) | [BUFFS/SCHEME_SYSTEM_COMPARISON.md](../BUFFS/SCHEME_SYSTEM_COMPARISON.md) ✅ |

---

## QUESTS (Fase 2D)

| Tema | Documento |
|------|-----------|
| Arquitectura (Quest/Script/Event, core vs datapack, ejemplo real) | [QUESTS/QUEST_ARCHITECTURE.md](../QUESTS/QUEST_ARCHITECTURE.md) ✅ |
| Ciclo de vida / carga / registro / reload | [QUESTS/QUEST_LIFECYCLE.md](../QUESTS/QUEST_LIFECYCLE.md) ✅ |
| Catálogo de eventos y callbacks (~40) + despacho | [QUESTS/QUEST_EVENTS.md](../QUESTS/QUEST_EVENTS.md) ✅ |
| Estados, cond, variables, memo, persistencia | [QUESTS/QUEST_STATES_VARIABLES.md](../QUESTS/QUEST_STATES_VARIABLES.md) ✅ |
| Timers de quest | [QUESTS/QUEST_TIMERS.md](../QUESTS/QUEST_TIMERS.md) ✅ |
| Recompensas (items/exp/sp/adena/sounds) | [QUESTS/QUEST_REWARDS.md](../QUESTS/QUEST_REWARDS.md) ✅ |
| Interacción Player↔NPC↔diálogo HTML | [QUESTS/QUEST_PLAYER_NPC_DIALOG.md](../QUESTS/QUEST_PLAYER_NPC_DIALOG.md) ✅ |
| Análisis transversal Q00039 RedEyedInvaders (SOURCE↔RUNTIME↔CLIENT↔GAMEPLAY) | [QUESTS/Q00039_REDEYEDINVADERS_ANALYSIS.md](../QUESTS/Q00039_REDEYEDINVADERS_ANALYSIS.md) ✅ |
| Análisis transversal Q00005 Miner's Favor (SOURCE↔RUNTIME, Newbie Guide conflict documented) | [QUESTS/Q00005_MINERSFAVOR_ANALYSIS.md](../QUESTS/Q00005_MINERSFAVOR_ANALYSIS.md) ✅ |
| Metodología de investigación (framework, plantilla A-W, taxon. clues/lore) | [QUESTS/QUEST_RESEARCH_FRAMEWORK.md](../QUESTS/QUEST_RESEARCH_FRAMEWORK.md) ✅ |
| Crédito de party en quests (getRandomPartyMember* / PARTY_CREDIT_*) | [QUESTS/QUEST_PARTY_CREDIT.md](../QUESTS/QUEST_PARTY_CREDIT.md) ✅ |
| Plantilla reutilizable de vertical slice (24 secciones + checklist) | [QUESTS/QUEST_VERTICAL_SLICE_TEMPLATE.md](../QUESTS/QUEST_VERTICAL_SLICE_TEMPLATE.md) ✅ |
| **Referencia central del engine de quests** (Quest#/QuestState# APIs, 15 categorías) | [QUESTS/QUEST_ENGINE_REFERENCE.md](../QUESTS/QUEST_ENGINE_REFERENCE.md) ✅ |
| **Mapa de cobertura de dominios** (COVERED/PARTIAL/MISSING/SOURCE_REQUIRED) | [GAPS.md](../GAPS.md) ✅ |
| Vertical slice Q00003 Will the Seal be Broken? (combate, race-lock Dark Elf, PARTY_CREDIT_SHARED) | [QUESTS/Q00003_PVE_VERTICAL_SLICE.md](../QUESTS/Q00003_PVE_VERTICAL_SLICE.md) ✅ |
| Vertical slice Q00005 Miner's Favor (delivery sin combate) | [QUESTS/Q00005_PVE_VERTICAL_SLICE.md](../QUESTS/Q00005_PVE_VERTICAL_SLICE.md) ✅ |

### Sistemas de juego planificados (fases futuras)

Los siguientes documentos NO existen todavía; toda consulta sobre ellos debe resolverse consultando el código fuente directamente.

| Question | Plan |
|----------|------|
| How do clans work? | SYSTEMS/CLAN_SYSTEM.md ⧗ |
| How do sieges work? | SYSTEMS/SIEGE_SYSTEM.md ⧗ |
| How do instances work? | SYSTEMS/INSTANCE_SYSTEM.md ⧗ |
| How do zones work? | SYSTEMS/ZONE_SYSTEM.md ⧗ |
| How do events work? | SYSTEMS/EVENT_SYSTEM.md ⧗ |
| How do spawns work? | SYSTEMS/SPAWN_SYSTEM.md ⧗ · consulta práctica ya cubierta por [WORLD/SPAWN_QUERY_GUIDE.md](../WORLD/SPAWN_QUERY_GUIDE.md) ✅ |

---

## PACKETS (Fase 2E)

| Tema | Documento |
|------|-----------|
| Arquitectura (commons + GameServer) | [PACKETS/PACKET_ARCHITECTURE.md](../PACKETS/PACKET_ARCHITECTURE.md) ✅ |
| Incoming packets (base, dispatch) | [PACKETS/INCOMING_PACKETS.md](../PACKETS/INCOMING_PACKETS.md) ✅ |
| Outgoing packets (base, serialización) | [PACKETS/OUTGOING_PACKETS.md](../PACKETS/OUTGOING_PACKETS.md) ✅ |
| Dispatch (opcode → clase, state check) | [PACKETS/PACKET_DISPATCH.md](../PACKETS/PACKET_DISPATCH.md) ✅ |
| Player/connection flow | [PACKETS/PACKET_PLAYER_FLOW.md](../PACKETS/PACKET_PLAYER_FLOW.md) ✅ |
| Security (state/flood/bypass/tamaño) | [PACKETS/PACKET_SECURITY.md](../PACKETS/PACKET_SECURITY.md) ✅ |
| Threading (NIO vs PacketExecutor) | [PACKETS/PACKET_THREADING.md](../PACKETS/PACKET_THREADING.md) ✅ |
| Integración (AI/COMBAT/Skills/Quests/Items) | [PACKETS/PACKET_SYSTEM_INTEGRATION.md](../PACKETS/PACKET_SYSTEM_INTEGRATION.md) ✅ |

---

## NETWORK & COMMUNICATION

| Question | Document |
|----------|----------|
| How does networking work? | [NETWORK/NETWORK_ARCHITECTURE.md](../NETWORK/NETWORK_ARCHITECTURE.md) ✅ |
| What is the packet protocol? | NETWORK/PACKET_PROTOCOL.md ⧗ |
| What are client packets? | NETWORK/CLIENT_PACKETS.md ⧗ |
| What are server packets? | NETWORK/SERVER_PACKETS.md ⧗ |
| How is encryption handled? | NETWORK/ENCRYPTION.md ⧗ |

---

## DATABASE

| Question | Document |
|----------|----------|
| How does database work? | [DATABASE/DATABASE_ARCHITECTURE.md](../DATABASE/DATABASE_ARCHITECTURE.md) ✅ |
| What is the database schema? | [DATABASE/SQL_SCHEMA.md](../DATABASE/SQL_SCHEMA.md) ✅ |
| How are database connections configured? | [DATABASE/DATABASE_CONFIGURATION.md](../DATABASE/DATABASE_CONFIGURATION.md) ✅ |
| How are database transactions handled? | [DATABASE/DATABASE_TRANSACTIONS.md](../DATABASE/DATABASE_TRANSACTIONS.md) ✅ |
| How are XML data files loaded? | [DATABASE/XML_DATA_LOADING.md](../DATABASE/XML_DATA_LOADING.md) ✅ |
| How does ID management work? | [DATABASE/ID_MANAGEMENT.md](../DATABASE/ID_MANAGEMENT.md) ✅ |

---

## CONFIGURATION

| Question | Document |
|----------|----------|
| How does configuration load? | [CONFIGURATION/CONFIGURATION_SYSTEM.md](../CONFIGURATION/CONFIGURATION_SYSTEM.md) ✅ |
| What are custom configs? | CONFIGURATION/CUSTOM_CONFIG.md ⧗ |
| What is the config loading order? | CONFIGURATION/CONFIG_LOADING_ORDER.md ⧗ |

---

## THREADING

| Question | Document |
|----------|----------|
| How does threading work? | [THREADING/THREADING_ARCHITECTURE.md](../THREADING/THREADING_ARCHITECTURE.md) ✅ |
| What are task managers? | THREADING/TASK_MANAGERS.md ⧗ |
| How is task scheduling done? | THREADING/TASK_SCHEDULING.md ⧗ |
| What is the thread safety model? | THREADING/THREAD_SAFETY.md ⧗ |

---

## SCRIPTING

| Question | Document |
|----------|----------|
| How does scripting work? | [SCRIPTING/SCRIPT_ENGINE.md](../SCRIPTING/SCRIPT_ENGINE.md) ✅ |
| How are scripts executed? | SCRIPTING/SCRIPT_EXECUTION.md ⧗ |
| What are handlers? | SCRIPTING/HANDLERS.md ⧗ |

---

## REFERENCE INDEXES

| Purpose | Document |
|---------|----------|
| List of all managers (53) | [MANAGERS_INDEX.md](MANAGERS_INDEX.md) ✅ |
| List of entity types | [ENTITY_TYPES_INDEX.md](ENTITY_TYPES_INDEX.md) ✅ |
| Overview of packets (669 clases: 280 client + 389 server) | PACKET_INDEX.md ⧗ |
| Configuration reference | CONFIG_REFERENCE.md ⧗ |
| Handler types | HANDLER_INDEX.md ⧗ |
| File tree overview | FILE_TREE.md ⧗ |

---

## QUICK LOOKUP TABLES

**"How do I add a new system?"**
→ Study [GAMESERVER_ARCHITECTURE.md](../GAMESERVER_ARCHITECTURE.md) for initialization pattern
→ Create manager singleton following existing patterns
→ Add to GameServer startup sequence

**"How do I add a network packet?"**
→ Read [NETWORK/NETWORK_ARCHITECTURE.md](../NETWORK/NETWORK_ARCHITECTURE.md) ✅
→ Study [PACKETS/PACKET_ARCHITECTURE.md](../PACKETS/PACKET_ARCHITECTURE.md) ✅ for dispatch/state
→ Create ClientPacket class extending ReadablePacket
→ Register in GamePacketHandler
→ Implement readImpl() and runImpl()

**"How do I add a database table?"**
→ Read [DATABASE/DATABASE_ARCHITECTURE.md](../DATABASE/DATABASE_ARCHITECTURE.md) ✅
→ Create SQL migration
→ Create TableData loader class
→ Load in GameServer initialization

**"How do I add a quest?"**
→ Read [QUESTS/QUEST_ARCHITECTURE.md](../QUESTS/QUEST_ARCHITECTURE.md) y [QUESTS/QUEST_LIFECYCLE.md](../QUESTS/QUEST_LIFECYCLE.md)
→ Create quest script extending Quest (`dist/game/data/scripts/quests/`)
→ El constructor se auto-registra vía ScriptManager (`addQuest` si id>0 / `addScript` si no)
→ Se carga al arranque por ScriptEngine (gate `DevelopmentConfig.NO_QUESTS`)

---

## VERIFICATION STATUS

| Document | Status | Last Verified |
|----------|--------|----------------|
| README / OVERVIEW / STRUCTURE | VERIFIED | 2026-08-23 |
| GAMESERVER / LOGINSERVER / COMMONS | VERIFIED | 2026-08-23 |
| NETWORK / CONFIG / THREADING / SCRIPTING | PARTIAL | 2026-08-23 |
| DATABASE/* (6 docs de Fase 2F) | VERIFIED con UNKNOWN explícitos | 2026-08-23 |
| SYSTEMS/ (8 docs de Fase 2A) | VERIFIED | 2026-08-23 |
| AI/* y COMBAT/* (12 docs de Fase 2B) | VERIFIED | 2026-08-23 |
| SKILLS/* (12 docs: 10 de Fase 2C + SKILL_SEMANTIC_REFERENCE + ABNORMAL_TYPE_REFERENCE) | VERIFIED (semántica 2.5 añadida) | 2026-08-26 |
| QUESTS/* (10 docs: 7 de Fase 2D + Q00039/Q00005 análisis + QUEST_RESEARCH_FRAMEWORK) | VERIFIED (Q00039/Q00005 server-side; framework metodológico) | 2026-08-25 |
| PACKETS/* (8 docs de Fase 2E) | VERIFIED | 2026-08-23 |
| INDEXES/* | VERIFIED (corregido 2A, ampliado 2B/2C/2D/2E) | 2026-08-23 |
| Systems planificados (CLAN, SIEGE...) | -- | pendiente |

---

## HOW TO USE THIS INDEX

1. **Identify your question topic** from the tables above
2. **Open the document link** to navigate
3. **Read the document** for detailed explanation
4. **Use document links** for deeper dives
5. **Verify against source code** before implementing
6. **Mark inconsistencies** as REQUIRES CODE VERIFICATION

---

## CONTACT PATTERNS FOR AIs

**Recommended approach**:
```
1. Read this MASTER_INDEX.md
2. Follow link to relevant document
3. Follow document's "Source of Truth" reference
4. Read actual source code
5. Implement based on verified understanding
```

**If UNKNOWN marker found**:
```
1. Check source code directly
2. Create a CODE VERIFICATION TODO
3. Mark in Knowledge Base for future update
4. Do not assume behavior
```

---

## KNOWLEDGE BASE STATUS

**Knowledge Base Version**: **2.4** (2026-08-26)  \
**Markdown Documents**: 88 técnicos + 5 (`00_PROJECT/`) + sistema 11 (AI_INSTRUCTIONS=7, VERSIONING=4) = **104**  \
**Last Updated**: 2026-08-26 (KB v2.4 — SCHEME ENGINE / VALIDATOR DESIGN)  \
**Status**: Proyecto documentado — 4 entidades (SERVER_SOURCE / SERVER_RUNTIME / CLIENT / KNOWLEDGE_BASE)  \
**Current phase**: KB v2.4 — Micro-Sprint 2.6 (SCHEME ENGINE / VALIDATOR) completado como **diseño documentation-only** (análisis SchemeBuffer/Community Board/comparación + diseño y ciclo de vida del validador). Próximos micro-sprints planificados: 2.7 COMMUNITY BOARD → 2.8 PLAYTEST LOOP (ver [00_PROJECT/ROADMAP.md](../00_PROJECT/ROADMAP.md)).  \
**Counting rule**: TÉCNICOS = `.md` excepto `00_PROJECT`, `AI_INSTRUCTIONS`, `VERSIONING`; SISTEMA = AI_INSTRUCTIONS + VERSIONING; 00_PROJECT por separado. Ver [VERSIONING/KB_VERSION.md](../VERSIONING/KB_VERSION.md).  \
**Audit Note 2026-08-26 (AUDIT-005)**: añadida referencia semántica de skills (SKILL_SEMANTIC_REFERENCE) y catálogo AbnormalType (ABNORMAL_TYPE_REFERENCE); extendidos EFFECT_SYSTEM, SKILL_TARGETING y SKILL_DATA_MODEL; GAPS.md actualizado; versión v2.2 → v2.3.
**Audit Note 2026-08-26 (AUDIT-006)**: creada carpeta `BUFFS/` con SCHEME_BUFFER_ANALYSIS, COMMUNITY_BOARD_SCHEME_ANALYSIS y SCHEME_SYSTEM_COMPARISON; creados SKILLS/SCHEME_VALIDATOR_DESIGN y SKILLS/SCHEME_VALIDATION_LIFECYCLE; añadida sección BUFFS a este índice; sistema ampliado (AI_INSTRUCTIONS=7); versión v2.3 → v2.4.

## GETTING STARTED FOR NEW AI TASKS

Ruta recomendada para una tarea nueva:

1. [README.md](../README.md) — reglas de uso de la KB.
2. Este índice ([INDEXES/MASTER_INDEX.md](MASTER_INDEX.md)) — localizar el dominio.
3. [GAPS.md](../GAPS.md) — comprobar si el dominio está COVERED/PARTIAL/MISSING antes de afirmar nada.
4. Documento del dominio correspondiente.
5. Si la tarea involucra quests → [QUESTS/QUEST_ENGINE_REFERENCE.md](../QUESTS/QUEST_ENGINE_REFERENCE.md) (+ [QUEST_PARTY_CREDIT.md](../QUESTS/QUEST_PARTY_CREDIT.md) si hay party/drops).
6. Todo lo clasificado SOURCE_REQUIRED / MISSING → verificar directamente en SERVER_SOURCE (`UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive/`).

Ver [VERSIONING/KB_VERSION.md](../VERSIONING/KB_VERSION.md) y [VERSIONING/CHANGELOG.md](../VERSIONING/CHANGELOG.md).