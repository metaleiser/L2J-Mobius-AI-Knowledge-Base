# QUEST REWARDS

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Quests — recompensas  
**Source of Truth**: `mechanics/script/Quest.java` (helpers estáticos/protected L4634–5140)  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## 1. ITEMS

### Entregar
```java
public static void giveItems(Player player, int itemId, long count)                       // L4782
public static void giveItems(Player player, ItemHolder holder)                            // L4792
public static void giveItems(Player player, int itemId, long count, int enchantlevel)     // L4803
public static void giveItems(Player player, int itemId, long count, byte attributeId, int attributeLevel) // L4846
```

### Entrega aleatoria (drop de quest)
```java
public static boolean giveItemRandomly(Player player, Npc npc, int itemId,
        long minAmount, long maxAmount, long limit, double dropChance, boolean playSound) // L4936
```
- Si alcanza `limit` → reproduce `ITEMSOUND_QUEST_MIDDLE`; si no, `ITEMSOUND_QUEST_ITEMGET`.

### Quitar
```java
public static boolean takeItems(Player player, int itemId, long amount)                   // L5011
public static boolean takeItems(Player player, int amount, int... itemIds)                // L5107 (amount -1 = todos)
```
- En `exitQuest` los quest items registrados se limpian: `takeItems(player, -1, _questItemIds)`.

### Consultar inventario de quest items
```java
public static boolean hasQuestItems(Player player, int itemId)                            // L4483 — existe ese ID
public static boolean hasQuestItems(Player player, int... itemIds)                        // L4494 — AND: TODOS deben existir
public static long getQuestItemsCount(Player player, int itemId)                          // L4379 — cantidad actual
public int[] getRegisteredItemIds()                                                       // L2338 — IDs pasados a registerQuestItems(...)
```
- `hasQuestItems(player, a, b, c)` es un **AND**: true solo si posee **todos** (patrón de colección completa; usado por Q00003 para cond2).
- `getRegisteredItemIds()` permite reutilizar la lista registrada sin repetir literales (usado por Q00003 en su helper `giveItem`).
- Familia usada por scripts RUNTIME (API nueva, verificada por uso en datapack; firma exacta REQUIRES CODE VERIFICATION en core runtime): `hasItem(player, ItemHolder)` · `hasAllItems(player, onlyThisItems?, ItemHolder...)` · `takeItem(player, ItemHolder)` · `takeAllItems(player, ItemHolder...)`. Semántica: trabajan sobre `ItemHolder` (ID+count) y respetan cantidad mínima.
- Distinción clave: **quest-item** (registrado, no comerciável, se limpia al salir) vs **reward item** (normal, permanece). No mezclar ID de item con cantidad.

### Adena
```java
public static void giveAdena(Player player, long count, boolean applyRates)               // L4634
// applyRates=true → rewardItems(ADENA_ID) ; false → giveItems directo
```

## 2. EXP / SP

```java
public static void addExpAndSp(Player player, long exp, int sp)                            // L5147
```
- Aplica: `calcStat(Stat.EXPSP_RATE, addExp * RatesConfig.RATE_QUEST_REWARD_XP, ...)`.
- Es decir: **rate de quest × EXPSP_RATE del jugador**, luego `player.addExpAndSp(...)`.

## 3. SONIDOS DE QUEST

```java
public static void playSound(Player player, String sound)          // L5126
public static void playSound(Player player, QuestSound sound)      // L5136
```
Enum `QuestSound`: p.ej. `ITEMSOUND_QUEST_MIDDLE`, `ITEMSOUND_QUEST_ITEMGET`, sonidos de accept/finish/giveup (valores completos en el enum).

## 4. MENSAJES ESTÁNDAR

```java
public static String getNoQuestMsg(Player player)           // L1442
public static String getAlreadyCompletedMsg(Player player)  // L1452
```
Usados como retorno por defecto de `onTalk/onFirstTalk`.

