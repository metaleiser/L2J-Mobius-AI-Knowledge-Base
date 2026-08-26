# ANÁLISIS DE QUEST: Q00005_MinersFavor

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Quest ID**: 5  
**Nombre**: Miner's Favor  
**Nivel mínimo**: 2  
**Capa**: Datapack Quest — Análisis Transversal SERVER SOURCE ↔ RUNTIME

**Sources of Truth** (raíces de entidad):
- **SERVER_SOURCE**: `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive/` — script en `dist/game/data/scripts/quests/Q00005_MinersFavor/`
- **SERVER_RUNTIME**: `L2J_Mobius_CT_2.6_HighFive/` — script en `game/data/scripts/quests/Q00005_MinersFavor/`
- **CLIENT**: `Lineage2-TCT-273-client/` — **fuera de alcance** en este análisis (ver §UNKNOWN_CLIENT)

**Baseline de referencia**: SERVER_SOURCE `e2518ab10872b28cd4c6860e102b493656ba8728` · SERVER_RUNTIME build **26/05/2024**

**Verified**: 2026-08-25  
**Status**: VERIFIED (server-side)

> **Nota de procedencia**: este documento es una reconstrucción reproducible hecha a partir del código existente (SOURCE + RUNTIME), data XML del servidor y diálogos HTML del datapack. No recupera ninguna investigación previa de conversación; no cita el transcript como evidencia.

---

## A. IDENTIFICACIÓN

| Campo | Valor | Estado |
|---|---|---|
| Quest ID | **5** (`super(5)`) | SOURCE_VERIFIED / RUNTIME_VERIFIED |
| Clase | `quests.Q00005_MinersFavor.Q00005_MinersFavor extends Quest` | SOURCE_VERIFIED / RUNTIME_VERIFIED |
| Nombre (docstring) | Miner's Favor | SOURCE_VERIFIED / RUNTIME_VERIFIED |
| Autor | malyelfik | SOURCE_VERIFIED / RUNTIME_VERIFIED |
| Nivel mínimo | 2 (`MIN_LEVEL = 2`) | SOURCE_VERIFIED |
| Repeatable | NO (`exitQuest(false, true)`) | SOURCE_VERIFIED / RUNTIME_VERIFIED |
| Tipo | Delivery / collection (sin kills) | SOURCE_VERIFIED / RUNTIME_VERIFIED |

---

## B. SCRIPTS / UBICACIÓN

| Entidad | Ruta | Archivos |
|---|---|---|
| SERVER_SOURCE | `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive/dist/game/data/scripts/quests/Q00005_MinersFavor/` | 17 |
| SERVER_RUNTIME | `L2J_Mobius_CT_2.6_HighFive/game/data/scripts/quests/Q00005_MinersFavor/` | 17 |

Ambos directorios contienen el **mismo conjunto de 17 archivos**: 1 `.java` + 15 `.html` + 2 `.htm`.

| Archivo | Tipo |
|---|---|
| `Q00005_MinersFavor.java` | Java (lógica) |
| `30517-01.html`, `30517-02.html` | HTML (Shari) |
| `30518-01.html`, `30518-02.html` | HTML (Garita) |
| `30520-01.html`, `30520-02.html` | HTML (Reed) |
| `30526-01.html`, `30526-02.html`, `30526-03.html`, `30526-04.html` | HTML (Brunon) |
| `30554-01.html`, `30554-04.html`, `30554-05.html`, `30554-06.html` | HTML (Bolter) |
| `30554-02.htm`, `30554-03.htm` | HTM (Bolter, aceptación) |

---

## C. NPCs

| ID | Nombre | Título | Función | Evidencia | Estado |
|---|---|---|---|---|---|
| 30554 | Bolter | Miner | Start NPC · talk · turn-in / reward | `Q00005_MinersFavor.java` L37 (SOURCE) · `stats/npcs/30500-30599.xml` L2177 | SOURCE_VERIFIED |
| 30517 | Shari | Armor Merchant | Entrega Boomboom Powder (1550) | `Q00005_MinersFavor.java` L38 (SOURCE) · `stats/npcs/30500-30599.xml` L686 | SOURCE_VERIFIED |
| 30518 | Garita | Accessory Merchant | Entrega Mining Boots (1548) | `Q00005_MinersFavor.java` L39 (SOURCE) · `stats/npcs/30500-30599.xml` L726 | SOURCE_VERIFIED |
| 30520 | Reed | Warehouse Chief | Entrega Redstone Beer (1551) | `Q00005_MinersFavor.java` L40 (SOURCE) · `stats/npcs/30500-30599.xml` L806 | SOURCE_VERIFIED |
| 30526 | Brunon | Blacksmith | Key-gate Miner's Pick (1549) | `Q00005_MinersFavor.java` L41 (SOURCE) · `stats/npcs/30500-30599.xml` L1046 | SOURCE_VERIFIED |

