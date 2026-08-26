# Q00003 — Will the Seal be Broken?

## 1. Identity

| Field | Value | Status | Confidence |
|---|---|---|---|
| Quest ID | **3** (`super(3)` en constructor) | SERVER_VERIFIED | Alta |
| Quest name | Will the Seal be Broken? (docstring `/** Will the Seal be Broken? (3) */`) | SERVER_VERIFIED | Alta |
| Java class | `quests.Q00003_WillTheSealBeBroken.Q00003_WillTheSealBeBroken extends Quest` | SERVER_VERIFIED | Alta |
| Author | malyelfik (`@author`) | SERVER_VERIFIED | Alta |
| Minimum level | **16** (`MIN_LEVEL = 16`; chequeo en `onTalk` CREATED) | SERVER_VERIFIED | Alta |
| Race lock | **DARK_ELF only** — `player.getRace() != Race.DARK_ELF ? "30141-00.htm" : ...` | SERVER_VERIFIED | Alta |
| Repeatable | **NO** — `qs.exitQuest(false, true)` | SERVER_VERIFIED | Alta |
| Start NPC | 30141 (Talloth, Tetrarch) — `addStartNpc(TALLOTH)` | SERVER_VERIFIED | Alta |
| Talk NPCs | solo 30141 — `addTalkId(TALLOTH)` | SERVER_VERIFIED | Alta |
| Completion NPC | 30141 (Talloth) | SERVER_VERIFIED | Alta |
| Kill mobs | 6 IDs registrados con `addKillId(...)` | SERVER_VERIFIED | Alta |

Fuente: `Q00003_WillTheSealBeBroken.java` (SOURCE y RUNTIME, lógica idéntica).

## 2. Authority & Sources

Jerarquía aplicada: `SOURCE > RUNTIME > XML > CLIENT > KB`.

| Capa | Ruta | Contenido | Estado |
|---|---|---|---|
| SERVER_SOURCE | `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive/dist/game/data/scripts/quests/Q00003_WillTheSealBeBroken/` | Java + 7 HTML/HTM | VERIFIED |
| SERVER_RUNTIME | `L2J_Mobius_CT_2.6_HighFive/game/data/scripts/quests/Q00003_WillTheSealBeBroken/` | Java + 7 HTML/HTM | VERIFIED |
| SERVER XML npcs (mobs) | `game/data/stats/npcs/20000-20099.xml` | defs 20031/20041/20046/20048/20052/20057 | VERIFIED |
| SERVER XML npc (start) | `game/data/stats/npcs/30100-30199.xml` L1502 | def 30141 Talloth | VERIFIED |
| SERVER XML items | `stats/items/01000-01099.xml`, `00900-00999.xml` | 1081/1082/1083 y reward 956 | VERIFIED |
| SERVER spawns RUNTIME | `game/data/spawns/HighFiveSpawns.xml` (archivo único consolidado) | spawns de mobs y NPC | VERIFIED |
| SERVER spawns SOURCE | `dist/game/data/spawns/Others/18_19.xml` (mobs) · `Others/20_18.xml` (Talloth) | mismos spawns, layout por celda | VERIFIED |
| SERVER mapregion | `mapregion/darkelf_town.xml` | región `darkelf_town` locId **915** + `Temple_of_Shilen` (915) | VERIFIED |
| Quest core API | `mechanics/script/Quest.java` (SOURCE) | getRandomPartyMember L1979/L1995-2057 · hasQuestItems L4483/L4494 · getRegisteredItemIds L2338 | VERIFIED |
| Config party range | SOURCE `dist/game/config/Player.ini` · RUNTIME `game/config/Character.ini` | `AltPartyRange = 1500` (default 1500 en `PlayerConfig.java` L490) | VERIFIED |
| CLIENT | `Lineage2-TCT-273-client/system/*.dat`, `L2text/` | questname/NpcName/itemname-e.dat | **UNKNOWN_CLIENT** (cifrado) |

## 3. NPC Chain

Un único NPC interactivo. Cadena real verificada:

