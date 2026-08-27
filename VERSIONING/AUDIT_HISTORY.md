# AUDIT_HISTORY

Registro cronológico de auditorías. Una auditoría NO incrementa KB_VERSION por sí sola (ver [KB_VERSION.md](KB_VERSION.md)).

---

## AUDIT-008

- **Date**: 2026-08-27
- **KB Version**: 2.4.1 → **2.5** (Sprint 0.6B — reorganización mayor de la KB)
- **Upstream Commit auditado**: `e2518ab10872b28cd4c6860e102b493656ba8728` (sin cambios upstream; baseline preservado)
- **Result**: REORGANIZED

**Checkpoints completados (C0–C4)**:
- **C0**: Inventory — protected areas verified.
- **C1**: Archive originals — 11 verbatim pre-sprint snapshots preserved under `ARCHIVE/PRE_SPRINT_0_6B/`.
- **C2**: `SOURCE_VS_RUNTIME.md` — consolidated divergence reference.
- **C3**: Compressions (ARCHITECTURE_OVERVIEW, COMMONS_ARCHITECTURE), SOURCE_ENTRY conversion (BUILD_AND_DEPLOYMENT), routing compression (MASTER_INDEX), relocations (FASE1_SUMMARY → ARCHIVE/, ABNORMAL_TYPE_REFERENCE → REFERENCE/ABNORMAL_TYPE_CATALOG), merge (SCHEME_VALIDATION.md).
- **C4**: SOURCE_ENTRY conversions for CONFIGURATION/, THREADING/, SCRIPTING/.

**Checkpoint actual (C5 — ROUTING & VERSION)**:
- `AI_BOOTSTRAP.md` — version bump to 2.5, SOURCE_VS_RUNTIME.md added to routing, scheme validation routing updated to consolidated SCHEME_VALIDATION.md.
- `AI_MANIFEST.md` — SOURCE_VS_RUNTIME.md added (ALWAYS, SYSTEM), FASE1_SUMMARY marked DEPRECATED, SCHEME_VALIDATOR_DESIGN + SCHEME_VALIDATION_LIFECYCLE replaced by SCHEME_VALIDATION.md, ABNORMAL_TYPE_REFERENCE → REFERENCE/ABNORMAL_TYPE_CATALOG.md. Line counts updated. Count verification section updated (105 active + 12 archived).
- `GAPS.md` — updated SKILLS, BUFFS/EFFECTS, SCHEME_VALIDATION rows to reflect new document paths.
- `KB_VERSION.md` — 2.4.1 → 2.5, counts updated (105 active / 12 archived / 117 total). New v2.5 changes section.
- `CHANGELOG.md` — Sprint 0.6B entry added with C0–C5 summary.
- `AUDIT_HISTORY.md` — this entry (AUDIT-008).
- `ROADMAP.md` — Sprint 0.6B status updated (C0–C4 complete, C5 in progress, C6/C7 pending).
- `SKILL_SEMANTIC_REFERENCE.md` — link to ABNORMAL_TYPE_REFERENCE → REFERENCE/ABNORMAL_TYPE_CATALOG.md; scheme validator links → SCHEME_VALIDATION.md.
- `MASTER_INDEX.md` — routing verification (already correct from C3).

**Post-change validation**: 105 activos + 12 archive = 117 físicos ✓ · 0 eliminaciones permanentes ✓ · archivos protegidos intactos ✓ · código fuente no modificado ✓ · enlaces internos revisados ✓

**Pendiente (C6/C7)**: Validación final de enlaces cruzados; limpieza de documentos originales redundantes (si corresponde); validación completa del sprint.

---

## AUDIT-007

- **Date**: 2026-08-26
- **KB Version**: 2.4 → **2.4.1** (corrección documental dentro de la línea 2.4; notación parche según política de [KB_VERSION.md](KB_VERSION.md))
- **Upstream Commit auditado**: `e2518ab10872b28cd4c6860e102b493656ba8728` (sin cambios upstream; baseline preservado)
- **Result**: SYNCHRONIZED

**Alcance (Micro-Sprint 2.7 — KB RECONCILIATION PATCH, knowledge-base-only)**: ningún archivo de SERVER_SOURCE / SERVER_RUNTIME / CLIENT / SQL modificado; sin commit ni push; sin documentos nuevos (conteo intacto en 104). No se implementó SchemeValidator ni features de Community Board.