Registro en constructor: `addStartNpc(BOLTER)` y `addTalkId(BOLTER, SHARI, GARITA, REED, BRUNON)` — ambos SOURCE_VERIFIED / RUNTIME_VERIFIED.

---

## D. MOBS

**ABSENT** — la quest no usa `addKillId` ni `onKill`; no hay mobs. `NOT_APPLICABLE`.

---

## E. ITEMS

| ID | Nombre | Función | Cantidad | Evidencia | Estado |
|---|---|---|---|---|---|
| 1547 | Bolter's List | QUEST_PROGRESS (lista de compras) | 1 (regalado) | `stats/items/01500-01599.xml` L517 · Java L44 (SOURCE) | SOURCE_VERIFIED |
| 1548 | Mining Boots | QUEST_PROGRESS (de Garita) | 1 | XML L529 · Java L45 | SOURCE_VERIFIED |
| 1549 | Miner's Pick | QUEST_PROGRESS (de Brunon, key-gate) | 1 | XML L541 · Java L46 | SOURCE_VERIFIED |
| 1550 | Boomboom Powder | QUEST_PROGRESS (de Shari) | 1 | XML L553 · Java L47 | SOURCE_VERIFIED |
| 1551 | Redstone Beer | QUEST_PROGRESS (de Reed) | 1 | XML L565 · Java L48 | SOURCE_VERIFIED |
| 1552 | Bolter's Smelly Socks | QUEST_PROGRESS (key para Brunon, consumible) | 1 → consumido | XML L577 · Java L49 | SOURCE_VERIFIED |
| 906 | Necklace of Knowledge | REWARD | 1 | `stats/items/00900-00999.xml` L95 · Java L50 | SOURCE_VERIFIED |

`registerQuestItems(1547, 1548, 1549, 1550, 1551, 1552)` — **906 NO se registra** como quest item (es reward, no colección).

---

## F. RECOMPENSAS

| Tipo | Valor | Estado |
|---|---|---|
| Item | Necklace of Knowledge (906) ×1 | SOURCE_VERIFIED / RUNTIME_VERIFIED |
| EXP | 5672 | SOURCE_VERIFIED / RUNTIME_VERIFIED |
| SP | 446 | SOURCE_VERIFIED / RUNTIME_VERIFIED |
| Adena | 2466 (`giveAdena(player, 2466, true)`) | SOURCE_VERIFIED / RUNTIME_VERIFIED |
| Repeatable | NO (`exitQuest(false, true)`) | SOURCE_VERIFIED / RUNTIME_VERIFIED |

**Orden de entrega difiere (cosmético)**: SOURCE `giveItems(906)` → `addExpAndSp` → `giveAdena`; RUNTIME `giveAdena` → `addExpAndSp` → `giveItems(906)`. Mismos valores; orden sin impacto funcional.

---

## G. ESTADOS / CONDICIONES

State machine vía `QuestState` (`CREATED` → `STARTED` → `COMPLETED`):

| Estado | Transición | Condición | Evidencia |
|---|---|---|---|
| CREATED | — | `qs.getState() == State.CREATED` | Java onTalk BOLTER (L120 SOURCE) |
| → STARTED | `onEvent("30554-03.htm")` | nivel ≥ 2 | `qs.startQuest()` + dar 1547 + 1552 (L78-80 SOURCE) |
| STARTED (cond 1) | inicio | `qs.isCond(1)` → reminder | `30554-04.html` (L127 SOURCE) |
| STARTED (cond 2) | `checkProgress()` | tener 1547+1548+1549+1550+1551 | `qs.setCond(2, true)` (L199-201 SOURCE) |
| → COMPLETED | `onTalk` BOLTER, STARTED, cond ≠ 1 | entrega recompensa | `qs.exitQuest(false, true)` (L136 SOURCE) |
| COMPLETED | ya completado | — | `getAlreadyCompletedMsg(player)` (L163 SOURCE) |