```
PLAYER (Dark Elf, nivel >= 16)
  ↓
30141 TALLOTH → onTalk(CREATED):
     raza != DARK_ELF      → 30141-00.htm (rechazo)
     raza == DE && lvl <16 → 30141-01.html (aún bajo nivel)
     raza == DE && lvl >=16→ 30141-02.htm (exposición) → bypass "30141-03.htm"
  ↓ onEvent("30141-03.htm") → qs.startQuest() → STARTED (cond 1)
  ↓ [matar mobs objetivo → drop garantizado del quest item correspondiente]
     20031 OMEN_BEAST          → giveItem(OMEN_BEAST_EYE = 1081)
     20041/20046 ZOMBIES       → giveItem(TAINT_STONE    = 1082)
     20048/20052/20057 SUCCUBI → giveItem(SUCCUBUS_BLOOD = 1083)
     helper giveItem(): si NO tiene ya ese item → da ×1 + sonido;
        si al tenerlo posee los 3 (hasQuestItems(all)) → setCond(2, true)
  ↓
30141 TALLOTH → onTalk(STARTED):
     cond == 1 → 30141-04.html (recordatorio) → bypass 30141-05.html (no-op)
     cond != 1 → giveItems(ENCHANT = 956, 1) + exitQuest(false, true) → COMPLETED → 30141-06.html
  ↓
COMPLETED → getAlreadyCompletedMsg (sin recompensa)
```

No hay otros NPCs registrados; las menciones a "Sentry Kayleen" y "Magister Kayla"/Giran en el HTML son LORE (no registradas en `addTalkId`).

## 4. Dialogue (server-side)

Inventario completo verificado (7 archivos, presentes en SOURCE y RUNTIME):

| Archivo | Condición previa | Contenido / acción |
|---|---|---|
| `30141-00.htm` | CREATED, raza ≠ Dark Elf | Rechazo: "no task reserved for those of foreign races". |
| `30141-01.html` | CREATED, DE, lvl < 16 | Intro lore + nota "(Quest for Dark Elven characters level 16 or above.)" |
| `30141-02.htm` | CREATED, DE, lvl ≥ 16 | Exposición (sello de Mitraell) + bypass → `30141-03.htm` (aceptar) |
| `30141-03.htm` | evento aceptación | Lista de materiales (omen beast's eye / Taint Stone / succubus' blood) y dónde conseguirlos (School of Dark Arts) |
| `30141-04.html` | STARTED cond 1 | Recordatorio + bypass → `30141-05.html` |
| `30141-05.html` | bypass recordatorio | Repite la lista (evento no-op en código) |
| `30141-06.html` | STARTED cond 2 (entrega) | Agradecimiento + entrega del token + referencia a Magister Kayla (Giran) |

SERVER_DIALOGUE = VERIFIED · CLIENT_DIALOGUE = UNKNOWN_CLIENT.

## 5. Conditions

| Condición | Implementación | Estado |
|---|---|---|
| Raza | `player.getRace() != Race.DARK_ELF` → rechazo (HTML -00) | SERVER_VERIFIED |
| Nivel mínimo | `player.getLevel() >= MIN_LEVEL(16)`; si no → HTML -01 | SERVER_VERIFIED |
| Aceptación | bypass `30141-03.htm` → `startQuest()` | SERVER_VERIFIED |
| Progreso | tener los **3** quest items (`hasQuestItems(1081,1082,1083)`) → `setCond(2,true)` | SERVER_VERIFIED |
| Finalización | `onTalk` STARTED con cond ≠ 1 | SERVER_VERIFIED |
| Abandono | no implementado en el script (sin handler) | SERVER_VERIFIED |
| Reinicio | no aplica (no repeatable, `exitQuest(false,true)`) | SERVER_VERIFIED |

Sin "difficulty tiers", sin memos, sin timers.

## 6. Objectives

1. Hablar con **30141 Talloth** (Dark Elf lvl ≥16) y aceptar.
2. Conseguir los 3 materiales matando mobs objetivo (ver §7):
   - **1081 Omen Beast's Eye** de Omen Beast (20031)
   - **1082 Taint Stone** de Tainted Zombie (20041) o Stink Zombie (20046)
   - **1083 Succubus Blood** de Lesser Succubus (20048), LS Turen (20052) o LS Tilfo (20057)
