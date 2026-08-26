# AUDIT_HISTORY

Registro cronológico de auditorías. Una auditoría NO incrementa KB_VERSION por sí sola (ver [KB_VERSION.md](KB_VERSION.md)).

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