**Nota de comportamiento**: la rama `else` de BOLTER en STARTED (cond ≠ 1) entrega recompensa sin re-validar items individuales; el gate es el `checkProgress` que fija cond 2. El patrón usa `if (qs.isCond(1)) … else …` (fallthrough), no un `else if (isCond(2))` explícito.

---

## H. FLUJO DE DIÁLOGO (SERVER-SIDE)

| Paso | NPC | Condición previa | Acción | HTML |
|---|---|---|---|---|
| 1 | 30554 (Bolter) | CREATED, nivel < 2 | Mensaje de rechazo | `30554-01.html` |
| 1b | 30554 (Bolter) | CREATED, nivel ≥ 2 | Aceptación | `30554-02.htm` → bypass `30554-03.htm` |
| 2 | 30554 (Bolter) | STARTED | `onEvent("30554-03.htm")`: startQuest + da 1547 + 1552 | `30554-03.htm` |
| 3 | 30517 (Shari) | STARTED | `giveItem(…, 1550)` → da 1550 o recordatorio | `30517-01.html` / `30517-02.html` |
| 4 | 30518 (Garita) | STARTED | `giveItem(…, 1548)` | `30518-01.html` / `30518-02.html` |
| 5 | 30520 (Reed) | STARTED | `giveItem(…, 1551)` | `30520-01.html` / `30520-02.html` |
| 6 | 30526 (Brunon) | STARTED | Si tiene 1552 → bypass; si no → `30526-01.html` | `30526-01.html` |
| 6b | 30526 (Brunon) | STARTED, tiene 1552 | `onEvent("30526-02.html")`: take 1552 + give 1549 + checkProgress | `30526-02.html` / `30526-04.html` |
| 6c | 30526 (Brunon) | STARTED, ya tiene 1549 | Recordatorio | `30526-03.html` |
| 7 | 30554 (Bolter) | STARTED, cond 1 | Reminder de lista | `30554-04.html` → bypass `30554-05.html` |
| 8 | 30554 (Bolter) | STARTED, cond 2 | Recompensa + exitQuest | `30554-06.html` |

---

## I. EVENTOS

| Evento | Disparador | Comprueba | Modifica | Resultado |
|---|---|---|---|---|
| `onEvent("30554-03.htm")` | bypass aceptación | quest activa | `startQuest()` + give 1547 + 1552 | STARTED |
| `onEvent("30526-02.html")` | bypass Brunon | tiene 1552 (sino `30526-04.html`) | take 1552 · give 1549 · checkProgress | cond 2 si completo |
| `onEvent("30554-05.html")` | bypass reminder | — | no-op (repite lista) | `30554-05.html` |
| `onTalk(npc)` | hablar NPC | estado / items / cond | ver §H | HTML según rama |

Helper privado `checkProgress(player, qs)`: si `hasQuestItems(1547,1548,1549,1550,1551)` → `setCond(2, true)`.

Helper privado `giveItem(player, qs, npcId, itemId)`: si no started → noQuestMsg; si ya tiene item → `npcId-02.html`; si no → give + playSound(ITEMSOUND_QUEST_ITEMGET) + checkProgress → `npcId-01.html`.

---

## J. MECÁNICAS CORE

