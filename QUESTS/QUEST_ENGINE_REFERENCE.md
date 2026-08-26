# QUEST ENGINE REFERENCE

**Proyecto**: L2J Mobius CT 2.6 HighFive
**Capa**: Quests — referencia central de APIs del engine (mapa semántico, no copia de código)
**Source of Truth**: `java/org/l2jmobius/gameserver/mechanics/script/Quest.java`, `QuestState.java`, `State.java` (SOURCE baseline `e2518ab`)
**Verified**: 2026-08-26
**Status**: VERIFIED (firmas y semántica contra SOURCE) — líneas = pistas de navegación

> ⚠️ **Method/class identity is authoritative; line numbers are navigation hints and may drift with upstream commits.**
> Usar siempre identidad `Quest#methodName` / `QuestState#methodName`. La línea se da solo como ayuda (baseline `e2518ab`, 2026-08-26).

---

## 1. Quest Lifecycle (registro/carga)

| API | Clase | Propósito | Fuente aprox. |
|---|---|---|---|
| `Quest(int questId)` | Quest | Captura `_scriptFile`, `initializeAnnotationListeners()`, registra en ScriptManager (`addQuest` si id>0, `addScript` si no) y llama `onLoad()`. | L217–232 |
| `ScriptManager#addQuest/addScript/getScript` | managers/ScriptManager.java | Registro y lookup por nombre simple. | — |
| `ScriptEngine#load/processDirectory` | scripting/ScriptEngine.java | Carga recursiva de `.java` del datapack al arranque (gate `DevelopmentConfig.NO_QUESTS`). | L191 |

Detalle: [QUEST_LIFECYCLE.md](QUEST_LIFECYCLE.md) · [QUEST_ARCHITECTURE.md](QUEST_ARCHITECTURE.md)

## 2. State (constantes + acceso)

`State.java`: `public static final byte CREATED = 0; STARTED = 1; COMPLETED = 2;`

| QuestState#API | Semántica |
|---|---|
| `getState()` → byte · `isCreated()/isStarted()/isCompleted()` | Lectura de estado (L114–141). |
| `setState(byte)` / `setState(byte, boolean saveInDb)` | Cambio manual de estado (L151/L162). |

Detalle: [QUEST_STATES_VARIABLES.md](QUEST_STATES_VARIABLES.md).

## 3. Variables / cond / memo

| QuestState#API | Semántica |
|---|---|
| `set(String,int)` / `set(String,String)` / `setInternal(...)` | Variables genéricas (`setInternal` sin persistir/notificar). L191/218/237. |
| `get(String)` · `getInt(String)` · `isSet(String)` · `unset(String)` | Lectura/borrado. L410/424/496/387. |
| `isCond(int)` · `setCond(int)` · `setCond(int, boolean playQuestMiddle)` · `getCond()` | Variable especial `"cond"`; fase dentro de STARTED. `playQuestMiddle=true` reproduce `ITEMSOUND_QUEST_MIDDLE`. L456–508. |
| `setMemoState(int)/getMemoState/isMemoState` · `setMemoStateEx(slot,value)/getMemoStateEx/isMemoStateEx` | Slots numéricos estilo bitmap (quests modernas). L522–576. |

## 4. Event callbacks (sobrescribir en la quest)

Catálogo completo: [QUEST_EVENTS.md](QUEST_EVENTS.md). Los más comunes:

| Callback | Firma resumida | Nota |
|---|---|---|
| `onTalk(Npc, Player)` | → String htmltext | Interacción NPC. Default: `getNoQuestMsg`. |
| `onEvent(String event, Npc, Player)` | → String htmltext | Bypass HTML; el String devuelto es el HTML a mostrar. |
| `onKill(Npc, Player, boolean isSummon)` | → void (SOURCE) o String (RUNTIME) | Despachado vía EventDispatcher con retardo `_onKillDelay=2500ms` (Attackable L125/L328). |
| `onFirstTalk`, `onSpawn`, `onAttack`, `onAggroRangeEnter`, `onDeath`, … | ver catálogo | ~40 callbacks documentados. |

## 5. Event registration (helpers públicos)

| Helper | Registra | Fuente aprox. |
|---|---|---|
| `addStartNpc(int...)` | NPCs que inician la quest | L1462 |
| `addTalkId(int...)` | NPCs con diálogo | L1588 |
| `addKillId(int...)` | kills → `onKill` (ON_ATTACKABLE_KILL) | L1570 |
| `addAttackId(int...)` | ataques a NPC | L1552 |
| `registerQuestItems(int...)` | quest items (limpieza automática en exit) | L2347 |

---

## 6. Party helpers (crédito compartido/pesado)

