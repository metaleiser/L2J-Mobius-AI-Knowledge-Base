# CHANGELOG — AI_KNOWLEDGE_BASE

Formato: entradas concisas y cronológicas (descendentes). El detalle completo vive en AUDIT_HISTORY.md y en los informes de sesión.

---

## KB 2.4.1 — 2026-08-26

**Base**: KB v2.4 **preservada sin documentos nuevos**. Objetivo del micro-sprint 2.7: **RECONCILIACIÓN DOCUMENTAL** (knowledge-base-only) — corregir contradicciones detectadas en BUFFS tras la investigación runtime de 2.7. Baseline upstream sin cambios (`e2518ab`). Ver AUDIT-007.

**Correcciones aplicadas**:
- `BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md` — implementación analizada identificada como **SERVER_RUNTIME**; persistencia `bbs_schemes` → **`buffer_schemes`** (runtime `HomeBoard.java` L167-169); hook de grabación corregido a **`_bbsbuffsingle`** (L285-330, `addBuffToScheme()` en L319) con `_bbsbuff` explícitamente **sin** escritura de esquemas; entrega HTML documentada (`CommunityBoardHandler.separateAndSend`, L633-640); `CommunityMaxSchemes` desde `config/Custom/CommunityBoard.ini` (default 5, carga L1090-1102); nueva §1.1 **SOURCE/RUNTIME PATH_DIVERGENCE** (attribution PARTIAL, mismatch `IParseBoardHandler` PARTIAL); nueva tabla de anchors runtime.
- `BUFFS/SCHEME_SYSTEM_COMPARISON.md` — celda Community Board corregida a `buffer_schemes`; celda NPC conservada (`buffer_schemes`, confirmada en SOURCE `SchemeBufferTable.java` L64-66, única copia en workspace) con nota **CONFLICT/SOURCE_REQUIRED** frente al registro runtime `custom_buff_schemes` de AUDIT-006; fila Max schemes ampliada (`config/Custom/CommunityBoard.ini`, property `CommunityMaxSchemes`, default 5).
- `GAPS.md` — dominio COMMUNITY BOARD sigue **PARTIAL**; cobertura arquitectónica verificada añadida (MasterHandler registration, routing `RequestBypassToServer`, dispatch `CommunityBoardHandler`, entrega HTML `HtmlUtil.sendCBHtml`/`separateAndSend`, PATH_DIVERGENCE); TELEPORT continúa MISSING como dominio separado.
- `AI_INSTRUCTIONS/AI_MANIFEST.md` — nota de layout: rutas relativas a `AI_KNOWLEDGE_BASE/`.
- `VERSIONING/KB_VERSION.md` — 2.4 → **2.4.1** (notación parche por política; conteos canónicos intactos: 88/5/11/**104**); LAST_AUDIT → AUDIT-007.
- `VERSIONING/AUDIT_HISTORY.md`, `VERSIONING/CHANGELOG.md` — entradas de esta reconciliación.

**Notas de límite**: documentation-only; nada de SERVER_SOURCE/RUNTIME/CLIENT/SQL tocado; sin commit/push; sin implementación de SchemeValidator ni features de Community Board. Ítem abierto: **CONFLICT/SOURCE_REQUIRED** (tabla NPC runtime `custom_buff_schemes` vs SOURCE `buffer_schemes`), diferido a auditoría futura.

**Conteos tras 2.4.1**: **104 .md** (sin cambios).

---

## KB 2.4 — 2026-08-26

**Base**: KB v2.3 preservada. Objetivo del sprint: **SCHEME ENGINE / VALIDATOR** (micro-sprint 2.6) — análisis del estado actual de los schemes de buffs y diseño documentation-only del validador.

**Nuevos documentos**:
- `BUFFS/SCHEME_BUFFER_ANALYSIS.md` — análisis VERIFIED del SchemeBuffer NPC (RUNTIME `SchemeBuffer.java`): persistencia DB (`custom_buff_schemes`), handlers `createscheme`/`skillselect`/`skillunselect`/`givebuffs`, storage CSV `"id,level"`, conteo real de slots (dances vs buffs), limitaciones.
- `BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md` — schemes vía Community Board: bypass/HTML, add/remove buff, cast flow, almacenamiento como string persistido.
- `BUFFS/SCHEME_SYSTEM_COMPARISON.md` — comparación lado a lado NPC buffer ↔ Community Board con veredicto por dimensión y gaps compartidos.
- `SKILLS/SCHEME_VALIDATOR_DESIGN.md` — diseño verification-first del validador: clasificaciones objetivo, árbol de decisión sobre la semántica v2.5, matriz de evidencia SOURCE/XML, fases futuras (**sin implementar código**).
- `SKILLS/SCHEME_VALIDATION_LIFECYCLE.md` — ciclo de vida runtime de una decisión de validación (entrada → carga → reglas → veredicto → cast/skip → logging).

**Documentos extendidos**: `SKILLS/SKILL_SEMANTIC_REFERENCE.md` (cross-links BUFFS + validator).

**Documentos de soporte actualizados**: `GAPS.md`, `AI_INSTRUCTIONS/AI_BOOTSTRAP.md`, `AI_INSTRUCTIONS/AI_MANIFEST.md`, `AI_INSTRUCTIONS/AI_README.md`, `INDEXES/MASTER_INDEX.md`, `00_PROJECT/ROADMAP.md`.

**Conteos tras Micro-Sprint 2.6**: 88 técnicos + 5 (`00_PROJECT/`) + sistema 11 (AI_INSTRUCTIONS=7, VERSIONING=4) = **104 .md**. KB_VERSION 2.3 → **2.4**.

**Notas de límite**:
- Diseño only: no se implementó ninguna clase nueva ni se modificó SERVER_SOURCE/RUNTIME/CLIENT.
- Toda regla del futuro validador queda condicionada a evidencia por-skill XML (`data/stats/skills/`) antes de afirmarse (`SOURCE_REQUIRED`).

---

## KB 2.3 — 2026-08-26

**Base**: KB v2.2 preservada. Objetivo del sprint: **SKILL / BUFF SEMANTIC FOUNDATION** para habilitar el Scheme Validator (2.6).

**Nuevos documentos**:
- `SKILLS/SKILL_SEMANTIC_REFERENCE.md` — referencia central semántica (skill identity, categories, beneficial/harmful 4-capas, targets, stacking, restrictions, evidence matrix para Scheme Validator).
- `SKILLS/ABNORMAL_TYPE_REFERENCE.md` — catálogo de 337 valores `AbnormalType` extraídos literalmente del enum SOURCE; reglas de stacking con `SOURCE_REQUIRED` para clasificación individual.

**Documentos extendidos**:
- `SKILLS/EFFECT_SYSTEM.md` — catálogos `EffectType` (42 valores) y `EffectFlag` (22 flags); buff category routing (`EffectList.getEffectList`); stacking/replacement rules (`EffectList.add`); slot overflow rules.
- `SKILLS/SKILL_TARGETING.md` — catálogos `AffectScope` (15 valores) y `AffectObject` (10 valores).
- `SKILLS/SKILL_DATA_MODEL.md` — categorías de skill y enrutamiento; semántica beneficial/harmful multicapa.

**Documentos de soporte actualizados**: `GAPS.md`, `INDEXES/MASTER_INDEX.md`, `00_PROJECT/ROADMAP.md`.

**Conteos tras Micro-Sprint 2.5**: 83 técnicos + 5 (`00_PROJECT/`) + sistema 9 = **97 .md**. KB_VERSION 2.2 → **2.3**.

**Notas de límite**:
- No se clasificó cada `AbnormalType` individualmente: la semántica real depende del skill XML correspondiente.
- Conflictos cross-`AbnormalType` → `SOURCE_REQUIRED`; solo se documentó la regla general de reemplazo por `abnormalLevel`.
- Ningún archivo de SERVER_SOURCE / SERVER_RUNTIME / CLIENT fue modificado.

---

## KB 2.2 — 2026-08-26

**Base**: KB v2.1 preservada. Objetivo del sprint: consolidar y endurecer la KB antes de usarla como especificación para construir gameplay con IA (Skill/Buff Semantic → Scheme Validator → Community Board).

**Nuevos documentos**:
- `QUESTS/QUEST_ENGINE_REFERENCE.md` — referencia central de APIs del motor (`Quest#*`, `QuestState#*`, `State`), 15 categorías, identidad por método (líneas = navegación).
- `GAPS.md` — mapa central de cobertura COVERED/PARTIAL/MISSING/SOURCE_REQUIRED de todos los dominios de gameplay.

**Documentos creados en micro-sprints previos ahora contabilizados**: slices Q00003/Q00005, `QUEST_PARTY_CREDIT.md`, `QUEST_VERTICAL_SLICE_TEMPLATE.md`, `WORLD/SPAWN_QUERY_GUIDE.md`.

**Modificados**:
- `QUESTS/QUEST_REWARDS.md` — consultas de inventario (`hasQuestItems`/`getQuestItemsCount`/`getRegisteredItemIds`) + patrones reutilizables (ENGINE PATTERN vs QUEST-SPECIFIC IMPLEMENTATION).
- `QUESTS/QUEST_ARCHITECTURE.md` — conteo quests corregido: 543 .java / 511 carpetas → **532 / 510** (filesystem 2026-08-26).
- `AI/AGGRO_SYSTEM.md` · `COMBAT/DEATH_FLOW.md` — sección "⚠️ QUEST PARTY CREDIT ≠ GENERAL MOB DROP/REWARD DISTRIBUTION" con cross-links a QUEST_PARTY_CREDIT.
- `AI_INSTRUCTIONS/VERIFICATION_RULES.md` — taxonomía ampliada (DIRECTLY_SUPPORTED, ORIENTATION_ONLY, SOURCE_REQUIRED, OUTDATED, CONTRADICTED, UNKNOWN_CLIENT) + metadata estándar de documento.
- `README.md` · `00_PROJECT/ROADMAP.md` · `INDEXES/MASTER_INDEX.md` — sincronizados a v2.2; roadmap de micro-sprints 2.5–2.8 (SKILL SEMANTIC → SCHEME VALIDATOR → COMMUNITY BOARD → PLAYTEST) documentado como planificación.

**Conteos tras consolidación**: 81 técnicos + 5 (`00_PROJECT/`) + sistema 9 = **95 .md**. KB_VERSION 2.1 → **2.2**.

**Notas de límite**:
- Ningún archivo de SERVER_SOURCE / SERVER_RUNTIME / CLIENT fue modificado.
- El commit anterior `63c9b40` usó "KB v0.9" como nomenclatura de trabajo; la versión canónica es v2.2 (documentado en KB_VERSION).
- Community Board / Scheme Buffer NO implementados: solo planificados (ROADMAP).

---

## KB 2.1 — 2026-08-25

**Base**: KB v2.0 preservada. Esta entrada cierra la **reconciliación de metadatos** (AUDIT-003: el número de documentos y el estado `SYNCHRONIZED` estaban desactualizados respecto al filesystem tras los commits Fase 3/3D).

**Evolución post-v2.0 (ya persistida en disco/git, no documentada antes en CHANGELOG)**:
- **Fase 3** (`CLIENT_RESEARCH/`): investigación del cliente H5 — estructura del cliente, cifrado de texto, caso piloto Q00001, mapeo autoridad CLIENT↔SERVER. El texto cliente sigue **cifrado** (bloqueo); no se ha validado un método de descifrado (ver `CLIENT_RESEARCH/CLIENT_ENCRYPTION.md`).
- **Q00039** (`QUESTS/Q00039_REDEYEDINVADERS_ANALYSIS.md`): análisis transversal SOURCE↔RUNTIME↔CLIENT↔GAMEPLAY (drops en party, branching, doble colección).
- **Fase 3D** (`QUESTS/QUEST_RESEARCH_FRAMEWORK.md`): marco metodológico A–W para análisis transversal de quests.
- **Q00005 Miner's Favor** (`QUESTS/Q00005_MINERSFAVOR_ANALYSIS.md`): análisis SOURCE↔RUNTIME con documented `SOURCE_RUNTIME_CONFLICT` en la integración Newbie Guide (presente solo en SOURCE; RUNTIME lo reemplaza por `showOnScreenMsg` directo). Incluye baseline SHA-256 de los 17 archivos y verificación server-side (Quest ID 5, NPCs 30517–30554, items 1547–1552 + 906, recompensas 5672 EXP / 446 SP / 2466 adena).
- **Integración de navegación**: referencias cruzadas añadidas en `INDEXES/MASTER_INDEX.md` y `QUESTS/QUEST_ARCHITECTURE.md`.

**Conteos tras reconciliación**: 74 técnicos + 5 (`00_PROJECT/`) + sistema 9 (AI_INSTRUCTIONS=5, VERSIONING=4) = **88 .md**. KB_VERSION 2.0 → **2.1** (nueva área CLIENT_RESEARCH + metodología Fase 3D superan "corrección puntual"; < 3.0).

**Notas de límite**:
- Ningún archivo de SERVER_SOURCE / SERVER_RUNTIME / CLIENT fue modificado.
- Q00005 sigue bajo `UNKNOWN_CLIENT` para textos cliente (no se afirma descifrado validado).
- No se reivindican nombres/localizaciones cliente como evidencia verificada.

### Arquitectura del workspace (nueva realidad documentada)

**Base**: actualización de KB v1.0 (preservando todo el conocimiento previo). Auditoría SOURCE↔SERVER (AUDIT-002) aprobada.

### Arquitectura del workspace (nueva realidad documentada)
- Separación formal en **4 entidades**: `SERVER_SOURCE` (UPSTREAM, Git GitLab, baseline `e2518ab`), `SERVER_RUNTIME` (`L2J_Mobius_CT_2.6_HighFive`, build 26/05/2024, sin Git), `CLIENT` (`Lineage2-TCT-273-client`), `KNOWLEDGE_BASE` (`AI_KNOWLEDGE_BASE`).
- Documentado **SOURCE baseline ≠ RUNTIME build** (source `e2518ab` 2026 vs runtime 26/05/2024; inventarios y configs difieren).
- Build del source = **Apache Ant** (checkRequirements→init→compile→jar→adding-core→adding-datapack→adding-readme→cleanup); genera LoginServer.jar, GameServer.jar, DatabaseInstaller.jar, libs de terceros y ZIP de distribución. **No Gradle.**

### Nuevos documentos (00_PROJECT/)
- `PROJECT_CONTEXT.md` — contexto y mapa de las 4 entidades + reglas de autoridad.
- `REFERENCE_SOURCES.md` — registro de fuentes (repo KB, GitLab Mobius, runtime, cliente, wiki) con propósito/autoridad/uso.
- `DECISIONS.md` — registro de decisiones (D-001…D-003).
- `ROADMAP.md` — hoja de ruta (pendientes + investigaciones futuras).
- `IDEAS.md` — Quest Assistant, AI features/Bots, crónicas superiores (solo ideas).

### Modificados
- `VERSIONING/UPSTREAM_BASELINE.md` — separado SOURCE baseline vs RUNTIME build; registrado el runtime (26/05/2024, sin Git).
- `VERSIONING/KB_VERSION.md` — KB_VERSION 1.0 → 2.0.
- `VERSIONING/AUDIT_HISTORY.md` — entrada AUDIT-002.
- `BUILD_AND_DEPLOYMENT.md` — flujo Ant real + artefactos (sin Gradle).
- `AI_INSTRUCTIONS/AI_README.md` — distinción SOURCE/RUNTIME/CLIENT/KB y rutas reales del workspace.
- `AI_INSTRUCTIONS/VERIFICATION_RULES.md` — taxonomía de 7 estados y compatibilidad histórica.
- `README.md` — estado KB 2.0 y separación de entidades.
- `INDEXES/MASTER_INDEX.md` — secciones 00_PROJECT, entidades, client research, experiments, decisiones, roadmap, ideas.

### Validación (resumen)
- Enlaces y huérfanos revisados (ver AUDIT_HISTORY AUDIT-002).
- Conocimiento previo preservado; no se eliminó ninguna investigación.

---

## KB 1.0 — 2026-08-24

**Baseline**: `e2518ab10872b28cd4c6860e102b493656ba8728` (master, 2026-08-22 03:06:03 +0300)


### Consolidación inicial
- KB consolidada sobre snapshot local verificado estructuralmente contra el commit baseline (inventarios exactos: java 3194 · xml 2907 · sql 116 · ini 73 · properties 0 · total 27657).

### Auditoría FASE 1
- Inventario físico: 67 documentos técnicos en 13 áreas; fechas únicas 2026-08-23.
- Enlaces: ~85 destinos internos; 1 enlace malformado (`](SYSTEMS/)`); huérfano legítimo: MASTER_INDEX.
- Contradicciones internas detectadas (C1–C6): estados de fase, conteos packets, tablas MANAGERS_INDEX, README obsoleto.

### Auditoría FASE 2 (código ↔ KB @ baseline)
- Hallazgos H1–H15 clasificados por severidad (HIGH/MEDIUM/LOW) con evidencia citada.
- Áreas verificadas contra código: configuración (16+44), DatabaseConfig/DatabaseFactory (HikariCP/L2JMobiusPool), sin FK en SQL, arranque GameServer L210–216, managers 58/52/6, bases de packets, layout/build.

### Correcciones aplicadas (autorización explícita del usuario)
- H1–H4: conteos unificados clientpackets=280 / serverpackets=389 / total=669.
- H5: eliminado `commons/network/network/NetworkThread` (clase inexistente); sin claims NIO.
- H6: `FloodProtectors` → `gameserver/util/`.
- H7/H8: CONFIGURATION_SYSTEM reescrito al mecanismo real (`ConfigReader` + `load()` + `.ini`; ejemplo verbatim de `DatabaseConfig.java`).
- H9: override por variables de entorno → UNKNOWN.
- H10: rutas personales obsoletas (`c:\L2J KNOLEDGE`) eliminadas en 5 documentos.
- H11/H12: README sincronizado con MASTER_INDEX (fase 2F) y estructura real de carpetas.
- H13: MANAGERS_INDEX regenerado desde código (58=52+6).
- H14: `dist/game/log` marcado runtime-generated.
- H15: referencias "750+" corregidas a 669.
- Nuevas durante validación: **H16** `SendablePacket`→`WritablePacket` (3 docs) · **H17** genérico `<String>`→`<GameClient>` · **H18** residuos 285/398/750+/Thread.ini/sobreafirmación singletons.
- Total documentos modificados: **15**. Documentos técnicos creados: **0** (67 constantes).

### Sistema de mantenimiento (FASE 3–4)
- Creado `AI_INSTRUCTIONS/` (AI_README, VERIFICATION_RULES, AUDIT_PROTOCOL, UPDATE_PROTOCOL, CHANGE_DETECTION).
- Creado `VERSIONING/` (KB_VERSION, UPSTREAM_BASELINE, CHANGELOG, AUDIT_HISTORY).
- MASTER_INDEX: sección mínima de descubrimiento hacia ambos directorios.

### Áreas VERIFIED (contra e2518ab)
CONFIGURATION (inventario+Database.ini+ConfigReader) · DATABASE core (pool/FK/arranque) · MANAGERS (recuento/firmas) · PACKETS (estructura/conteos/bases) · BUILD & LAYOUT · arranque GameServer.

### Áreas PARTIAL
CONFIGURATION_SYSTEM (claves PVP/Feature RCV) · PACKET docs (opcodes/enums RCV) · MANAGERS_INDEX (descripciones heredadas) · COMMONS/NETWORK/ARCHITECTURE_OVERVIEW/PROJECT_STRUCTURE/README/LOGINSERVER_ARCHITECTURE (parcialmente auditadas).

### UNKNOWN relevantes
hot-reload configs · env override · autoCommit global · backup retención · claves Interface/PVP/Feature.ini · catálogo opcodes/enums · jerarquía entidades profunda · mecanismo E/S red · multi-cliente · roles 4 managers games/.

---