**Hallazgos reconciliados** (investigación 2.7 completada antes de editar; anclas revalidadas puntualmente en RUNTIME `HomeBoard.java`):
- Persistencia Community Board = tabla **`buffer_schemes`** (L167-169); la KB afirmaba `bbs_schemes` → CORREGIDO en ambos docs BUFFS.
- Hook de grabación de schemes = **`_bbsbuffsingle`** (L285-330; llamada `addBuffToScheme()` en **L319**; registro bypass en L83). **`_bbsbuff` NO graba esquemas** (solo aplicación directa) → CORREGIDO.
- Entrega HTML verificada: **L633-640** → `returnHtml` → `CommunityBoardHandler.separateAndSend(returnHtml, player)` (ancla send en L639).
- **`CommunityMaxSchemes`**: propiedad de `config/Custom/CommunityBoard.ini`, default **5** (carga runtime L1090-1102, `getProperty` en L1096; INI físico verificado).
- **SOURCE/RUNTIME PATH_DIVERGENCE**: el engine de schemes existe SOLO en RUNTIME `game/data/scripts/handlers/communityboard/HomeBoard.java`; el homólogo SOURCE `dist/game/data/scripts/handlers/bypass/communityboard/HomeBoard.java` es otra generación sin ese subsystem y con interfaz `IParseBoardHandler` distinta → mismatch marcado **PARTIAL**. Origen exacto de la customización runtime: **PARTIAL** (no se afirma como probado).

**Verificación targeted pre-edición (exactamente una lectura)**:
- `SchemeBufferTable.java` — única copia en el workspace: `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive/java/org/l2jmobius/gameserver/data/SchemeBufferTable.java` (el árbol RUNTIME no contiene Java core). Resultado: LOAD/TRUNCATE/INSERT sobre **`buffer_schemes`** (L64-L66) + `_schemesTable` en memoria (L75). Valor de la celda NPC de SCHEME_SYSTEM_COMPARISON ya coincidía → se PRESERVA. **CONFLICT/SOURCE_REQUIRED**: AUDIT-006 registró persistencia runtime como `custom_buff_schemes`; resolución diferida a una auditoría con acceso al core del build 26/05/2024.

**Line refs válidos preservados**: L372-404 (`_bbsscheme_create`), L444-501 (`_bbsscheme_apply`), L516-556 (`_bbsbuff`), L937-945/L953-967/L975-982/L989-1009/L1017-1022 (helpers de schemes, AUDIT-006).

**Corrections applied**: `BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md` (persistencia, hook, entrega HTML, config, identificación RUNTIME, §1.1 divergencia, tabla de anchors) · `BUFFS/SCHEME_SYSTEM_COMPARISON.md` (celdas persistencia/max schemes, notas de verdad-fuente y conflicto NPC) · `GAPS.md` (arquitectura COMMUNITY BOARD añadida; **PARTIAL intacto**; TELEPORT sigue MISSING aparte) · `AI_INSTRUCTIONS/AI_MANIFEST.md` (layout note) · `VERSIONING/KB_VERSION.md` (2.4.1 + LAST_AUDIT; conteos intactos) · `VERSIONING/CHANGELOG.md` (entrada 2.4.1). Autorización: plan MICRO-SPRINT 2.7 (modo ACT).

**Post-change validation**: 104 físicos ✓ (88 técnicos + 5 + 11) · 0 `.md` nuevos ✓ · enlaces internos revisados en los 7 archivos tocados ✓ · `git diff` confinado a AI_KNOWLEDGE_BASE ✓ · repositorio upstream clean ✓.

---

## AUDIT-006

- **Date**: 2026-08-26
- **KB Version**: 2.3 → **2.4**
- **Upstream Commit auditado**: `e2518ab10872b28cd4c6860e102b493656ba8728` (sin cambios upstream; baseline preservado)
- **Result**: SYNCHRONIZED

**Filesystem state at audit start**:
- KB canónica: v2.3, 83 técnicos + 5 + 11 = **99 `.md`** físicos al inicio (manifest histórico reportaba 97 antes del añadido de AI_INSTRUCTIONS VERIFICATION_RULES/CHANGE_DETECTION).
- Micro-Sprint objetivo: **2.6 SCHEME ENGINE / VALIDATOR** (documentation-only).