Referencia completa con taxonomía: [QUEST_PARTY_CREDIT.md](QUEST_PARTY_CREDIT.md).

| Quest#API | Semántica | Fuente aprox. |
|---|---|---|
| `getRandomPartyMember(Player player, int cond)` | Delega en `(player,"cond",String.valueOf(cond))` — match exacto de variable. | L1979 |
| `getRandomPartyMember(Player player, String var, String value)` | Núcleo: sin party evalúa al propio jugador; con party reúne candidatos (match exacto, radio `ALT_PARTY_RANGE`, mismo instanceId) y elige uniforme. | L1995–2057 (filtro L2043) |
| `getRandomPartyMember(Player player, Npc npc)` | Miembro aleatorio cercano al NPC (solo distancia). | L2135 |
| `getRandomPartyMemberState(Player player, byte state)` | Miembro aleatorio con ese estado de quest. | L2067 |
| `getRandomPartyMemberState(Player player, int condition, int playerChance, Npc target)` | Ponderada: killer entra `playerChance` veces (tickets); condición exacta (`-1` = cualquier iniciada); distancia al target. | L2190–2235 |
| `checkPartyMemberConditions(QuestState,int,Npc)` (private) | Filtro de condición reutilizable. | L2235 |
| `checkDistanceToTarget(...)` (private) | Filtro distancia. | L2200 |

Config: `PlayerConfig.ALT_PARTY_RANGE` default 1500 (`AltPartyRange`), configurado 1500 en `Player.ini` (SOURCE) / `Character.ini` (RUNTIME).

## 7. Item helpers

Referencia extendida: [QUEST_REWARDS.md](QUEST_REWARDS.md).

| Quest#API | Semántica | Fuente aprox. |
|---|---|---|
| `hasQuestItems(Player, int itemId)` | true si existe ese ID en inventario. | L4483 |
| `hasQuestItems(Player, int... itemIds)` | **AND**: true solo si posee TODOS. | L4494 |
| `getQuestItemsCount(Player, int itemId)` | Cantidad actual. | L4379 |
| `getRegisteredItemIds()` | IDs pasados a `registerQuestItems(...)`. | L2338 |
| `giveItems(Player, int itemId, long count)` (+ overloads enchant/attribute) | Entrega directa. | L4782/4803/4846 |
| `giveItemRandomly(Player, Npc, int itemId, long minAmount, long maxAmount, long limit, double dropChance, boolean playSound)` | Drop probabilístico con límite; reproduce MIDDLE al alcanzar límite. | L4936 (overloads L4901/4918) |
| `takeItems(Player, int itemId, long amount)` / `(Player, long amount, int... itemIds)` | Retirada (-1 = todos). | L5011/5107 |
| Familia RUNTIME (uso-verificada): `hasItem(Player, ItemHolder)` · `hasAllItems(...)` · `takeItem(...)` · `takeAllItems(...)` | Trabajan sobre ItemHolder(ID+count). Firma exacta en core runtime: REQUIRES CODE VERIFICATION. | datapack scripts |

## 8. Reward helpers

| Quest#API | Semántica | Fuente aprox. |
|---|---|---|
| `giveAdena(Player, long count, boolean applyRates)` | applyRates=true → `rewardItems(ADENA_ID)`; false → giveItems directo. | L4634 |
| `addExpAndSp(Player, long exp, int sp)` | Aplica `EXPSP_RATE × RATE_QUEST_REWARD_XP`. | L5147 |
| `playSound(Player, QuestSound/String)` | Sonidos de quest. | L5126/5136 |

## 9. Quest completion (QuestState)

| QuestState#API | Semántica | Fuente aprox. |
|---|---|---|
| `startQuest()` | CREATED→STARTED (cond=1, marca DB). | L615 |
| `exitQuest(QuestType type[, boolean playExitQuest])` | Salida tipada. | L633/663 |
| `exitQuest(boolean repeatable)` | Limpia quest items registrados (`removeRegisteredQuestItems`) + `deleteQuestInDb(this, repeatable)`; repeatable→elimina estado (re-hacible); !repeatable→`setState(COMPLETED)`. `_vars=null`. | L680–704 |
| `exitQuest(boolean repeatable, boolean playExitQuest)` | Ídem + sonido finish. | L715 |
| `showQuestionMark(int number)` | TutorialShowQuestionMark. | L724 |
| `setRestartTime()` / `isNowAvailable()` | Disponibilidad futura (daily quests). | L732/749 |
| `addNotifyOfDeath(Creature)` · `setIsExitQuestOnCleanUp(boolean)` / `isExitQuestOnCleanUp()` | Notificación de muerte / limpieza al salir. | L585/598–606 |

