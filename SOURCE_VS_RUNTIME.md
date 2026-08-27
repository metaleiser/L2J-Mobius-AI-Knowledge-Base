# SOURCE vs RUNTIME

**Purpose**: Consolidate all known SOURCE↔RUNTIME divergences already documented in the existing KB, as a single reference for AI agents that implement gameplay features and need to distinguish between upstream code (`SOURCE`) and the deployed server (`RUNTIME`).

**Scope**: This document is a **consolidation only**. It introduces no new investigation, does not resolve any contradiction, and does not declare any divergence as a bug. Each divergence was identified and classified by the original documenting checkpoint or research document.

**Authority**: Every claim cites the KB document and line where it was originally established.

**Status**: PARTIAL. This document captures all divergences that were *already* documented as of KB v2.4.1 (Sprint 0.6B — CHECKPOINT 0). As new divergences are discovered, they should be added using the same classification scheme.

---

## Baseline

| Entity | Identifier | Source |
|---|---|---|
| **SOURCE** (upstream) | Commit `e2518ab10872b28cd4c6860e102b493656ba8728` — cloned from `https://gitlab.com/MobiusDevelopment/L2J_Mobius.git` branch `master` | `VERSIONING/UPSTREAM_BASELINE.md` :L9 |
| **RUNTIME** (deployed build) | Build date **26/05/2024** — `GameServer.jar`/`LoginServer.jar`; no `.git` — extracted from a distribution archive | `VERSIONING/UPSTREAM_BASELINE.md` :L25–L28 |
| **Relationship** | The RUNTIME "derives from the same H5 ecosystem but **does not match** `e2518ab` (configurations, tables and datapack differ)". | `VERSIONING/UPSTREAM_BASELINE.md` :L28 |
| **Structural audit** | SOURCE: 27.657 files (3.194 `.java`, 2.907 `.xml`). RUNTIME: 25.075 files (1.320 `.java` — scripts only —, 2.390 `.xml`). Differences confirm non-identity. | `VERSIONING/UPSTREAM_BASELINE.md` :L39–L50 |

> ⚠️ **Rule**: "Do **NOT** assert that the runtime is identical to upstream or that both represent the same commit. The correct relationship is **SOURCE BASELINE vs RUNTIME BUILD**." — `UPSTREAM_BASELINE.md` :L30–L31
## Divergence Matrix