**Verificaciones RUNTIME realizadas**:
- `SchemeBuffer.java` (RUNTIME, sin Git): persistencia DB (`custom_buff_schemes`), handlers `createscheme` L292-332 / `skillselect`/`skillunselect` L246-282 / `givebuffs` L143-199, storage CSV `"id,level"`, límites (`BUFFER_MAX_SCHEMES`, `DANCES_MAX_AMOUNT`, `getMaxBuffCount()`).
- Community Board scheme flows: bypass → HTML, add/remove buff sobre string persistido, cast flow.
- Semántica 2.5 usada como base del diseño (Skill identity/categorías/slots ya verificadas en AUDIT-005).

**Nuevos documentos**:
- `BUFFS/SCHEME_BUFFER_ANALYSIS.md`, `BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md`, `BUFFS/SCHEME_SYSTEM_COMPARISON.md` (carpeta nueva).
- `SKILLS/SCHEME_VALIDATOR_DESIGN.md`, `SKILLS/SCHEME_VALIDATION_LIFECYCLE.md`.
- Extendido: `SKILLS/SKILL_SEMANTIC_REFERENCE.md` (cross-links).

**Límites**:
- Diseño only — ninguna clase nueva implementada; SERVER_SOURCE/RUNTIME/CLIENT intactos.
- Reglas del validador condicionadas a evidencia XML por-skill (`SOURCE_REQUIRED`) antes de creerse.

**Post-change validation**: 104 físicos ✓ (= 99 al inicio + 5 nuevos) · links revisados ✓ · solo AI_KNOWLEDGE_BASE modificada ✓.

---

## AUDIT-005

- **Date**: 2026-08-26
- **KB Version**: 2.2 → **2.3**
- **Upstream Commit auditado**: `e2518ab10872b28cd4c6860e102b493656ba8728` (sin cambios upstream; baseline preservado)
- **Result**: SYNCHRONIZED

**Filesystem / Git state at audit start**:
- HEAD `62a47c3` ("KB v2.2: consolidate quest engine and coverage foundations").
- KB canónica: v2.2, 81 técnicos + 5 + 9 = 95 `.md`.

**Verificaciones SOURCE realizadas (contra baseline `e2518ab`)**:
- `Skill.java`: identity fields L72-80, `isDebuff`, `hasNegativeEffect` (`_effectPoint < 0 && targetType != SELF` L941), `isDance()`/`isToggle()`/`isPassive()`/`isTriggeredSkill()`, `minPledgeClass` L867, `checkCondition()` L946-994, `applyEffects()` invul/zone checks L1125/L1130/L1347.
- `EffectList.java`: `getEffectList(Skill)` L211-241 (6 queues), `add(BuffInfo)` L1414-1567 (stacking by AbnormalType, replacement by abnormalLevel, slot overflow, herb/instant handling).
- `AbstractEffect.java`: lifecycle `onStart/onExit/onActionTime/canStart/calcSuccess/getEffectType`.
- `EffectType.java`: 42 enum values.
- `EffectFlag.java`: 22 enum values + mask.
- `TargetType.java`: 38 enum values.
- `AffectScope.java`: 15 enum values.
- `AffectObject.java`: 10 enum values.
- `AbnormalType.java`: 337 enum values.

**Outdated/Contradicted found & fixed**:
- `EFFECT_SYSTEM.md`: stacking marcado como REQUIRES CODE VERIFICATION → actualizado a VERIFIED con reglas de `EffectList.add()`.
- `SKILL_TARGETING.md`: AffectScope/Object mencionados pero no catalogados → catálogos 15/10 añadidos.
- `SKILL_DATA_MODEL.md`: beneficial/harmful reducido a `hasNegativeEffect()` → añadidas 4 capas y categorías.

**Corrections applied**: ver [CHANGELOG](CHANGELOG.md) KB 2.3 (SKILL_SEMANTIC_REFERENCE + ABNORMAL_TYPE_REFERENCE + extensiones). Autorización: plan corregido MICRO-SPRINT 2.5 (PASS WITH WARNINGS).