3. Al reunir los 3 → cond 2 (automático en el kill flow).
4. Volver a Talloth → recompensa **956 ×1** + COMPLETED.

El orden entre grupos de mobs es libre; solo importa poseer 1 de cada tipo.

## 7. Combat / Mobs — mob → quest-item mapping

Todos `type="Monster"` en `stats/npcs/20000-20099.xml`. Drop **garantizado de 1 unidad** por kill correspondiente (no usa `giveItemRandomly`, no hay probabilidad); no se entrega duplicado si ya se posee:

| Mob | ID | Nivel | Quest item entregado |
|---|---|---|---|
| Omen Beast | 20031 | 17 | **1081** Omen Beast's Eye |
| Tainted Zombie | 20041 | 18 | **1082** Taint Stone |
| Stink Zombie | 20046 | 19 | **1082** Taint Stone |
| Lesser Succubus | 20048 | 20 | **1083** Succubus Blood |
| Lesser Succubus Turen | 20052 | 21 | **1083** Succubus Blood |
| Lesser Succubus Tilfo | 20057 | 22 | **1083** Succubus Blood |

Semántica de items:
- 1081/1082/1083 son **quest items registrados** (`registerQuestItems(1081,1082,1083)`); se limpian al completar la quest (comportamiento engine de `exitQuest`).
- El avance a cond2 exige **poseer simultáneamente los 3** (`hasQuestItems(player, getRegisteredItemIds())`).
- Cantidad necesaria por item: **1** (el helper no vuelve a entregar si ya existe).

## 8. World / Locations

| Entidad | Valor | Estado |
|---|---|---|
| Región formal NPC | `darkelf_town` (town "Darkelven Town", castle 4, **locId 915**, bbs 3) — `mapregion/darkelf_town.xml` L5 | SERVER_VERIFIED |
| Sub-región formal | `Temple_of_Shilen` (misma town 915) — ídem L42 | SERVER_VERIFIED |
| Spawns de mobs | celda de mapa `18_19` (territorio Dark Elf de caza, coords ≈ x −50853…−45076, y 42934…50044, z −5400/−5912) | SERVER_VERIFIED |
| **"School of Dark Arts"** | **LORE/CONTEXT** — 0 ocurrencias como zona/región/grupo en `zones/*.xml`, `mapregion/*.xml` y headers `<spawn name=...>`; solo aparece en HTML (-03/-05). NO afirmar zona formal. | SERVER_VERIFIED (ausencia) |
| Nota complementaria | Sí existe como **nombre de ubicación**: punto de teleport `Center of the School of Dark Arts` (locId **9103**, coords −49185/49441/−5912, en dump SQL) y `NpcStringId` SCHOOL_OF_DARK_ARTS / ENTRANCE_TO_THE… (ids 1010488/1010489/1010564). Refuerza: nombre de lugar/lore, **no** objeto Zone del engine. | SERVER_VERIFIED |

## 9. Spawn Verification

Layout dual (ver guía [WORLD/SPAWN_QUERY_GUIDE.md](../WORLD/SPAWN_QUERY_GUIDE.md)):

| ID | Nombre | Spawns RUNTIME (`HighFiveSpawns.xml`) | Spawns SOURCE (`Others/18_19.xml` / `20_18.xml`) | Coinciden |
|---|---|---|---|---|
| 20031 | Omen Beast | 12 | 12 | ✅ |
| 20041 | Tainted Zombie | 12 | 12 | ✅ |
| 20046 | Stink Zombie | 13 | 13 | ✅ |
| 20048 | Lesser Succubus | 12 | 12 | ✅ |
| 20052 | LS Turen | 7 | 7 | ✅ |
| 20057 | LS Tilfo | 10 | 10 | ✅ |
| **30141** | **Talloth** | **1** (`x=11012 y=14128 z=-4240 heading=14500 respawn=60`) | **1** (`20_18.xml`) | ✅ |

