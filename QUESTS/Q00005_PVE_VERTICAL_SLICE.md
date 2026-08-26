# Q00005 — Miner's Favor

## 1. Identity

| Field | Value | Status | Confidence |
|---|---|---|---|
| Quest ID | **5** (`super(5)` en constructor) | VERIFIED | Alta |
| Quest name | Miner's Favor (docstring `/** Miner's Favor (5) */`) | VERIFIED | Alta |
| Java class | `quests.Q00005_MinersFavor.Q00005_MinersFavor extends Quest` | VERIFIED | Alta |
| Author | malyelfik (`@author`) | VERIFIED | Alta |
| Minimum level | **2** (`MIN_LEVEL = 2`, chequeo `player.getLevel() >= MIN_LEVEL` en `onTalk BOLTER`) | VERIFIED | Alta |
| Quest type | **Delivery / collection** (sin kills, sin loot, sin drop chance). No `addKillId`, no `onKill`. | VERIFIED | Alta |
| Repeatable | **NO** — `qs.exitQuest(false, true)` (condición `false` = no repetible) | VERIFIED | Alta |
| Start NPC | 30554 (Bolter, Miner) — `addStartNpc(BOLTER)` | VERIFIED | Alta |
| Talk NPCs | 30554, 30517, 30518, 30520, 30526 — `addTalkId(BOLTER, SHARI, GARITA, REED, BRUNON)` | VERIFIED | Alta |
| Completion NPC | 30554 (Bolter) | VERIFIED | Alta |

**Fuente:** `Q00005_MinersFavor.java` (SOURCE y RUNTIME).

## 2. Authority & Sources

Jerarquía aplicada: `SOURCE CODE > RUNTIME > XML/DATA > CLIENT > KB`.

| Capa | Ruta | Contenido | Estado |
|---|---|---|---|
| SERVER_SOURCE | `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive/dist/game/data/scripts/quests/Q00005_MinersFavor/` | Java + 17 HTML/HTM | VERIFIED |
| SERVER_RUNTIME | `L2J_Mobius_CT_2.6_HighFive/game/data/scripts/quests/Q00005_MinersFavor/` | Java + 17 HTML/HTM | VERIFIED |
| SERVER XML items | `game/data/stats/items/01500-01599.xml`, `00900-00999.xml` | IDs y stats de items 1547–1552, 906 | VERIFIED |
| SERVER XML npcs | `game/data/stats/npcs/30500-30599.xml` | IDs/nombres/títulos 30517/30518/30520/30526/30554 | VERIFIED |
| SERVER spawns | `game/data/spawns/HighFiveSpawns.xml` | coordenadas/heading de los 5 NPCs | VERIFIED |
| SERVER mapregion | `game/data/mapregion/dwarf_town.xml` | región `dwarf_town` (locId 921) + subregión `Quarry` | VERIFIED |
| SERVER NpcStringId | `UPSTREAM/.../java/org/l2jmobius/gameserver/network/NpcStringId.java` | ids 501/502/503 MINER_S_FAVOR; `DELIVERY_DUTY_COMPLETE_N_GO_FIND_THE_NEWBIE_GUIDE` | VERIFIED |
| CLIENT | `Lineage2-TCT-273-client/system/*.dat`, `L2text/` | questname-e.dat, NpcName-e.dat, itemname-e.dat, NpcString-e.dat | **UNKNOWN_CLIENT** (cifrado) |
| CLIENT blocking | `AI_KNOWLEDGE_BASE/CLIENT_RESEARCH/CLIENT_ENCRYPTION.md` | documenta el bloqueo del cifrado | VERIFIED (estado del bloqueo) |
| KB previa | `AI_KNOWLEDGE_BASE/QUESTS/Q00005_MINERSFAVOR_ANALYSIS.md` | análisis transversal | *secondary*; supervisado por code |

**Conflicto real documentado:** la integración **Newbie Guide** (`NpcStringId.DELIVERY_DUTY_COMPLETE_N_GO_FIND_THE_NEWBIE_GUIDE`) está **presente en SOURCE** (bloque con `ScriptManager.getScript(NewbieGuide)`, NRMemo, `GUIDE_MISSION=41`, mensaje condicional) pero **ausente en RUNTIME** (RUNTIME lo reemplaza por un `showOnScreenMsg` directo; no hay script NewbieGuide en `game/data/scripts`). **La lógica de la misión es idéntica en ambos**; el conflicto es solo de la capa Newbie Guide (SOURCE_RUNTIME_CONFLICT).

## 3. NPC Chain

Reconstrucción real (evidencia: `onTalk`/`onEvent` + `giveItem`/`checkProgress`):

```
PLAYER (nivel >= 2)
  ↓
30554 BOLTER → onTalk(CREATED): 30554-02/03 → onEvent("30554-03.htm")
              → startQuest() → STARTED(cond1)
              → giveItems(BOLTERS_LIST=1547,1) + giveItems(BOLTERS_SMELLY_SOCKS=1552,1)
  ↓  (orden NO estricto entre Shari/Garita/Reed/Brunon)
30517 SHARI   → onTalk → giveItem(BOOMBOOM_POWDER=1550,1)
30518 GARITA  → onTalk → giveItem(MINING_BOOTS=1548,1)
30520 REED    → onTalk → giveItem(REDSTONE_BEER=1551,1)
30526 BRUNON  → onTalk(STARTED, NO tiene 1549)→30526-01 (bypass "30526-02")
                onEvent("30526-02.html"):
                  si !hasQuestItems(1552) → devuelve 30526-04
                  sí tiene 1552 → takeItems(1552,-1) + giveItems(MINERS_PICK=1549,1) + checkProgress
30554 BOLTER  → onTalk(STARTED, cond != 1) → giveItems(NECKLACE=906,1)
               addExpAndSp(5672,446); giveAdena(2466,true)
               qs.exitQuest(false,true) → COMPLETED (+ showOnScreenMsg Newbie Guide)
```

| NPC | ID | Título | Tipo | Rol exacto | Entrega (cant) | Condición |
|---|---|---|---|---|---|---|
| Bolter | 30554 | Miner | Folk | start + accept + turn-in + reward | da 1547,1552 al iniciar; recompensa 906 + EXP + SP + Adena | nivel ≥ 2 |
| Shari | 30517 | Armor Merchant | Merchant | proveedor 1550 | 1550 ×1 | STARTED |
| Garita | 30518 | Accessory Merchant | Merchant | proveedor 1548 | 1548 ×1 | STARTED |
| Reed | 30520 | Warehouse Chief | VillageMasterDwarf | proveedor 1551 | 1551 ×1 | STARTED |
| Brunon | 30526 | Blacksmith | Trainer | proveedor 1549 (key-gate) | 1549 ×1 | STARTED **y tener 1552** (1552 se consume) |

Los 5 participan en la quest de forma directa (verificado en constructor `addStartNpc`/`addTalkId`). Ningún NPC "recibe entregas de items al final"; el gate de condición lo impone `checkProgress`.

## 4. Conditions

| Condición | Implementada |
|---|---|
| Nivel mínimo | `player.getLevel() >= 2` (onTalk BOLTER CREATED). HTML `30554-01` rechaza si < 2. |
| Prerequisitos de quest | Ninguno (no hay `questId` previas). |
| Condición de aceptación | Clic en bypass `30554-03.htm` desde `30554-02.htm` (solo si nivel ≥ 2). |
| Condición de progreso | `checkProgress()` → `setCond(2)` al tener 1547+1548+1549+1550+1551. |
| Condición de finalización | `onTalk BOLTER` en STARTED con cond ≠ 1. |
| Condición de abandono | No implementada en el script (no hay onQuitQuest). |
| Condición de reinicio | No aplica (no repeatable; `exitQuest(false,true)`). |
| Brunon | requiere `hasQuestItems(player, 1552)` (si no → `30526-04`). |

**IMPORTANTE:** no existen "difficulty tiers" ni ramas condicionales complejas. La rama `if (qs.isCond(1)) {...} else {...}` en BOLTER usa fallthrough, **no** `else if(cond2)`.

## 5. Objectives (real, desde código)

1. Hablar con **30554 Bolter** (nivel ≥2) y aceptar → recibir **1547 Bolter's List** (1) y **1552 Bolter's Smelly Socks** (1). Cond → STARTED cond1.
2. Recoger 4 supplies (el orden entre estos NPCs **NO es estricto**):
   - 30517 Shari → **1550 Boomboom Powder** (1)
   - 30518 Garita → **1548 Mining Boots** (1)
   - 30520 Reed → **1551 Redstone Beer** (1)
   - 30526 Brunon → **1549 Miner's Pick** (1) — únicamente si dispones de 1552 (se consume).
3. `checkProgress()` verifica 5 items: 1547, 1548, 1549, 1550, 1551 → `setCond(2)`.
4. Volver a **30554 Bolter** (cond2) → recibir recompensa y finalizar.

**Categorías de ítems (no confundir):**
- QUEST/COLECCIÓN (regalados o intercambiados): 1547, 1548, 1549, 1550, 1551, 1552.
- **ITEM-GATE consumible:** **1552** (Smelly Socks; `takeItems(player,1552,-1)` en el intercambio con Brunon).
- **REWARD:** 906 (cantidad 1).
- El "entrega" final NO implica `takeItems` extra; el gate es `checkProgress`.

## 6. Items (XML verificado)

`stats/items/01500-01599.xml` y `00900-00999.xml`.

| ID | Name | Type | Stackable | Weight | isQuestItem | Regalo por | Cant | Consumido | Reward | Comentario |
|---|---|---|---|---|---|---|---|---|---|---|
| 1547 | **Bolter's List** | EtcItem | true | — | true | BOLTER (inicio) | 1 | No | No | **REQUERIDO** en `checkProgress`. |
| 1548 | Mining Boots | EtcItem | true | — | true | GARITA | 1 | No | No | REQUERIDO en `checkProgress`. |
| 1549 | Miner's Pick | EtcItem | true | — | true | BRUNON (intercambio 1552) | 1 | No | No | REQUERIDO en `checkProgress`. |
| 1550 | **Boomboom Powder** | EtcItem | true | — | true | SHARI | 1 | No | No | REQUERIDO en `checkProgress`. |
| 1551 | Redstone Beer | EtcItem | true | — | true | REED | 1 | No | No | REQUERIDO en `checkProgress`. |
| 1552 | Bolter's Smelly Socks | EtcItem | true | — | true | BOLTER (inicio) | 1 | **Sí** (`takeItems -1`) | No | Gate de Brunon; NO en `checkProgress`. |
| 906 | **Necklace of Knowledge** | Armor (neck) | no | 150 | **No** (reward) | — | 1 | No | **Sí** | Equipable (mDef +18, price 830). |

`registerQuestItems(1547,1548,1549,1550,1551,1552)`. `906` NO está registrado como quest item.

Items involucrados = **7 IDs distintos**: 1547, 1548, 1549, 1550, 1551, 1552, 906. Las 6 primeras son quest items; 906 es reward.

## 7. Combat / Mobs

- `MOBS_REQUIRED = NONE`. `KILLS_REQUIRED = NO`. `COMBAT_REQUIRED = NO`.
- El script NO contiene `addKillId`, `onKill`, `Attackable`, `Monster`, `dropChance`, `giveItemRandomly` ni `getRandomPartyMember`.
- No se crea ningún spawn de mob ficticio.
- No hay "mobs indirectos": todos los items son entregados por NPCs.

## 8. World / Locations

- **Región formal**: `mapregion/dwarf_town.xml` → región `dwarf_town` (town "Dwarven Town", castle 9, **locId = 921**, bbs 9), con subregión **`Quarry`** (misma town 921).
- **"Strip Mine" = LORE / CONTEXT**. No existe ninguna zona formal (`zones/*.xml`) ni `mapregion` con nombre "Strip Mine" (búsqueda: 0 coincidencias). Aparece solo como texto narrativo en `30554-01.html`: *"Gray Pillar Guild controls this Strip Mine."*
- La quest **no** fija ni teletraslada zonas; solo los NPCs están posicionados en el mapa.

## 9. Spawns

Fuente: `game/data/spawns/HighFiveSpawns.xml` (spawn estático `<npc id=…/>`). Todos respawn 60s.

| NPC | ID | x | y | z | heading |
|---|---|---|---|---|---|
| Bolter | 30554 | 112656 | -174890 | -608 | 8192 |
| Reed | 30520 | 115205 | -180024 | -872 | 32768 |
| Brunon | 30526 | 115315 | -182155 | -1440 | 49152 |
| Garita | 30518 | 115900 | -177316 | -880 | 50000 |
| Shari | 30517 | 116192 | -181072 | -1336 | 40960 |

Todas las coordenadas están dentro de la zona Dwarven Village / mina (región 921 + Quarry). Mecanismo de spawn = **XML estático** (`HighFiveSpawns.xml`), no script, no database, no spawn group dinámico.

## 10. Rewards (real, literal del código)

| Tipo | Item ID | Nombre | Quantity | Código exacto | Origen |
|---|---|---|---|---|---|
| Item | 906 | **Necklace of Knowledge** | **1** | `giveItems(player, NECKLACE, 1)` | `onTalk BOLTER STARTED else` |
| EXP | — | — | 5672 | `addExpAndSp(player, 5672, 446)` | ídem |
| SP | — | — | 446 | `addExpAndSp(player, 5672, 446)` | ídem |
| Adena | — | — | 2466 | `giveAdena(player, 2466, true)` | ídem |

**Aclaración obligatoria sobre "906":**
- `906` es un **ITEM ID** (`static final int NECKLACE = 906`), **no** una cantidad.
- La cantidad recibida es **1** (`giveItems(player, NECKLACE, 1)`). Interpretar 906 como una cantidad de unidades es **INCORRECTO** y se rechaza en este documento.
- No hay otro reward item, ni reward condicional, ni bonus adicional.

Orden de entregas (cosmético, sin impacto funcional):
- SOURCE: `giveItems(906)` → `addExpAndSp` → `giveAdena`.
- RUNTIME: `giveAdena` → `addExpAndSp` → `giveItems(906)`.

`qs.exitQuest(false, true)` → COMPLETED, no repeatable. `showOnScreenMsg(... DELIVERY_DUTY_COMPLETE_N_GO_FIND_THE_NEWBIE_GUIDE)` al finalizar (capa Newbie Guide; SOURCE con memo-state, RUNTIME con `showOnScreenMsg` directo).

## 11. Quest State Machine

```
AVAILABLE / CREATED
   → [onEvent "30554-03.htm" (startQuest)] → STARTED (cond=1)
        • recibe 1547 + 1552
   → [checkProgress() tras dar cada supply + takeItems(1552) en Brunon] → STARTED (cond=2)
        • requiere: 1547,1548,1549,1550,1551 reunidos
   → [onTalk BOLTER STARTED, cond != 1] → COMPLETED
        • reward 906 + EXP 5672 + SP 446 + Adena 2466 + exitQuest(false,true)
   → [onTalk BOLTER COMPLETED] → getAlreadyCompletedMsg (sin reward)
```

- Estado de "no iniciada" = `State.CREATED`.
- Progreso visual: `cond` (1→2) + sonido `ITEMSOUND_QUEST_MIDDLE` al subir a cond2.
- No existe estado FAILED/ABANDONED como constante (`State.java` solo define CREATED/STARTED/COMPLETED); no hay abandono implementado en el script.

## 12. Server Flow (SOURCE ↔ RUNTIME)

| Aspecto | SOURCE | RUNTIME | Igualdad lógica |
|---|---|---|---|
| Constructor / registro | `super(5)`; addStartNpc/addTalkId/registerQuestItems | identico | SÍ |
| onEvent 30554-03 / 30526-02 / 30554-05 | identico | identico | SÍ |
| onTalk (todos los NPCs) | identico | identico | SÍ |
| checkProgress / giveItem / takeItems | identico | identico | SÍ |
| Reward values | 906×1, EXP 5672, SP 446, Adena 2466 | igual | SÍ |
| Reward order | 906 → exp/sp → adena | adena → exp/sp → 906 | COSMÉTICO (igual) |
| Newbie Guide | bloque con `ScriptManager.getScript(NewbieGuide)`, NRMemo, `GUIDE_MISSION=41`, mensaje condicional | `showOnScreenMsg` directo; sin script NewbieGuide | **CONFLICTO de capa** (misma finalidad) |
| Script NewbieGuide | presente en `ai/others/NewbieGuide/` | **ausente** (0 archivos en runtime) | CONFLICTO documentado |

**Conclusión:** la misión (progresión + recompensas) es **idéntica**. El único diferencial es la integración de la capa Newbie Guide (SOURCE vs RUNTIME). En RUNTIME el jugador aun recibe el on-screen msg ("Go find the Newbie Guide"); la máquina de estados NRMemo (SOURCE) no existe.

## 13. Player Gameplay Walkthrough (real)

1. **Crear** un personaje (preferiblemente Enano; nivel ≥ 2 — subir de lvl 1 a 2).
2. **Dirigirse a 30554 Bolter** (Miner) cerca de la mina entre Dwarven Town y Quarry (coords §9: 112656, -174890, -608).
3. **Aceptar Q00005**: se recibe **1547 Bolter's List** y **1552 Bolter's Smelly Socks**. Estado → STARTED cond1.
4. **Recoger los 4 supplies** (orden libre): Shari(→1550), Garita(→1548), Reed(→1551), Brunon(→1549 entregando 1552).
5. Al reunir 1547,1548,1549,1550,1551 → cond cambia a 2 (`setCond(2)`).
6. **Volver a Bolter** → recompensa automática: **Necklace of Knowledge (906) ×1**, EXP 5672, SP 446, Adena 2466; quest marcada COMPLETED (no repetible). Mensaje on-screen dirige al Newbie Guide.

## 14. Solo / Party

- **Solo**: la quest se completa íntegramente de forma individual (QuestState por jugador; `giveItems`/`takeItems` sobre `player`).
- **Party**: `NO_PARTY_MECHANICS`. El script **no** verifica miembros de party, ni impide que un compañero esté cerado, ni usa `getRandomPartyMember`. **No se afirma** que "rechace miembros de party"; simplemente no hay lógica de party.

## 15. Failure / Blocking Conditions

| Caso | Implementado | Evidencia |
|---|---|---|
| Nivel insuficiente (<2) | Sí | `30554-01.html` vs `30554-02.htm` (onTalk BOLTER CREATED) |
| Hablar sin estar en quest | Sí | `getNoQuestMsg` (retorno default en onTalk/onEvent) |
| Incompleto al volver a Bolter (cond1) | Sí | onTalk BOLTER cond1 → `30554-04.html` (recordatorio) |
| Falta 1552 ante Brunon | Sí | `onEvent("30526-02.html")` → `30526-04` si `!hasQuestItems(1552)` |
| Ya completada | Sí | `getAlreadyCompletedMsg` |
| Abandono | No implementado | no hay handler de abandono en el script |
| Repetir | Bloqueado | `exitQuest(false,true)` → COMPLETED; `getAlreadyCompletedMsg` en reintentos |

## 16. Client ↔ Server

- **SERVER AUTHORITATIVE (verificado en claro)**: texto HTML/diálogos, items, NPCs, coordenadas, rewards, lógica.
- **CLIENT PRESENTATION (no verificable)**: `questname-e.dat`, `NpcName-e.dat`, `itemname-e.dat`, `NpcString-e.dat` → **UNKNOWN_CLIENT** (cifrado "Lineage2 Ver 413" + blob binario; sin algoritmo de descifrado validado — véase `CLIENT_RESEARCH/CLIENT_ENCRYPTION.md`).
- **CROSS-VERIFIABLE por ID**: (quest 5; npc 30517-30554; items 1547-1552, 906; NpcStringId 501/502/503 + `DELIVERY_DUTY...`). La correlación `ID ↔ texto cliente` es definible, pero el **texto efectivo del cliente** no se puede leer todavía.
- Los `NpcStringId` usados por el server (`MINER_S_FAVOR` id 501/502/503; `DELIVERY_DUTY_COMPLETE_N_GO_FIND_THE_NEWBIE_GUIDE`) están en el **enum server-side** (`NpcStringId.java`); el mensaje localizado que el cliente muestra es **UNKNOWN_CLIENT**.

## 17. Dialogue (server-side)

Todo el diálogo está en el datapack server-side (en claro) → **VERIFIED**. El **texto localizado del cliente** es **UNKNOWN_CLIENT**.

| Archivo | NPC | Acción | Item ref | next |
|---|---|---|---|---|
| 30554-01 | Bolter | rechazo lvl<2 | — | — |
| 30554-02 | Bolter | exposición + bypass 30554-03 | — | botón "Say yes" |
| 30554-03 | Bolter | lista de suministros + socks | 1547,1552 | startQuest |
| 30554-04 | Bolter | recordatorio cond1 | 1547 | bypass → 30554-05 |
| 30554-05 | Bolter | repite lista | — | onEvent no-op |
| 30554-06 | Bolter | cierre agradecimiento | — | reward |
| 30517-01/02 | Shari | entrega/recuerda 1550 | Boomboom Powder | — |
| 30518-01/02 | Garita | entrega/recuerda 1548 | Mining Boots | — |
| 30520-01/02 | Reed | entrega/recuerda 1551 | Redstone Beer | — |
| 30526-01 | Brunon | rechazo → bypass socks | 1552 | "Take out Bolter's Smelly Socks" |
| 30526-02 | Brunon | entrega 1549 (consume 1552) | Miner's Pick | — |
| 30526-03 | Brunon | ya entregó pick | — | — |
| 30526-04 | Brunon | niega (sin socks) | — | — |

## 18. Evidence Matrix

| # | Claim | Evidence | Source | Status | Confidence |
|---|---|---|---|---|---|
| 1 | Quest ID = 5 | `super(5)` | Q00005_MinersFavor.java L52 (S/R) | VERIFIED | Alta |
| 2 | Name = Miner's Favor | docstring | .java L28 | VERIFIED | Alta |
| 3 | Author = malyelfik | `@author` | .java L29 | VERIFIED | Alta |
| 4 | MIN_LEVEL = 2 | `MIN_LEVEL=2` + check | .java L48, onTalk | VERIFIED | Alta |
| 5 | Start NPC 30554 | addStartNpc | .java L53/54 | VERIFIED | Alta |
| 6 | Talk NPCs 5 | addTalkId | .java L54/55 | VERIFIED | Alta |
| 7 | Shari da 1550 | giveItem(BOOMBOOM_POWDER) | .java L158; XML 1550 | VERIFIED | Alta |
| 8 | Garita da 1548 | giveItem(MINING_BOOTS) | .java L163; XML 1548 | VERIFIED | Alta |
| 9 | Reed da 1551 | giveItem(REDSTONE_BEER) | .java L153; XML 1551 | VERIFIED | Alta |
| 10 | Brunon 1549 key 1552 | onEvent 30526-02: takeItems(1552)+giveItems(1549) | .java L83-91 | VERIFIED | Alta |
| 11 | 1547 = Bolter's List | XML nombre + da BOLTER | .java L73; XML 1547 | VERIFIED | Alta |
| 12 | 1550 = Boomboom Powder | XML nombre + da SHARI | .java L158; XML 1550 | VERIFIED | Alta |
| 13 | 906 = Necklace (x1); qty = 1 (no es qty 906) | giveItems(NECKLACE,1); NECKLACE=906 | .java L133/127/50 | VERIFIED | Alta |
| 14 | 7 IDs (1547-1552 + 906), NO 907 | XML + registerQuestItems | .java L55; XML items | VERIFIED | Alta |
| 15 | EXP 5672; SP 446; Adena 2466 | addExpAndSp(5672,446); giveAdena(2466) | .java L134-135 / L126-127 | VERIFIED | Alta |
| 16 | No mobs / no combate | no addKillId/onKill | Q00005_MinersFavor.java | VERIFIED | Alta |
| 17 | checkProgress pide 5 (1547..1551), excluye 1552 | hasQuestItems(...) | .java L172/199 | VERIFIED | Alta |
| 18 | State CREATED/STARTED/COMPLETED | startQuest/setCond/exitQuest | .java + State.java | VERIFIED | Alta |
| 19 | No repeatable | exitQuest(false,true) | .java L136/128 | VERIFIED | Alta |
| 20 | Spawns en HighFiveSpawns.xml | coordenadas de los 5 | spawns/HighFiveSpawns.xml | VERIFIED | Alta |
| 21 | Región 921 dwarf_town + Quarry | mapregion/dwarf_town.xml | XML mapregion | VERIFIED | Alta |
| 22 | "Strip Mine" no zona formal | 0 hits zones/mapa | zones/*.xml, mapregion | VERIFIED | Media |
| 23 | Texto cliente | cifrado, no descifrado | CLIENT_ENCRYPTION.md | UNKNOWN_CLIENT | Baja |
| 24 | Reward order S≠R (cosmético) | 906→exp→adena vs adena→exp→906 | .java S L133-135 / R L125-127 | PARTIALLY_VERIFIED | Alta |
| 25 | Newbie Guide conflict S↔R | presente S, ausente R | .java + filesystem | VERIFIED | Alta |
| 26 | Party: NO_PARTY_MECHANICS | sin código party | Q00005_MinersFavor.java | VERIFIED | Alta |
| 27 | Gameplay jugado | no ejecutado todavía | — | PARTIALLY_VERIFIED | Media |

## 19. Known Gaps

- **Texto cliente localizado** (nombres en UI, tooltips, on-screen msgs): `UNKNOWN_CLIENT` (cifrado no resuelto).
- **Gameplay ejecutado por un jugador**: todavía `PARTIALLY_VERIFIED`; pendiente el FIRST PLAYABLE TEST.
- **"Strip Mine" como nombre de zona cliente** (`huntingzone-e.dat`/`zonename-e.dat`): `UNKNOWN_CLIENT`.
- **Confirmación en runtime en vivo** del mensaje on-screen final.

## 20. Vertical Slice Verdict

### Veredicto final: `VERTICAL_SLICE_PARTIALLY_VERIFIED`

- **Todo lo server-side** (identity, 5 NPCs, 6 quest items, reward 906×1, EXP 5672/SP 446/Adena 2466, state machine, condiciones, objetivos, spawns, región 921) está **VERIFIED**.
- **`906` es item ID (cantidad 1).** Se rechaza leer 906 como cantidad ni como un total de ítems.
- **No hay mobs, no hay combate, no hay party mechanics**, y la quest no es repeatable.
- El único conflicto real es la capa Newbie Guide (SOURCE vs RUNTIME); la misión es idéntica.
- El **cliente** es `UNKNOWN_CLIENT` (cifrado no resuelto) y el **gameplay real** aún no ejecutado ⇒ `PARTIALLY_VERIFIED`.

Este documento **sobrescribe y corrige explícitamente** las anomalías citadas: (i) leer 906 como cantidad de unidades — falso, 906 es item ID y la qty es 1; (ii) afirmar un total de 907 ítems — falso, hay **7 IDs distintos** (1547,1548,1549,1550,1551,1552,906); (iii) atribuir el nombre "Boomboom Powder" al ID 1547 — falso, **1547 = Bolter's List** y el Boomboom Powder es el ID 1550; (iv) interpretar 906 como cantidad — falso, es item ID.

## 21. FIRST PLAYABLE TEST (preparado — sin tocar server/client)

1. Iniciar servidor `L2JMobius_CT_2.6_HighFive`.
2. Login; crear personaje nivel ≥ 2 (p. ej. Enano, subir a lvl 2).
3. Localizar **30554 Bolter** (Miner) en la mina (coords §9: 112656, -174890, -608).
4. Aceptar Q00005 → verificar recepción de **1547 List** + **1552 Smelly Socks**, estado STARTED cond1.
5. Visitar Shari(1550), Garita(1548), Reed(1551), Brunon(1549 vía 1552) — verificar cond2.
6. Hablar a Bolter → verificar reward: **Necklace 906 ×1**, EXP 5672, SP 446, Adena 2466; COMPLETED.
7. Intentar repetir → confirmar `getAlreadyCompletedMsg` (sin reward).
8. Registrar: estado, condiciones, inventario y — si es posible leer UI — observar nombres cliente (será `UNKNOWN_CLIENT` si está cifrado).

### Expected vs Actual (matriz a rellenar en gameplay)

| Gameplay Event | KB Expected | Actual Game | Match |
|---|---|---|---|
| Start a nivel<2 | 30554-01 (rechazo) | — | ⬜ |
| Iniciar quest | da 1547 + 1552, STARTED cond1 | — | ⬜ |
| Shari/Garita/Reed | +items | — | ⬜ |
| Brunon sin 1552 | 30526-04 (rechazo) | — | ⬜ |
| Volver al recoger todo | cond1 (recordatorio) | — | ⬜ |
| Volver al cumplir | reward 906 + EXP/SP/Adena + COMPLETED | — | ⬜ |
| Repetir | already-completed msg | — | ⬜ |
| Texto cliente (nombres/UI) | esperado "Miner's Favor", "Necklace of Knowledge"… | — | ⬜ → UNKNOWN_CLIENT |