## 5. RECOMPENSAS NO-ÍTEM (verificadas como conexiones)

| Recompensa | Mecanismo real |
|------------|----------------|
| Skills | `player.addSkill(...)` (helper propio del script; conexión SKILL_LEARNING) |
| Reputación clan | vía Clan API desde script (conexión CLAN, fase futura) |
| Flags/variables de mundo | variables de QuestState u otras managers |
| Fame/karma | métodos de Player invocables desde scripts |

No existe un helper único "giveReward" que englobe todo: cada script compone su recompensa con estos helpers + APIs de Player.

## 5b. PATRONES REUTILIZABLES (generalizables)

> Distinción obligatoria: **ENGINE PATTERN** = capacidad que las APIs del engine soportan de forma general; **QUEST-SPECIFIC IMPLEMENTATION** = cómo una quest concreta decide usarla. Un patrón engine no convierte la implementación de una quest en regla universal.

| Patrón | Tipo | Mecánica (APIs) | Ejemplo verificado |
|---|---|---|---|
| Gate de no-reentrega (`!hasQuestItems`) | ENGINE PATTERN | `if (!hasQuestItems(player, ITEM)) { giveItems(ITEM,1); … }` — garantiza entrega única sin duplicados. Posible gracias a `hasQuestItems(int)` + `giveItems`. | Q00003 helper `giveItem` (`!hasQuestItems` antes de dar) |
| Advance on collection complete | ENGINE PATTERN | `if (hasQuestItems(player, allIds)) setCond(n)` — dispara transición al completar colección. `hasQuestItems(int...)` es AND. | Q00003 (`getRegisteredItemIds()`), Q00039 (`hasItem` sobre ItemHolder completo) |
| Conteo con límite → transición | ENGINE PATTERN | `getQuestItemsCount(...) >= LIMIT` → `setCond` (colección por cantidad). | Q00042/43/44 (×30), Q00050–53 (×100) |
| Drop probabilístico con límite | ENGINE PATTERN | `giveItemRandomly(player,npc,item,min,max,limit,chance,true)` retorna true al alcanzar el límite → usar como trigger de `setCond`. | Q00039 §3.3 |
| Key-item gating (intercambio) | QUEST-SPECIFIC IMPLEMENTATION | `takeItems(KEY,-1)` + `giveItems(REWARD)` tras comprobar posesión. Depende del diseño de la quest. | Q00005 (1552→1549 vía Brunon) |
| Party credit compartido / ponderado | ENGINE PATTERN | Ver [QUEST_PARTY_CREDIT.md](QUEST_PARTY_CREDIT.md): SHARED_RANDOM (`getRandomPartyMember`) vs KILLER_WEIGHTED (`getRandomPartyMemberState`). | Q00003 / Q00039 |

> Regla: documentar el patrón como capacidad del engine y citar la quest solo como ejemplo de uso; nunca elevar la implementación de una quest a norma general.


## 6. TRAZABILIDAD

| Helper | Path |
|--------|------|
| giveItems/rewardItems/giveAdena/takeItems/giveItemRandomly | `mechanics/script/Quest.java` L4634–5114 |
| hasQuestItems / getQuestItemsCount / getRegisteredItemIds | ídem L4379–4505 · L2338 |
| addExpAndSp | ídem L5147 |
| playSound | ídem L5126–5140 |
| getNoQuestMsg/getAlreadyCompletedMsg | ídem L1442–1452 |
| getRandomPartyMember*/party credit | [QUEST_PARTY_CREDIT.md](QUEST_PARTY_CREDIT.md) |

---
## Ver también

- [SYSTEMS/ITEM_SYSTEM.md](../SYSTEMS/ITEM_SYSTEM.md) — ItemTemplate vs Item, entrega y consumo
- [SKILLS/SKILL_LEARNING.md](../SKILLS/SKILL_LEARNING.md) — si la quest otorga skills

---
**Status**: VERIFIED  
**Verified**: 2026-08-23