---

## 10. Timers

Referencia completa: [QUEST_TIMERS.md](QUEST_TIMERS.md).
- `Quest#startQuestTimer(String name, long time, Npc npc, Player player[, boolean repeating])` — L448/471.

## 11. HTML / result routing

| Quest#API | Semántica | Fuente aprox. |
|---|---|---|
| `showResult(Player, String res)` / `(Player, String res, Npc npc)` | Enruta el resultado (HTML/evento) hacia el jugador tras onTalk/onEvent/onFirstTalk. | L1190 (usos L631–702) |
| `getNoQuestMsg(Player)` · `getAlreadyCompletedMsg(Player)` | Mensajes estándar de retorno. | L1442/1452 |

Detalle de interacción Player↔NPC↔HTML: [QUEST_PLAYER_NPC_DIALOG.md](QUEST_PLAYER_NPC_DIALOG.md).

## 12. Spawn helpers

Overloads verificados (mayoría static):

| Quest#API | Fuente aprox. |
|---|---|
| `addSpawn(int npcId, IPositionable pos)` | L4082 |
| `Npc addSpawn(int npcId, Location pos, int instanceId)` | L4096 |
| `addSpawn(Npc summoner, int npcId, IPositionable pos, boolean randomOffset, long despawnDelay)` | L4110 |
| `addSpawn(int npcId, IPositionable pos[, randomOffset][, despawnDelay][, isSummonSpawn][, instanceId])` | L4124–4175 |
| `addSpawn(int npcId, int x, int y, int z, int heading[, randomOffset][, despawnDelay][, isSummonSpawn][, instanceId])` | L4193–4271 |

> Spawns persistentes del mundo → `data/spawns/*.xml` (ver [../WORLD/SPAWN_QUERY_GUIDE.md](../WORLD/SPAWN_QUERY_GUIDE.md)). `addSpawn` = spawns dinámicos desde script.

## 13. Door helpers

| Quest#API | Fuente aprox. |
|---|---|
| `openDoor(int doorId, int instanceId)` | L5334 |
| `closeDoor(int doorId, int instanceId)` | L5352 |

## 14. Player iteration / AI desires

| Quest#API | Semántica | Fuente aprox. |
|---|---|---|
| `executeForEachPlayer(Player, Npc, boolean isSummon, boolean includeParty, boolean includeCommandChannel)` | Itera jugador/party/command channel ejecutando una acción. | L5290 |
| `addAttackDesire(Npc npc, Creature target[, int desire])` | Deseo de ataque a NPC controlado (`addDamageHate`). | L5422/5433–5437 |
| `addMoveToDesire(Npc npc, Location loc, int desire)` | Deseo de movimiento. | L5450 |
| Cast desire (`addSkillCastDesire`) | Helper de AI-scripting; firma exacta REQUIRES CODE VERIFICATION (no localizada con nombre literal en Quest.java SOURCE). | — |

> Los helpers de "desire" son para scripts AI (paquete Script/Event), no para quests normales.

## 15. Special / retail APIs

| Quest#API | Semántica | Fuente aprox. |
|---|---|---|
| `playMovie(Player, Movie)` / `(Collection<Player>, …)` / `(Set<Player>, …)` / `(InstanceWorld, Movie)` | Reproducción de cinemáticas. | L5604–5634 |
| `specialCamera(...)` | No localizado en Quest.java SOURCE con este nombre (posible en scripts de instancias/AI). REQUIRES CODE VERIFICATION. | — |

## Cross-links

- Estados/variables/memo/persistencia → [QUEST_STATES_VARIABLES.md](QUEST_STATES_VARIABLES.md)
- Catálogo completo de callbacks y despacho → [QUEST_EVENTS.md](QUEST_EVENTS.md)
- Recompensas e items → [QUEST_REWARDS.md](QUEST_REWARDS.md)
- Crédito de party → [QUEST_PARTY_CREDIT.md](QUEST_PARTY_CREDIT.md)
- Timers → [QUEST_TIMERS.md](QUEST_TIMERS.md)
- Método de investigación → [QUEST_RESEARCH_FRAMEWORK.md](QUEST_RESEARCH_FRAMEWORK.md)
- Plantilla de slice → [QUEST_VERTICAL_SLICE_TEMPLATE.md](QUEST_VERTICAL_SLICE_TEMPLATE.md)

---
**Fuente**: `mechanics/script/{Quest,QuestState,State}.java` (SOURCE `e2518ab`) · líneas verificadas 2026-08-26.
**Status**: VERIFIED (firmas/semántica) · líneas = navegación.



---