**Post-change validation**: 97 físicos ✓ (= 95 al inicio + 2 nuevos: SKILL_SEMANTIC_REFERENCE, ABNORMAL_TYPE_REFERENCE) · links revisados ✓ · solo AI_KNOWLEDGE_BASE modificada ✓ · SERVER_SOURCE/RUNTIME/CLIENT intactos ✓.

---

## AUDIT-004

- **Date**: 2026-08-26
- **KB Version**: 2.1 → **2.2**
- **Upstream Commit auditado**: `e2518ab10872b28cd4c6860e102b493656ba8728` (sin cambios upstream; baseline preservado)
- **Result**: SYNCHRONIZED

**Filesystem / Git state at audit start**:
- HEAD `63c9b40` ("KB v0.9: PvE vertical slice + party credit + spawn query consolidation"); commit anterior `d029681` (Q00005 slice). Internamente la KB estaba en v2.1/88 sin reflejar los micro-sprints 2.2–2.3.
- Físicos: **93 `.md`** · Tracked+untracked reconciliados en este sprint.

**Verificaciones SOURCE realizadas (todas contra baseline `e2518ab`)**:
- Quest.java: helpers giveAdena L4634, giveItems L4782/4803/4846, giveItemRandomly L4901/4918/4936, takeItems L5011/5107, playSound L5126/5136, addExpAndSp L5147, getNoQuestMsg/getAlreadyCompletedMsg L1442/1452, getQuestItemsCount L4379, getRegisteredItemIds L2338, hasQuestItems L4483/4494, getRandomPartyMember*/State L1979–2235, addSpawn overloads L4082–4271, openDoor/closeDoor L5334/5352, executeForEachPlayer L5290, addAttackDesire/addMoveToDesire L5422–5450, playMovie L5604–5634, showResult L1190.
- QuestState.java: API completa (startQuest L615, exitQuest L633–722 con limpieza de quest items + COMPLETED/delete según repeatable), memo/cond/variables.
- State.java: CREATED=0/STARTED=1/COMPLETED=2 (byte).
- World.java: utilidad estática (`public class World` + `private World()`), sin getInstance.
- Monster.java: `extends Attackable`.
- Attackable.java: `_onKillDelay=2500`, despacho ON_ATTACKABLE_KILL vía EventDispatcher (L328).

**Outdated/Contradicted found & fixed**:
- QUEST_ARCHITECTURE: conteo quests 543 .java / 511 carpetas → **532 / 510** (recuento filesystem 2026-08-26). Corregido.
- Version metadata stale (88 .md / v2.1) vs real 93 → normalizado a **v2.2 / 93**. Commit `63c9b40` documentado como etiqueta de trabajo "v0.9", no versión canónica.
- Sin contradicciones funcionales detectadas en los dominios auditados (quests/world/entities/party).

**Corrections applied**: ver [CHANGELOG](CHANGELOG.md) KB 2.2 (QUEST_ENGINE_REFERENCE + GAPS.md + hardening). Autorización: plan aprobado del MICRO-SPRINT 2.4.

**Post-change validation**: 95 físicos ✓ (= 93 al inicio del sprint + 2 nuevos: QUEST_ENGINE_REFERENCE, GAPS) · links revisados ✓ · solo AI_KNOWLEDGE_BASE modificada ✓ · SERVER_SOURCE/RUNTIME/CLIENT intactos ✓.

---

## AUDIT-003

- **Date**: 2026-08-25
- **KB Version**: 2.0 → **2.1**
- **Upstream Commit auditado**: `e2518ab10872b28cd4c6860e102b493656ba8728` (sin cambios upstream; baseline preservado)
- **Result**: SYNCHRONIZED

**Filesystem / Git state at audit start**:
- **Físicos**: 88 `.md` (incl. Q00005_MINERSFAVOR_ANALYSIS.md, trabajando aún como *untracked*)
- **Tracked**: 87 · **Untracked**: 1 (Q00005)
- Ramas/directorios: 16 subdirs + root; rama `master`, HEAD `d77b6fd`, 5 commits.

