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
| Carga y reload | [SKILLS/SKILL_LOADING.md](../SKILLS/SKILL_LOADING.md) ✅ |
| Aprendizaje / adquisición | [SKILLS/SKILL_LEARNING.md](../SKILLS/SKILL_LEARNING.md) ✅ |
| Flujo de casteo | [SKILLS/CAST_FLOW.md](../SKILLS/CAST_FLOW.md) ✅ |
| Targeting de skills | [SKILLS/SKILL_TARGETING.md](../SKILLS/SKILL_TARGETING.md) ✅ |
| Condiciones | [SKILLS/SKILL_CONDITIONS.md](../SKILLS/SKILL_CONDITIONS.md) ✅ |
| Sistema de efectos | [SKILLS/EFFECT_SYSTEM.md](../SKILLS/EFFECT_SYSTEM.md) ✅ |
| Daño mágico / resistencias | [SKILLS/MAGIC_DAMAGE.md](../SKILLS/MAGIC_DAMAGE.md) ✅ |
| Handlers como scripts | [SKILLS/SKILL_HANDLERS_SCRIPTS.md](../SKILLS/SKILL_HANDLERS_SCRIPTS.md) ✅ |

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
| Metodología de investigación (framework, plantilla A-W, taxon. clues/lore) | [QUESTS/QUEST_RESEARCH_FRAMEWORK.md](../QUESTS/QUEST_RESEARCH_FRAMEWORK.md) ✅ |

### Sistemas de juego planificados (fases futuras)

Los siguientes documentos NO existen todavía; toda consulta sobre ellos debe resolverse consultando el código fuente directamente.

| Question | Plan |
|----------|------|
| How do clans work? | SYSTEMS/CLAN_SYSTEM.md ⧗ |
| How do sieges work? | SYSTEMS/SIEGE_SYSTEM.md ⧗ |
| How do instances work? | SYSTEMS/INSTANCE_SYSTEM.md ⧗ |
| How do zones work? | SYSTEMS/ZONE_SYSTEM.md ⧗ |
| How do events work? | SYSTEMS/EVENT_SYSTEM.md ⧗ |
| How do spawns work? | SYSTEMS/SPAWN_SYSTEM.md ⧗ |

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
| SKILLS/* (10 docs de Fase 2C) | VERIFIED | 2026-08-23 |
| QUESTS/* (7 docs de Fase 2D) | VERIFIED | 2026-08-23 |
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

**Knowledge Base Version**: **2.0** (2026-08-25)  \
**Markdown Documents**: 67 técnicos + 5 (`00_PROJECT/`) + sistema (AI_INSTRUCTIONS=5, VERSIONING=4)  \
**Last Updated**: 2026-08-25 (KB v2.0)  \
**Status**: Proyecto documentado — 4 entidades (SERVER_SOURCE / SERVER_RUNTIME / CLIENT / KNOWLEDGE_BASE)  \
**Current phase**: KB v2.0 (contexto + separación de entidades + taxonomía) completada  \
**Audit Note 2026-08-25 (AUDIT-002)**: SOURCE↔SERVER auditado (source `e2518ab` vs runtime build 26/05/2024); build=Apache Ant; creado `00_PROJECT/`. Histórico 2026-08-24 (AUDIT-001): correcciones FASE 2 — PACKETS/* (280/389), `WritablePacket`, `FloodProtectors` → `gameserver/util`, CONFIGURATION_SYSTEM `.ini`, MANAGERS_INDEX 58/52/6, rutas obsoletas eliminadas.

Ver [VERSIONING/KB_VERSION.md](../VERSIONING/KB_VERSION.md) y [VERSIONING/CHANGELOG.md](../VERSIONING/CHANGELOG.md).