| # | System | KB Document | Classification | Status in Source Doc |
|---|---|---|---|---|
| 1 | Q00001 (Letters of Love) | `CLIENT_RESEARCH/QUEST_PILOT_Q00001.md` | **CONFLICT** | CONFLICT / registrada |
| 2 | Q00005 (Miner's Favor) | `QUESTS/Q00005_MINERSFAVOR_ANALYSIS.md` | **SOURCE_RUNTIME_CONFLICT** | SOURCE_RUNTIME_CONFLICT (real) |
| 3 | Q00039 (RedEyedInvaders) | `QUESTS/Q00039_REDEYEDINVADERS_ANALYSIS.md` | **CONFLICT** | CONFLICT (SOURCE↔RUNTIME §7 [VERIFIED]) |
| 4 | Community Board schemes | `BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md` | **PATH_DIVERGENCE** (attribution PARTIAL) | PATH_DIVERGENCE / PARTIAL |
| 5 | SchemeBuffer persistence | `BUFFS/SCHEME_SYSTEM_COMPARISON.md` | **CONFLICT / SOURCE_REQUIRED** | CONFLICT (⚠️ persistencia NPC) |
| 6 | SOURCE baseline vs RUNTIME build | `VERSIONING/UPSTREAM_BASELINE.md` | **DIVERGENCE** (framed as baseline vs build) | disparidad documentada baseline ↔ build |
| 7 | Configuration system | `CONFIGURATION/CONFIGURATION_SYSTEM.md` | **PARTIAL / UNKNOWN** | PARTIAL (hot-reload/validation UNKNOWN) |

---

## Detailed Divergences

### 1. Q00001 — Letters of Love (NewbieGuide integration)

**Evidence location**: `CLIENT_RESEARCH/QUEST_PILOT_Q00001.md` §7 (lines 66–79), §9 (line 104)

| Aspect | SOURCE | RUNTIME |
|---|---|---|
| NewbieGuide integration (mission 41) | **YES** — imports `ScriptManager` and `ai.others.NewbieGuide.NewbieGuide`; consults/advances GUIDE_MISSION via `haveNRMemo`/`setNRMemo`/`setNRMemoState` | **NO** — no NewbieGuide integration; emits `showOnScreenMsg(...DELIVERY_DUTY...)` directly |
| `showOnScreenMsg(DELIVERY_DUTY...)` | Conditional (only in specific case) | Direct (always in that branch) |
| Imports | Includes `ScriptManager` and `NewbieGuide` | Does not include them |

**SOURCE text** (line 70–71):
> "SERVER_SOURCE: en cond 4 (Darin) incluye integración con `ai.others.NewbieGuide.NewbieGuide`: recupera el script `NewbieGuide`, consulta/avanza una misión guía `GUIDE_MISSION = 41` con `haveNRMemo/setNRMemo/setNRMemoState`, y solo en ciertos casos emite `showOnScreenMsg(...DELIVERY_DUTY...)`. Importa `ScriptManager` y `NewbieGuide`."

**RUNTIME text** (line 71):
> "SERVER_RUNTIME: en cond 4 (Darin) NO hay integración NewbieGuide; simplemente emite `showOnScreenMsg(...DELIVERY_DUTY...)`, da 906, 5672/446 EXP/SP, 2466 adena, y `exitQuest`."

**Conclusion from source doc** (line 79):
> "el runtime es una versión más simple/anterior respecto al source en esta quest (el source añade la integración con la guía de novatos). Es consistente con el hecho conocido de que SOURCE baseline ≠ RUNTIME build."

**Classification**: CONFLICT.
### 2. Q00005 — Miner's Favor (NewbieGuide block)

**Evidence location**: `QUESTS/Q00005_MINERSFAVOR_ANALYSIS.md` L4 (lines 220–231), summary line 365

| Aspect | SOURCE | RUNTIME |
|---|---|---|
| Newbie Guide block | Complete block (ScriptManager + NRMemo + conditional) | Replaced by direct `showOnScreenMsg` |
| Script lookup | `ScriptManager.getScript(NewbieGuide.class.getSimpleName())` | Absent |
| Datapack script NewbieGuide | **Present** (`ai/others/NewbieGuide/NewbieGuide.java`) | **Absent** (search `*NewbieGuide*` = 0 in RUNTIME datapack) |

**SOURCE doc states** (lines 229, 231):
> "RUNTIME no contiene el script NewbieGuide en su datapack. En SOURCE, `ScriptManager.getScript(NewbieGuide.class.getSimpleName())` devolvería null porque la clase no está registrada."

> "Estado: `SOURCE_RUNTIME_CONFLICT` — comportamiento real de la integración Newbie Guide. No son idénticos. La lógica de la quest en sí (progresión/recompensa) es idéntica."

**Classification**: **SOURCE_RUNTIME_CONFLICT**.
### 3. Q00039 — RedEyedInvaders (package/import/license + HTML count)

**Evidence location**: `QUESTS/Q00039_REDEYEDINVADERS_ANALYSIS.md` §7 (lines 144–154), §6 (lines 135–138), §10 (line 185)

| Aspect | SOURCE (UPSTREAM) | RUNTIME |
|---|---|---|
| License header | MIT ("Copyright (c) 2013 L2jMobius") | GPL v3+ |
| Imports | `entity.actor.*`, `entity.item.holders.ItemHolder`, `mechanics.script.Quest` | `model.actor.*`, `model.holders.ItemHolder`, `model.quest.Quest` |
| `@author` | `Janiko` | `janiko` |

**Logic** (line 154):
> "La lógica de la quest (constantes, eventos, onKill, condiciones) es **idéntica** en ambos [VERIFIED]. La divergencia de paquetes sugiere que RUNTIME proviene de una generación distinta del source del mismo ecosistema."

**Additional documented discrepancy** (lines 136–138):
> "4 `.htm`: 30334-01, 30334-02, 30334-03, 30334-04 (Guard Babenco)"
> "**CONFLICT**: una instrucción de consolidación indicaba '3 .htm + 11 .html'. El conteo real verificado en filesystem (SOURCE y RUNTIME) es **4 .htm + 10 .html**."

**Classification**: **CONFLICT** (SOURCE↔RUNTIME, also HTML count).
### 4. Community Board scheme subsystem (PATH_DIVERGENCE)

**Evidence location**: `BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md` §1.1 (lines 18–23)

| Aspect | SOURCE | RUNTIME |
|---|---|---|
| Scheme subsystem | **Absent** — `HomeBoard.java` in upstream baseline `e2518ab` is a different generation without scheme subsystem; interface uses `IParseBoardHandler` | **Present** — `game/data/scripts/handlers/communityboard/HomeBoard.java` (build 26/05/2024); HTML delivery via `CommunityBoardHandler.separateAndSend(returnHtml, player)` (L633–640) |
| Interface | `IParseBoardHandler` — does not match runtime handler | Separate handler, interface mismatch |

**Source doc** (lines 20–23):
> "El motor de schemes existe solo en SERVER_RUNTIME."
> "En el baseline upstream SOURCE (`e2518ab`), el homólogo es una generación distinta sin subsystem de schemes; su interfaz `IParseBoardHandler` no coincide con la del handler runtime → mismatch de interfaz respecto al baseline marcado PARTIAL."
> "El origen exacto de la personalización runtime no se afirma como probado (clasificación PARTIAL; posible port/adaptación de otra fuente, sin evidencia concluyente en este sprint)."
> "Registrado también en GAPS.md bajo COMMUNITY BOARD (PARTIAL) como SOURCE/RUNTIME PATH_DIVERGENCE."

**Classification**: **PATH_DIVERGENCE** (attribution: **PARTIAL**).
### 5. SchemeBuffer persistence table name (CONFLICT / SOURCE_REQUIRED)

**Evidence location**: `BUFFS/SCHEME_SYSTEM_COMPARISON.md` line 27 (⚠️ note)

| Aspect | SOURCE | RUNTIME |
|---|---|---|
| Persistence table | `SchemeBufferTable.java` L64–L66: `buffer_schemes` (the only copy in workspace is in SOURCE) | AUDIT-006 previously recorded `custom_buff_schemes` (runtime) |

**Source doc** (line 27):
> "⚠️ **Persistencia NPC (CONFLICT / SOURCE_REQUIRED)**: la única copia de `SchemeBufferTable.java` en el workspace está en SOURCE (`java/org/l2jmobius/gameserver/data/`, L64-L66: `buffer_schemes`). AUDIT-006 había registrado la persistencia runtime del NPC como `custom_buff_schemes`. Sin acceso al core del build 26/05/2024 no se resuelve qué generación aplica al deploy actual → la celda NPC se conserva tal cual y el conflicto queda documentado."

**Classification**: **CONFLICT / SOURCE_REQUIRED**.
**Status**: Unresolved — requires access to the runtime build core JAR to determine which table name is actually used.
### 6. SOURCE baseline vs RUNTIME build (structural divergence)

**Evidence location**: `VERSIONING/UPSTREAM_BASELINE.md` passim (lines 1–50)

| Metric | SOURCE (`e2518ab`) | RUNTIME (26/05/2024) |
|---|---|---|
| Total files | 27.657 | 25.075 |
| `.java` | 3.194 | 1.320 (scripts only) |
| `.xml` | 2.907 | 2.390 |
| `.sql` | 116 | 118 |
| `.ini` | 73 | 58 |
| Git | Yes (cloned upstream) | No `.git` |

**Source doc** (lines 15, 28, 30–31):
> "IMPORTANTE: este baseline describe el SERVER_SOURCE (código upstream). NO describe el SERVER_RUNTIME."
> "Deriva del mismo ecosistema H5, pero NO coincide con `e2518ab` (configs, tablas y datapack difieren)."
> "NO afirmar que el runtime es idéntico al upstream ni que ambos representan el mismo commit. La relación correcta es SOURCE BASELINE vs RUNTIME BUILD."

**Classification**: Structural **DIVERGENCE** — framed as "SOURCE BASELINE vs RUNTIME BUILD", not as a bug.
### 7. Configuration system unknowns (PARTIAL / UNKNOWN)

**Evidence location**: `CONFIGURATION/CONFIGURATION_SYSTEM.md` lines 6, 96, 103, 170–173, 217

| Aspect | Documented status |
|---|---|
| Overall status | **PARTIAL** — inventory of configs VERIFIED against code; examples corrected to INI |
| Hot-reload | **UNKNOWN** — "Whether configs can be reloaded without server restart" |
| Validation | **UNKNOWN** — "UNKNOWN - exact validation rules" |
| Override mechanism | **Defaults**: hardcoded as second argument in `config.getXxx(key, default)`. **Runtime**: `.ini` file values override. **Environment/hot-reload**: UNKNOWN — "sin evidencia en el código auditado" |

**Source doc** (lines 6, 170–173, 217):
> "Status: PARTIAL (inventario de configs VERIFIED contra código; ejemplos corregidos; hot-reload/validation UNKNOWN)"
> "Defaults hardcodeados como segundo argumento de cada llamada `config.getXxx(clave, default)`."
> "Otros posibles mecanismos de override (variables de entorno, recarga en caliente): **UNKNOWN / REQUIRES CODE VERIFICATION** — sin evidencia en el código auditado."
> "Status: PARTIAL (inventario VERIFIED; ejemplos corregidos a INI; hot-reload/validation UNKNOWN)"

**Classification**: **PARTIAL / UNKNOWN**. The documented defaults and INI overrides are VERIFIED; everything about hot-reload and validation remains unresearched.
## Known Unknowns / Unresolved Conflicts

The following remain **open** and explicitly exclude resolution:

1. **NewbieGuide gap (Q00001, Q00005)**: SOURCE contains NewbieGuide/`ScriptManager` integration; RUNTIME lacks both the script and the integration. The cause (upstream new feature? runtime stripped? different branch?) is **not determined**.

2. **Q00039 generation**: Package/import divergence suggests different source-tree generations. The actual generation relationship between the two versions is **not determined**.

3. **Community Board subsystem origin**: The runtime scheme subsystem in `HomeBoard` has no matching SOURCE. Whether it was ported, custom-developed, or pulled from another branch is **not determined** (PARTIAL attribution).

4. **SchemeBuffer persistence table**: `buffer_schemes` (SOURCE) vs `custom_buff_schemes` (runtime — AUDIT-006). Which table the actual runtime build 26/05/2024 uses is **not confirmed** — requires core JAR inspection.

5. **Configuration validation and hot-reload**: Existence and exact rules are **UNKNOWN / REQUIRES CODE VERIFICATION**.

6. **Client-side text (Q00001, Q00005, Q00039)**: All client-localized names (NPC, item, quest, NpcString) are **UNKNOWN** due to client encryption (see `CLIENT_RESEARCH/CLIENT_ENCRYPTION.md`).
## Authority & Usage Rules

1. **SOURCE is authoritative for upstream intent**. Use it as the design reference for implementations aligned with the baseline.

2. **RUNTIME is authoritative for the actual deployed behavior**. When implementing agent-side features, test against RUNTIME, not SOURCE alone.

3. **When SOURCE and RUNTIME conflict, do NOT automatically treat RUNTIME as a bug**. It may represent a different baseline, a fix, a port, or a deliberate customization.

4. **This document consolidates existing evidence; it does not perform new investigation.** Adding new divergences requires updating the source KB document first, then updating this consolidation.

5. **Do NOT claim resolution for any CONFLICT, PATH_DIVERGENCE, or UNKNOWN listed here.** Until explicitly resolved by a future sprint, these are open.

6. **When implementing gameplay features that cross the SOURCE/RUNTIME boundary, consult the source KB document cited for each divergence**, not just this summary.
## Cross-links

- Baseline technical reference: `VERSIONING/UPSTREAM_BASELINE.md`
- Q00001 divergence: `CLIENT_RESEARCH/QUEST_PILOT_Q00001.md` §7
- Q00005 divergence: `QUESTS/Q00005_MINERSFAVOR_ANALYSIS.md` L4 (Newbie Guide)
- Q00039 divergence: `QUESTS/Q00039_REDEYEDINVADERS_ANALYSIS.md` §7
- Community Board divergence: `BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md` §1.1
- SchemeBuffer persistence: `BUFFS/SCHEME_SYSTEM_COMPARISON.md` ⚠️ note
- Configuration unknowns: `CONFIGURATION/CONFIGURATION_SYSTEM.md`
- GAPS (PARTIAL community board): `GAPS.md`
- Client encryption: `CLIENT_RESEARCH/CLIENT_ENCRYPTION.md`
- Change detection protocol: `AI_INSTRUCTIONS/CHANGE_DETECTION.md`

---

*Created during CHECKPOINT 2 of Sprint 0.6B (preservative reorganization). All divergences compiled from existing KB evidence only — no new investigation, no contradictions resolved.*