**Recuentos reconciliados (regla canónica, ver KB_VERSION.md)**:
- Técnicos (`EXCL. 00_PROJECT` + `AI_INSTRUCTIONS` + `VERSIONING`) = **74** (incluye INDEXES, root/README, CLIENT_RESEARCH, QUESTS 10).
- `00_PROJECT` = **5**
- Sistema (`AI_INSTRUCTIONS`=5 + `VERSIONING`=4) = **9**
- **TOTAL = 88** (74 + 5 + 9)

**Mayor hallazgo**: el número `67` registrado en AUDIT-001/AUDIT-002 era correcto para el **snapshot v2.0 (81 docs)**; quedó **desactualizado** tras Fase 3 (CLIENT_RESEARCH +4), Q00039 (+1), Fase 3D framework (+1) y Q00005 (+1) → +7 = 74/88. KB_VERSION reportaba 2.0/SYNCHRONIZED aunque esas entregas ya estaban persistidas → metadata DESYNCHRONIZED.

**Reconciliación aplicada**: KB_VERSION 2.0→2.1; STATUS↔SYNCHRONIZED revalidado tras integrar Q00005; conteos 74/5/9/88; AUDIT_PROTOCOL apuntado a la regla canónica (no a un número hardcodeado); MASTER_INDEX/ROADMAP/README/CHANGELOG actualizados. AUDIT-001 y AUDIT-002 **preservados sin cambios** (sus valores `67`/`81` siguen siendo históricos-correctos para el snapshot v2.0).

**Post-change validation**: 88 físicos = 88 tracked tras commit ✓ · enlaces revisados (AUDIT-002) ✓ · 0 huérfanos NUEVOS · SERVER_SOURCE/RUNTIME/CLIENT no modificados ✓.

---

## AUDIT-002

- **Date**: 2026-08-25
- **KB Version**: 1.0 → **2.0** (cambio mayor de organización/metodología → 2.0)
- **Focus**: Auditoría SOURCE↔SERVER de la estructura real del workspace + actualización KB v2.0.
- **Result**: SYNCHRONIZED (documental) — aprobada por el usuario.

**Hallazgos de estructura (verificado, no asumido)**:
- `UPSTREAM/L2J_Mobius` = repo Git real (GitLab oficial `MobiusDevelopment/L2J_Mobius`, rama master, HEAD `e2518ab`, partial clone `blob:none`, sparse-checkout solo H5). Dentro contiene el **source tree completo** `L2J_Mobius_CT_2.6_HighFive/`.
- `L2J_Mobius_CT_2.6_HighFive/` (raíz) = **SERVER_RUNTIME** desplegado: **sin `.git`**, JARs compilados fechados **26/05/2024**, config/data/geodata/runtime. No contiene `java/org/l2jmobius` (solo scripts de datapack en `game/data/scripts`).
- `Lineage2-TCT-273-client/` = cliente H5 (v413, `system/l2.exe`), no modificado.
- `AI_KNOWLEDGE_BASE/` = KB (repo GitHub propio).
- **SOURCE ≠ RUNTIME**: inventarios difieren (source 27.657 vs runtime 25.075; java 3.194 vs 1.320; xml 2.907 vs 2.390; sql 116 vs 118; ini 73 vs 58). Configs `game/config` 23 vs 18 (runtime añade `Character.ini`, omite Database/Development/IdManager/Threads/UndergroundColiseum). `Server.ini` difiere. El runtime incluye geodata; el source no.
- Build del source = **Apache Ant** (`build.xml`), flujo `checkRequirements→init→compile→jar→adding-core→adding-datapack→adding-readme→cleanup`; genera LoginServer.jar, GameServer.jar, DatabaseInstaller.jar, libs de terceros y ZIP de distribución. **No Gradle.**
- Cliente: `system/l2.ini` descifra como "Lineage2 Ver413".

**Cambios aplicados en KB v2.0** (autorización explícita del usuario):
- Creado `00_PROJECT/` (PROJECT_CONTEXT, REFERENCE_SOURCES, DECISIONS, ROADMAP, IDEAS).
- Actualizado `VERSIONING/UPSTREAM_BASELINE.md` (SO baseline vs RUNTIME build), `KB_VERSION.md` (2.0), `AUDIT_HISTORY.md` (esta entrada), `CHANGELOG.md`.
- Actualizado `BUILD_AND_DEPLOYMENT.md` (Ant real + artefactos), `AI_INSTRUCTIONS/AI_README.md` (4 entidades + rutas reales), `AI_INSTRUCTIONS/VERIFICATION_RULES.md` (taxonomía 7 estados), `README.md` (estado v2.0), `INDEXES/MASTER_INDEX.md` (módulo 00_PROJECT y entidades).