Muestra de coordenadas de mobs (todas z −5400 salvo succubi profundos −5656…−5912): 20031 `(-50853,45399)`,`(-47391,43134)`; 20041 `(-50777,45264)`; 20046 `(-49581,45076)`; 20048 `(-49872,44748)`; 20052 `(-48003,44962)`; 20057 `(-48102,45762)`. Todos `respawnDelay=60`.

## 10. Rewards

| Tipo | Item ID | Nombre | Cantidad | Código | Estado |
|---|---|---|---|---|---|
| Item | **956** | **Scroll: Enchant Armor (D-Grade)** | **1** | `giveItems(player, ENCHANT, 1)` en `onTalk` STARTED cond≠1 | SERVER_VERIFIED |

- **NO hay EXP, SP ni Adena** en el código de la quest (sin `addExpAndSp`, sin `giveAdena`).
- No hay rewards alternativos ni condicionales.
- `exitQuest(false, true)` → COMPLETED, no repetible.
- Los quest items registrados (1081/1082/1083) se limpian al salir (comportamiento engine).

## 11. Quest State Machine

```
CREATED
   → [onEvent "30141-03.htm" → startQuest()] → STARTED (cond = 1)
   → [kill flow: poseer simultáneamente 1081+1082+1083] → setCond(2) → STARTED (cond = 2)
   → [onTalk Talloth, STARTED, cond ≠ 1]
        giveItems(956,1); exitQuest(false,true) → COMPLETED
   → [onTalk COMPLETED] → getAlreadyCompletedMsg (sin reward)
```

- Estados del engine: `State.CREATED / STARTED / COMPLETED` (+ `cond` 1→2 dentro de STARTED).
- Sin estados FAILED/ABANDONED; sin mecanismo de abandono en el script.

## 12. Server Flow (SOURCE ↔ RUNTIME)

| Aspecto | SOURCE (`dist/`) | RUNTIME (`game/`) | Veredicto |
|---|---|---|---|
| Lógica de quest (constructor, onEvent, onKill, onTalk, giveItem) | idéntica | idéntica | ✅ SIN CONFLICTO |
| Imports | `entity.actor.*`, `mechanics.script.*`, `entity.actor.enums.creature.Race` | `model.actor.*`, `model.quest.*`, `enums.Race` | Cosmético (refactor de paquetes) |
| Firma `onKill` | `public void onKill(...)` con `return;` temprano | `public String onKill(...)` retornando `super.onKill(...)` | Cosmético (firma API), misma semántica |
| HTML | 7 archivos; `-02.htm` y `-04.html` difieren 2 bytes (line-endings) | 7 archivos | Contenido funcionalmente idéntico |

## 13. Party / Credit Mechanics

Clasificación: **`PARTY_CREDIT_SHARED`** (selección uniforme aleatoria entre miembros elegibles). Detalle completo en [QUEST_PARTY_CREDIT.md](QUEST_PARTY_CREDIT.md).

- Llamada real: `member = getRandomPartyMember(player, 1)` → delega en `getRandomPartyMember(player, "cond", "1")`.
- **Match exacto** de la variable `cond == "1"` (no es mínimo ni rango).
- Sin party: se evalúa al propio jugador (su cond debe ser "1").
- Con party: candidatos = **todos los miembros** con `cond=="1"` que estén dentro de `ALT_PARTY_RANGE = 1500` respecto al target/player y en el **mismo instanceId**; se elige **uno al azar uniformemente** (`candidates.get(Rnd.get(size))`).
- El **killer es un candidato más** (sin peso extra); un **miembro que no hizo el kill puede recibir el item**.
- Si nadie cumple (p.ej. todos en cond2) → no se entrega nada.
- Referencias core: `Quest.java` L1979 (delegación), L1995–2057 (lógica), L2043 (filtro distancia+instance).

## 14. Player Gameplay Walkthrough

1. Crear personaje **Dark Elf** y subir a nivel ≥ 16.
2. Ir al pueblo Dark Elf (región 915) y hablar con **30141 Tetrarch Talloth** `(11012, 14128, -4240)`.
3. Aceptar la misión (bypass en `30141-02.htm`) → STARTED cond1.
4. Ir a la zona de caza Dark Elf (celda `18_19`, la que el lore llama "School of Dark Arts") y farmear:
   - Omen Beast → **Omen Beast's Eye** (1081)
   - Tainted/Stink Zombie → **Taint Stone** (1082)
   - Succubus (cualquiera de las 3) → **Succubus Blood** (1083)