- **Key-item gating**: 1552 (Smelly Socks) desbloquea el diálogo con Brunon y el intercambio por 1549 (Miner's Pick). Sin 1552, `onEvent("30526-02.html")` devuelve `30526-04.html`.
- **Collection trigger**: `checkProgress` verifica 5 items de colección (1547,1548,1549,1550,1551) → cond 2.
- **Consumo**: 1552 se consume (takeItems -1).
- **Sin kills / sin drops / sin party logic**.

---

## K. ARCHIVOS HTML (J)

Inventario completo de 15 `.html` + 2 `.htm` en §B. 4 archivos difieren entre SOURCE y RUNTIME (ver §L. SOURCE vs RUNTIME); los 13 restantes son IDENTICALES.

---

## L. SOURCE vs RUNTIME

### L1. Baseline de hashes (SHA-256)

| Archivo | SOURCE size | RUNTIME size | Estado |
|---|---|---|---|
| 30517-01.html | 367 | 367 | IDENTICAL |
| 30517-02.html | 235 | 235 | IDENTICAL |
| 30518-01.html | 277 | 277 | IDENTICAL |
| 30518-02.html | 220 | 220 | IDENTICAL |
| 30520-01.html | 494 | 494 | IDENTICAL |
| 30520-02.html | 202 | 202 | IDENTICAL |
| **30526-01.html** | 330 | 332 | **DIFFERENT** |
| 30526-02.html | 329 | 329 | IDENTICAL |
| 30526-03.html | 197 | 197 | IDENTICAL |
| 30526-04.html | 198 | 198 | IDENTICAL |
| 30554-01.html | 637 | 637 | IDENTICAL |
| **30554-02.htm** | 607 | 609 | **DIFFERENT** |
| 30554-03.htm | 668 | 668 | IDENTICAL |
| **30554-04.html** | 216 | 218 | **DIFFERENT** |
| 30554-05.html | 623 | 623 | IDENTICAL |
| 30554-06.html | 310 | 310 | IDENTICAL |
| **Q00005_MinersFavor.java** | 6087 | 5069 | **DIFFERENT** |

### L2. Diferencia Java (4 diferencias categorizadas)

| # | Categoría | SOURCE | RUNTIME |
|---|---|---|---|
| 1 | import | `entity.actor.*`, `mechanics.script.*`, `managers.ScriptManager`, `ai.others.NewbieGuide.NewbieGuide` | `model.actor.*`, `model.quest.*`, `enums.QuestSound` |
| 2 | constant | `GUIDE_MISSION = 41` presente | ausente |
| 3 | Newbie Guide block | bloque completo (ScriptManager + NRMemo + condicional) | reemplazado por `showOnScreenMsg` directo |
| 4 | reward order | necklace → exp/sp → adena | adena → exp/sp → necklace (mismo set) |

Lógica core (constructor, onEvent, onTalk, checkProgress, giveItem): **idéntica** en ambos.

### L3. Diferencia HTML (bypass directive convention)

Los 3 HTML difieren solo en el prefijo del bypass:

| Archivo | SOURCE | RUNTIME | Target igual |
|---|---|---|---|
| 30526-01.html | `bypass Script Q00005_MinersFavor 30526-02.html` | `bypass -h Quest Q00005_MinersFavor 30526-02.html` | ✅ |
| 30554-02.htm | `bypass Script Q00005_MinersFavor 30554-03.htm` | `bypass -h Quest Q00005_MinersFavor 30554-03.htm` | ✅ |
| 30554-04.html | `bypass Script Q00005_MinersFavor 30554-05.html` | `bypass -h Quest Q00005_MinersFavor 30554-05.html` | ✅ |

> **El target/evento es el mismo en ambos lados.** La diferencia verificada es la **convención del bypass** (`Script` → `-h Quest`), no un cambio de ruta. Cosmético para el routing.

### L4. Newbie Guide — SOURCE_RUNTIME_CONFLICT (real)

| Aspecto | SOURCE | RUNTIME |
|---|---|---|
| Script lookup | `ScriptManager.getScript(NewbieGuide.class.getSimpleName())` | — (ausente) |
| NRMemo state | `haveNRMemo / setNRMemo / setNRMemoState(GUIDE_MISSION=41)` | — (ausente) |
| Mensaje | condicional (solo si estado NG cambió) | incondicional |
| Datapack script NewbieGuide | **presente** (`ai/others/NewbieGuide/NewbieGuide.java`) | **AUSENTE** (búsqueda `*NewbieGuide*` = 0 en RUNTIME) |

**Verificado**: RUNTIME no contiene el script NewbieGuide en su datapack. En SOURCE, `ScriptManager.getScript(NewbieGuide.class.getSimpleName())` devolvería `null` en un entorno sin el script, y el bloque completo sería no-op. El RUNTIME reemplazó el bloque por un `showOnScreenMsg` directo (mismo NpcStringId `DELIVERY_DUTY_COMPLETE_N_GO_FIND_THE_NEWBIE_GUIDE`) para conservar el mensaje visible al jugador sin la máquina de estados NRMemo.

**Estado**: `SOURCE_RUNTIME_CONFLICT` — comportamiento real de la integración Newbie Guide. No son idénticos. La lógica de la quest en sí (progresión/recompensas) es idéntica; el conflicto es específico de la capa Newbie Guide.

---

## M. CLIENT — UNKNOWN_CLIENT

**Fuera de alcance / no verificado.** Este documento no presenta nombres localizados de cliente como evidencia.

| Entidad | Estado |
|---|---|
| `questname-e.dat` (nombre localizado "Miner's Favor"?) | UNKNOWN_CLIENT |
| `NpcName-e.dat` (nombres localizados de 30517/30518/30520/30526/30554) | UNKNOWN_CLIENT |
| `itemname-e.dat` (1547-1552, 906) | UNKNOWN_CLIENT |
| `NpcString-e.dat` (incl. DELIVERY_DUTY strings) | UNKNOWN_CLIENT |

> No se afirma que exista `l2_decrypt.py`. No se afirma que los nombres cliente estén verificados. La correlación cliente↔servidor sigue pendiente de un método de descifrado validado (ver [../CLIENT_RESEARCH/CLIENT_ENCRYPTION.md](../CLIENT_RESEARCH/CLIENT_ENCRYPTION.md)).

---

## N. PATRONES REUTILIZABLES

| Patrón | Descripción | ¿Generalizable? |
|---|---|---|
| Key-item gating | Un item (1552) desbloquea diálogo/trade con un NPC | ⚠️ Solo quests con item-gating |
| Collection trigger | `checkProgress` con `hasQuestItems(all)` → `setCond(2)` | ⚠️ Quest de colección |
| NPC "already gave" fallback | `npcId-02.html` cuando ya diste el item | ✅ Generalizable |
| Progressive reminder | NPC repite la lista si cond=1 | ✅ Generalizable |
| Reward signature | 906 + EXP 5672 + SP 446 + Adena 2466 | ⚠️ Coincide con Q00001 (ASSOCIATION) |

---

## O. TRAZABILIDAD

| Claim | File | Class/Method | Estado |
|---|---|---|---|
| super(5) | `Q00005_MinersFavor.java` (S/R) | constructor | SOURCE_VERIFIED / RUNTIME_VERIFIED |
| NPCs 30517-30554 | `stats/npcs/30500-30599.xml` L686-2177 (S) | NPC data | SOURCE_VERIFIED |
| Items 1547-1552 | `stats/items/01500-01599.xml` L517-577 (S) | Item data | SOURCE_VERIFIED |
| Item 906 | `stats/items/00900-00999.xml` L95 (S) | Item data | SOURCE_VERIFIED |
| Start/give list+socks | Java onEvent `30554-03.htm` | onEvent | SOURCE_VERIFIED / RUNTIME_VERIFIED |
| Key-gate socks | Java onEvent `30526-02.html` | onEvent | SOURCE_VERIFIED / RUNTIME_VERIFIED |
| checkProgress → cond 2 | Java `checkProgress` | helper | SOURCE_VERIFIED / RUNTIME_VERIFIED |
| Reward 906/5672/446/2466 | Java onTalk BOLTER STARTED else | onTalk | SOURCE_VERIFIED / RUNTIME_VERIFIED |
| NewbieGuide block | Java L138-155 (S) | onTalk | SOURCE_VERIFIED |
| NewbieGuide absent RUNTIME | `L2J_Mobius_CT_2.6_HighFive/game/data/scripts` `*NewbieGuide*` = 0 | filesystem search | RUNTIME_VERIFIED |

---

## P. PLAYER CLUES

| Fuente | Tipo | Texto (verificado HTML) | Objetivo implícito |
|---|---|---|---|
| `30554-03.htm` | EXPLICIT | "get a sack of Boom-boom Powder from Trader Shari … Mining Boots from Trader Garita … Redstone Beer from Warehouse Chief Reed … Miner's Pick from Blacksmith Brunon" | Recoger 4 items |
| `30554-03.htm` | EXPLICIT | "take these [Smelly Socks] -- just in case you need to shock him back to reality!" | Llevar 1552 a Brunon |
| `30526-01.html` | EXPLICIT | "Take out Bolter's Smelly Socks." (bypass) | Usar 1552 para activar diálogo |
| `30554-04.html` | EXPLICIT | "Where's the rest of the supplies I ordered?" | Recordatorio |
| `30554-05.html` | EXPLICIT | repite lista de 4 items | Recordatorio |
| `30517-02.html` | EXPLICIT | "Hurry and take it to Miner Bolter" | Volver a Bolter |
| State | STATE | cond 1 → falta completar colección | Progreso visual |

---

## Q. LORE / INFERENCE

| Afirmación | Tipo | Estado |
|---|---|---|
| "Gray Pillar Guild controls this Strip Mine" | Lore/Narrative (HTML `30554-01/02`) | PROJECT-VERIFIED (HTML) |
| "Fundal broke his leg in a mining accident" | Lore/Narrative (HTML) | PROJECT-VERIFIED (HTML) |
| "Bolter is absent-minded; Brunon is absent-minded" | Player Clue + Lore (HTML) | PROJECT-VERIFIED (HTML) |
| "Dion-brewed Redstone Beer" | Lore/Worldbuilding (HTML `30520-01.html`) | PROJECT-VERIFIED (HTML) |
| Shari "used too much powder… singed his beard" | Character detail (HTML) | PROJECT-VERIFIED (HTML) |
| Filaur/Gray Pillar inter-quest connection | INFERENCE (fuera de esta quest) | UNKNOWN — no investigado aquí |

No se añade lore externo de Lineage II. Todo lo anterior es texto del datapack server-side.

---

## R. QUEST ASSISTANT DATA (extracción)

Datos que una futura herramienta Quest Assistant necesitaría de esta quest:
- Quest state: CREATED / STARTED (cond 1) / STARTED (cond 2) / COMPLETED
- Player context: items de colección (1547-1552), cond actual
- Next objective por estado: cond 1 → recoger items; cond 2 → volver a Bolter
- NPCs: 30554 (start/turn-in), 30517/30518/30520/30526 (colección)
- Recompensas: 906 ×1, EXP 5672, SP 446, Adena 2466
- Claves: 1552 como item-gate
- Confianza: server-side VERIFIED; cliente UNKNOWN_CLIENT

---

## S. TOKEN EFFICIENCY

- Documentos reutilizados: `QUEST_RESEARCH_FRAMEWORK.md` (metodología), `QUEST_ARCHITECTURE.md`, `QUEST_REWARDS.md`, `QUEST_STATES_VARIABLES.md` (APIs core ya documentadas).
- APIs no releídas desde `Quest.java`: `giveItems`, `takeItems`, `giveAdena`, `addExpAndSp`, `playSound`, `setCond`, `exitQuest`, `hasQuestItems`.
- Evitado: leer el core `Quest.java` completo; lectura dirigida de los 17 archivos de la quest + XML puntuales.

---

## T. EJEMPLO

`NOT_APPLICABLE` — no es un ejemplo hipotético; es una quest real investigada.

---

## U. VERIFICACIÓN

- [x] Quest ID 5, clase, autor, MIN_LEVEL — verificados en SOURCE y RUNTIME.
- [x] NPCs (30517, 30518, 30520, 30526, 30554) — verificados en código y XML.
- [x] Items (1547-1552, 906) — verificados en código y XML.
- [x] Recompensas exactas (906, 5672, 446, 2466) — verificadas.
- [x] State machine (CREATED/STARTED/cond 1/cond 2/COMPLETED) — verificada.
- [x] 17 archivos, 4 diferentes SOURCE↔RUNTIME (hash baseline) — verificado.
- [x] Conflicto Newbie Guide documentado como SOURCE_RUNTIME_CONFLICT.
- [x] HTML diffs: solo convención de bypass; targets idénticos.
- [x] CLIENT marcado UNKNOWN_CLIENT — sin claims no verificados.

---

## V. ENLACES

- [QUEST_RESEARCH_FRAMEWORK.md](QUEST_RESEARCH_FRAMEWORK.md)
- [QUEST_ARCHITECTURE.md](QUEST_ARCHITECTURE.md)
- [QUEST_STATES_VARIABLES.md](QUEST_STATES_VARIABLES.md)
- [QUEST_REWARDS.md](QUEST_REWARDS.md)
- [Q00039_REDEYEDINVADERS_ANALYSIS.md](Q00039_REDEYEDINVADERS_ANALYSIS.md)
- [CLIENT_ENCRYPTION.md](../CLIENT_RESEARCH/CLIENT_ENCRYPTION.md)
- [UPSTREAM_BASELINE.md](../VERSIONING/UPSTREAM_BASELINE.md)

---

## W. FUENTE / ESTADO FINAL

**Fuentes**: `game/data/scripts/quests/Q00005_MinersFavor/**` (SOURCE y RUNTIME) · `stats/npcs/30500-30599.xml` · `stats/items/01500-01599.xml` · `stats/items/00900-00999.xml` · `ai/others/NewbieGuide/NewbieGuide.java` (SOURCE)

**Estado**: VERIFIED (server-side) · SOURCE_RUNTIME_CONFLICT en capa Newbie Guide · CLIENT UNKNOWN_CLIENT

**Baseline**: SOURCE `e2518ab` · RUNTIME build 26/05/2024

---

**Status**: VERIFIED (server-side)  
**Verified**: 2026-08-25