**Areas partial / UNKNOWN conservadas**: estados PARTIAL/UNKNOWN/RCV de v1.0 se mantienen donde corresponda; ninguna investigación previa se eliminó.

**Integridad post-cambio**: conocimiento previo preservado · documentos técnicos 67 · sistema (AI_INSTRUCTIONS=5, VERSIONING=4) + 00_PROJECT=5 · enlaces revisados · sin huérfanos injustificados.

---

## AUDIT-001

- **Date**: 2026-08-24
- **KB Version**: 1.0
- **Upstream Commit**: `e2518ab10872b28cd4c6860e102b493656ba8728`
- **Commit Date**: 2026-08-22 03:06:03 +0300
- **Result**: PARTIALLY SYNCHRONIZED

**Areas verified**:
- CONFIGURATION: inventario 16 core + 44 custom configs; `Database.ini` claves/defaults 1:1; `ConfigReader` real; rutas `./config/*.ini`
- DATABASE: `DatabaseFactory` HikariCP (`L2JMobiusPool`, L89); sin FOREIGN KEY/REFERENCES en 116 SQL; arranque `GameServer.java` L210–216 (ConfigLoader→DatabaseFactory→ThreadPool)
- MANAGERS: 58 clases (53 raíz + 5 en games/); 52 con firma `public static …getInstance()`; 6 sin ella
- PACKETS: estructura `commons/network` real; bases `ClientPacket extends ReadablePacket<GameClient>` / `ServerPacket extends WritablePacket<GameClient>`; conteos 280/389
- BUILD/LAYOUT: dist{game,login,db_installer,libs,backup,images}, launcher×3, build.xml; quests en `data/scripts/quests` (513 entradas)

**Areas partial**:
- CONFIGURATION_SYSTEM (claves PVP.ini/Feature.ini sin transcribir), PACKET docs (opcodes/enums RCV; mecánica citada no revalidada), MANAGERS_INDEX (descripciones heredadas para 48; 4 pendientes), COMMONS_ARCHITECTURE, PROJECT_STRUCTURE, NETWORK_ARCHITECTURE, ARCHITECTURE_OVERVIEW, README, LOGINSERVER_ARCHITECTURE

**Unknown (relevantes)**:
hot-reload configs · env-var override · autoCommit global · backup retención · claves Interface.ini/PVP.ini/Feature.ini · catálogo opcodes/enums · jerarquía entidades profunda · mecanismo E/S red (AIO/NIO) · multi-cliente versioning · roles KrateisCube/Lottery/MonsterRace/UndergroundColiseum managers

**Corrections applied**: H1–H18 (ver CHANGELOG.md KB 1.0). Docs modificados: 15. Docs nuevos sistema: 9.

**Integridad post-cambio**: servidor 27.657 archivos ✓ · cliente intacto ✓ · clone clean @ baseline ✓ · 67 docs técnicos ✓ · 0 enlaces rotos · 0 huérfanos.

---

## PLANTILLA PARA FUTURAS AUDITORÍAS

Copiar y rellenar. No inventar resultados.

```
## AUDIT-00N

- **Date**: AAAA-MM-DD
- **KB Version**: <versión vigente al iniciar>
- **Previous Baseline Commit**: <hash o "igual">
- **Upstream Commit auditado**: <hash>
- **Result**: SYNCHRONIZED | PARTIALLY SYNCHRONIZED | DESYNCHRONIZED

**Upstream changes detected**: <ninguno | commits nuevos + resumen name-status>

**Areas verified**: …

**Areas partial**: …

**Outdated/Contradicted found**: …

**Unknown**: …

**Corrections applied**: <IDs o descripción breve; autorización referenciada>

**Docs modified**: <n y lista corta>

**Post-change validation**: conteos ✓/✗ · enlaces ✓/✗ · huérfanos ✓/✗ · integridades ✓/✗

**New baseline (si aplica)**: NEW_UPSTREAM_BASELINE_COMMIT = <hash>
```