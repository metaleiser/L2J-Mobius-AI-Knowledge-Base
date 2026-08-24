# QUEST TIMERS

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Quests — timers  
**Source of Truth**: `mechanics/script/QuestTimer.java`, `mechanics/script/Quest.java` (métodos de timer L448–598, interfaces L192)  
**Verified**: 2026-08-23  
**Status**: VERIFIED · persistencia entre reinicios → UNKNOWN

---

## 1. INTERFACES IMPLEMENTADAS POR QUEST

```java
public class Quest implements IEventTimerEvent<String>, IEventTimerCancel<String>
{
    public void onTimerEvent(TimerHolder<String> holder)                       // L236
    public void onTimerCancel(TimerHolder<String> holder)                      // L242
    public void onTimerEvent(String event, StatSet params, Npc npc, Player player)   // L247 (hook script)
    public void onTimerCancel(String event, StatSet params, Npc npc, Player player)  // L252
    public TimerExecutor<String> getTimers()                                   // L259
    public boolean hasTimers()                                                 // L275
}
```
- Los scripts sobrescriben las versiones `(String event, StatSet params, Npc npc, Player player)`.

## 2. API DE TIMERS EN QUEST

```java
public void startQuestTimer(String name, long time, Npc npc, Player player)                  // L448
public void startQuestTimer(String name, long time, Npc npc, Player player, boolean repeating) // L471
public QuestTimer getQuestTimer(String name, Npc npc, Player player)                         // L497
public void cancelQuestTimers(String name)                                                   // L525 (todos con ese nombre)
public void cancelQuestTimer(String name, Npc npc, Player player)                            // L555
public void removeQuestTimer(QuestTimer timer)                                               // L582
public Map<String, List<QuestTimer>> getQuestTimers()                                        // L457
```

## 3. `QuestTimer`

- **Path**: `mechanics/script/QuestTimer.java` — `public class QuestTimer`.
- Encapsula: nombre del evento, tiempo, NPC objetivo, Player objetivo y la tarea programada.
- Al dispararse → invoca `onTimerEvent(name, params, npc, player)` de la quest.
- Si es `repeating`, se reprograma; si no, se elimina tras ejecutarse.

## 4. PROGRAMACIÓN SUBYACENTE

- Los timers usan el sistema genérico de **event timers** (`TimerHolder<String>` + `TimerExecutor<String>`), agendado sobre `commons.threads.ThreadPool`.
- Cancelación: explícita por nombre/objetivo, o automática en `unload()` de la quest.

## 5. PERSISTENCIA

- No se encontró evidencia de persistencia de timers activos entre reinicios → los timers viven en memoria mientras la quest esté cargada → `UNKNOWN / REQUIRES CODE VERIFICATION`.

## 6. TRAZABILIDAD

| Elemento | Path |
|----------|------|
| API pública | `mechanics/script/Quest.java` L448–598 |
| Interfaces | `IEventTimerEvent<String>`, `IEventTimerCancel<String>` (importadas por Quest L192) |
| Clase | `mechanics/script/QuestTimer.java` |
| Executor | `TimerExecutor<String>` / `TimerHolder<String>` (paquete timers — detalle REQUIRES) |

---
**Status**: VERIFIED (API) · persistencia UNKNOWN  
**Verified**: 2026-08-23