5. Al obtener el tercero → sonido de progreso y cond 2.
6. Volver a Talloth → recibir **Scroll: Enchant Armor (D-Grade)** ×1; quest COMPLETED (no repetible).

## 15. Solo / Party Behavior

- **Solo**: funciona igual; sin party, `getRandomPartyMember` valida al propio jugador.
- **Party**: `PARTY_CREDIT_SHARED` — cada kill elegible entrega el item a un miembro aleatorio con cond1 en rango ≤1500 y mismo instance; puede ir al killer o a otro. Ver §13.

## 16. Failure / Blocking Conditions

| Caso | Comportamiento | Evidencia |
|---|---|---|
| Raza no Dark Elf | HTML `-00.htm`, sin opción de aceptar | onTalk CREATED |
| Nivel < 16 (siendo DE) | HTML `-01.html` | ídem |
| Hablar sin quest / estado raro | `getNoQuestMsg` | default onTalk |
| Kill con todos en cond2 (party) | nadie recibe item (sin candidatos cond1) | getRandomPartyMember → null |
| Volver con cond1 (faltan items) | HTML `-04.html` recordatorio | onTalk STARTED isCond(1) |
| Ya completada | `getAlreadyCompletedMsg`, sin reward | onTalk COMPLETED |
| Abandono | no implementado | sin handler |

## 17. Client ↔ Server

- **SERVER_AUTHORITATIVE**: lógica, IDs, diálogos HTML (en claro), spawns, rewards.
- **CLIENT_PRESENTATION**: `questname-e.dat` / `NpcName-e.dat` / `itemname-e.dat` / `NpcString-e.dat` → **UNKNOWN_CLIENT** (cifrado "Lineage2 Ver 413"; ver [CLIENT_ENCRYPTION](../CLIENT_RESEARCH/CLIENT_ENCRYPTION.md)).
- **CROSS-VERIFICABLE por ID**: quest 3 · NPC 30141 · mobs 20031-20057 · items 1081/1082/1083/956.

## 18. Evidence Matrix

| # | Claim | Evidence | Source | Status |
|---|---|---|---|---|
| 1 | Quest ID = 3 | `super(3)` | .java L55/S · L52/R | SERVER_VERIFIED |
| 2 | MIN_LEVEL = 16 | constante + ternary onTalk | .java L51/L135 (S) | SERVER_VERIFIED |
| 3 | Race lock DARK_ELF | `getRace() != Race.DARK_ELF` | .java L135/S · L131/R | SERVER_VERIFIED |
| 4 | No repeatable | `exitQuest(false,true)` | .java L147/S · L143/R | SERVER_VERIFIED |
| 5 | NPC único 30141 Talloth lvl70 Folk "Tetrarch" | addStartNpc/addTalkId + XML | .java L56-57 · npcs XML L1502 | SERVER_VERIFIED |
| 6 | 6 mobs con addKillId | lista explícita | .java L58/S | SERVER_VERIFIED |
| 7 | Mapping mob→item (1081/1082/1083) | switch en onKill | .java L103-123/S | SERVER_VERIFIED |
| 8 | Drop garantido ×1, sin duplicado | helper giveItem (!hasQuestItems) | .java L162-173/S | SERVER_VERIFIED |
| 9 | cond2 al tener los 3 | hasQuestItems(getRegisteredItemIds()) | .java L168-171/S · Quest.java L4494 | SERVER_VERIFIED |
| 10 | Reward 956 ×1, sin EXP/SP/Adena | giveItems(ENCHANT,1) único reward | .java L146/S · L142/R | SERVER_VERIFIED |
| 11 | PARTY_CREDIT_SHARED uniforme | getRandomPartyMember(player,1) | Quest.java L1979/L1995-2057 | SERVER_VERIFIED |
| 12 | ALT_PARTY_RANGE = 1500 | PlayerConfig L490 + Player.ini/Character.ini | config | SERVER_VERIFIED |
| 13 | Mismo instance + radio 1500 | isInsideRadius3D(target, ALT_PARTY_RANGE) | Quest.java L2043 | SERVER_VERIFIED |
| 14 | Spawns mobs: 12/12/13/12/7/10 | conteo por ID | HighFiveSpawns.xml == Others/18_19.xml | SERVER_VERIFIED |
| 15 | Talloth: 1 spawn | conteo por ID | HighFiveSpawns.xml L6164 == Others/20_18.xml | SERVER_VERIFIED |
| 16 | Región formal darkelf_town locId 915 | mapregion XML | darkelf_town.xml L5 | SERVER_VERIFIED |
| 17 | School of Dark Arts = LORE/CONTEXT | 0 hits zones/mapregion/spawn-names | búsqueda exhaustiva | SERVER_VERIFIED (ausencia) |
| 18 | Diálogo server completo (7 HTML) | inventario + lectura | quests dir S/R | SERVER_VERIFIED |
| 19 | Texto localizado cliente | cifrado sin descifrar | CLIENT_ENCRYPTION.md | UNKNOWN_CLIENT |
| 20 | Gameplay real ejecutado | pendiente first playable test | — | PARTIALLY_VERIFIED |

