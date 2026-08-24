# QUEST EVENTS

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Quests — eventos y callbacks  
**Source of Truth**: `mechanics/script/Quest.java` (callbacks L601–1156, registro L2704+), `entity/actor/Attackable.java` (doDie L315–330), `entity/actor/tasks/attackable/OnKillNotifyTask.java`, `mechanics/events/listeners/ConsumerEventListener.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## 1. DOS MECANISMOS DE DESPACHO (verificados)

1. **Notificación directa**: el core llama `quest.notifyXxx(...)` / `quest.onXxx(...)` (el script sobrescribe).
2. **EventDispatcher con consumers**: los scripts registran callbacks por NPC-ID:
```java
addKillId(npcIds)  →  setAttackableKillId(kill -> onKill(kill.getTarget(), kill.getAttacker(), kill.isSummon()), npcIds)
                     → registerConsumer(callback, EventType.ON_ATTACKABLE_KILL, ListenerRegisterType.NPC, npcIds)
```
   → crea `ConsumerEventListener`s asociados a cada NPC.

Además existe **registro por anotaciones**: `initializeAnnotationListeners()` en el constructor de Quest (con validación que emite warning `"Non properly defined annotation listener on method"`). Detalle del sistema de anotaciones → REQUIRES CODE VERIFICATION.

## 2. FLUJO REAL DE KILL (verificado)

```
Attackable.doDie(killer)                       [Attackable.java L315]
   → super.doDie ok
   → killer player && EventDispatcher.hasListener(ON_ATTACKABLE_KILL, this)
        → EventDispatcher.notifyEventAsyncDelayed(
              new OnAttackableKill(player, this, killer.isSummon()), this)   // tras _onKillDelay=2500ms
   → OnKillNotifyTask.run() → quest.onKill(attackable, killer, isSummon)
```
- `_onKillDelay` configurable: `setOnKillDelay(int)` / `getOnKillDelay()` (Attackable L1744).

## 3. CATÁLOGO COMPLETO DE CALLBACKS (Quest.java, verificados)

### NPC / mundo
| Callback | Firma resumida |
|----------|----------------|
| onTalk | `(Npc, Player) → String` |
| onFirstTalk | `(Npc, Player) → String` |
| onEvent | `(String event, Npc, Player) → String` |
| onSpawn | `(Npc)` |
| onAttack | `(Npc, Player attacker, int damage, boolean isSummon[, Skill])` |
| onKill | `(Npc, Player killer, boolean isSummon)` |
| onDeath | `(Creature killer, Creature victim, QuestState qs)` |
| onSkillSee | `(Npc, Player caster, Skill, List<WorldObject>, boolean isSummon)` |
| onSpellFinished | `(Npc, Player, Skill)` |
| onTrapAction | `(Trap, Creature trigger, TrapAction)` |
| onFactionCall | `(Npc, Npc caller, Player attacker, boolean isSummon)` |
| onAggroRangeEnter | `(Npc, Player, boolean isSummon)` |
| onCreatureSee | `(Npc, Creature)` |
| onMoveFinished / onNodeArrived / onRouteFinished | `(Npc)` (+notify*) |
| notifyTeleport | `(Npc)` |
| onNpcHate / notifyOnCanSeeMe / onCanSeeMe | control de agro/visibilidad |

### Jugador / social
| Callback | Firma |
|----------|-------|
| onEnterWorld | `(Player)` |
| onEnterZone / onExitZone | `(Creature, ZoneType)` |
| onAcquireSkillList / onAcquireSkillInfo / onAcquireSkill | skill learning |
| onOlympiadMatchFinish / onOlympiadLose / notifyOlympiadMatch | olimpiada |
| onItemTalk / notifyItemTalk | `(Item, Player)` |
| onItemEvent | `(Item, Player, String event)` |

### Inter-quests / timers
| Callback | Firma |
|----------|-------|
| notifyEventReceived / onEventReceived | `(eventName, Npc sender, Npc receiver, WorldObject ref)` |
| onTimerEvent / onTimerCancel | desde `IEventTimerEvent<String>` / `IEventTimerCancel<String>` |
| onSummonSpawn / onSummonTalk | `(Summon)` |

## 4. REGISTRO DE IDs (helpers públicos de Quest)

| Helper | Línea | Registra |
|--------|-------|----------|
| `addStartNpc(int...)` | L1462 | NPCs que inician la quest |
| `addTalkId(int...)` | L1588 → setNpcTalkId | NPCs con diálogo |
| `addKillId(int...)` | L1570 → setAttackableKillId(→onKill) | NPC kills |
| `addAttackId(int...)` | L1552 | ataques a NPC |
| `registerQuestItems(int...)` | L2347 | quest items |
| `setCreatureKillId(...)` | L2755 | ON_CREATURE_DEATH genérico |

Otros `setXxxId(...)` análogos existen para el resto de eventos (patrón idéntico).

## 5. TRAZABILIDAD

| Elemento | Path |
|----------|------|
| Catálogo callbacks | `mechanics/script/Quest.java` L601–1156 |
| Despacho kill | `entity/actor/Attackable.java` L315–330 + `tasks/attackable/OnKillNotifyTask.java` |
| Registro consumers | `mechanics/events/listeners/ConsumerEventListener.java` + `ListenerRegisterType.NPC` |
| Anotaciones | `Quest.initializeAnnotationListeners()` → REQUIRES detalle |

---
## Ver también

- [COMBAT/DEATH_FLOW.md](../COMBAT/DEATH_FLOW.md) — muerte del NPC y cadena hasta rewards
- [COMBAT/COMBAT_ARCHITECTURE.md](../COMBAT/COMBAT_ARCHITECTURE.md) — eventos ON_CREATURE_* / ON_ATTACKABLE_* verificados
- [AI/AI_TASKS.md](../AI/AI_TASKS.md) — task managers relacionados con el ciclo AI

---
**Status**: VERIFIED · sistema de anotaciones REQUIRES CODE VERIFICATION  
**Verified**: 2026-08-23