## 19. Known Gaps

- **UNKNOWN_CLIENT**: nombres localizados de quest/NPC/mobs/items en la UI del cliente (cifrado no resuelto).
- **GAMEPLAY_UNVERIFIED**: el flujo no ha sido jugado todavía (first playable test pendiente); todo lo server-side está verificado por código/XML.
- `PARTY_CREDIT_SHARED` documentado desde código; **no observado en ejecución**.
- No se han auditado otros usos de los mobs 20031–20057 en otras quests/AI (fuera de scope).

## 20. Vertical Slice Verdict

### Veredicto final: `VERTICAL_SLICE_PARTIALLY_VERIFIED`

- Todo lo **server-side** (identidad, race-lock, NPC único, 6 mobs + mapping, drop garantizado, party-credit compartido, spawns duales SOURCE/RUNTIME, región 915, reward 956 ×1) está **SERVER_VERIFIED**.
- **CLIENT = UNKNOWN_CLIENT**; **GAMEPLAY = PARTIALLY_VERIFIED** (no ejecutado).
- Corrige respecto al discovery previo: el crédito de kill **NO es individual** → es **PARTY_CREDIT_SHARED** uniforme.
- "School of Dark Arts" queda registrada como LORE/CONTEXT, nunca como zona formal.

## 21. First Playable Test

1. Iniciar servidor RUNTIME y login.
2. Crear **Dark Elf** y subir a nivel 16+.
3. Hablar con **Talloth** (30141) en el pueblo Dark Elf → aceptar (cond1, sin items aún).
4. Farmear en la celda `18_19`: 1× Omen Beast (1081), 1× Zombie (1082), 1× Succubus (1083). Registrar qué item cae en cada kill.
5. Al tener los 3 → verificar cambio a cond2 (sonido/marcador).
6. Volver a Talloth → verificar entrega de **956 ×1** y estado COMPLETED.
7. Reintentar la quest → confirmar mensaje de ya completada.
8. (Opcional party) 2 jugadores DE cond1 juntos: matar un mob y registrar **quién** recibe el item — validar PARTY_CREDIT_SHARED.

### Expected vs Actual

| Gameplay Event | KB Expected | Actual Game | Match |
|---|---|---|---|
| Hablar siendo otra raza | HTML -00 rechazo | — | ⬜ |
| DE lvl<16 | HTML -01 | — | ⬜ |
| Aceptar | STARTED cond1 | — | ⬜ |
| Kill Omen Beast | +1081 (una vez) | — | ⬜ |
| Kill Zombie | +1082 | — | ⬜ |
| Kill Succubus | +1083 → cond2 | — | ⬜ |
| Entrega | 956 ×1 + COMPLETED | — | ⬜ |
| Repetir | already-completed msg | — | ⬜ |
| Party kill | item a miembro aleatorio cond1 | — | ⬜ |
| Nombres en UI cliente | UNKNOWN_CLIENT | — | ⬜ |




