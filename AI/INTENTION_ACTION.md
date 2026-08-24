# INTENTION / ACTION SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: AI — intenciones y acciones  
**Source of Truth**: `ai/Intention.java`, `ai/Action.java`, `ai/NextAction.java`, `ai/CreatureAI.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## 1. `Intention` (enum)

**Path**: `ai/Intention.java`  
**Package**: `org.l2jmobius.gameserver.ai`

Javadoc real:
> "Enumeration of generic intentions of an NPC/PC, an intention may require several steps to be completed."

Valores (en orden):
- `IDLE` — no hace nada.
- `ACTIVE` — alerta sin objetivo; escaneo, random walk.
- `REST` — descansar (sentado hasta que ataquen).
- `ATTACK` — atacar objetivo (usar magia de combate, ir al objetivo).
- `CAST` — lanzar conjuro (puede iniciar o parar ataque según el conjuro).
- `MOVE_TO` — moverse a coordenada.
- `FOLLOW` — seguir a un objetivo.
- `PICK_UP` — recoger un item.
- `INTERACT` — moverse al objetivo e interactuar.

**Almacenamiento**: `AbstractAI._intention` (campo). Exposición: `getIntention()`.

## 2. `Action` (enum)

**Path**: `ai/Action.java`

Enum real: *"Enum representing possible actions that can occur for an AI character."*

Valores:
- `THINK`, `ATTACKED`, `AGGRESSION`, `STUNNED`, `PARALYZED`, `SLEEPING`, `ROOTED`,
- `EVADED`, `READY_TO_ACT`, `USER_CMD`, `ARRIVED`, `ARRIVED_REVALIDATE`, `ARRIVED_BLOCKED`,
- `FORGET_OBJECT`, `CANCEL`, `DEATH`, `FAKE_DEATH`, `CONFUSED`, `MUTED`, `AFRAID`, `FINISH_CASTING`, `BETRAYED`.

### Semántica (según los comentarios del enum)
- `THINK` → el AI debe decidir la siguiente acción tras un cambio.
- `AGGRESSION` → aumentar/disminuir agresión hacia un objetivo o reducer agresión global.
- `EVADED` → evitó un ataque.
- `READY_TO_ACT` → la acción previa terminó, listo para la siguiente.
- `ARRIVED*` → llegó al destino / punto intermedio / bloqueado.
- `FORGET_OBJECT` → olvida un objeto específico.
- `CANCEL` → cancelar el paso actual sin cambiar la intención.
- `DEATH` / `FAKE_DEATH` → muerte real/aparente.
- `BETRAYED` → traiciona al maestro.
- etc.

## 3. `NextAction` (clase + Callback)

**Path**: `ai/NextAction.java`

```java
public class NextAction {
    public interface Callback { void doAction(); }
    private final Action _action;
    private final Intention _intention;
    private final Callback _callback;
    ...
    public boolean isTriggeredBy(Action action) { return _action == action; }
    public boolean isRemovedBy(Intention intention) { return _intention == intention; }
    public void doAction() { _callback.doAction(); }
}
```

**Concepto (Javadoc)**: acción diferida que se dispara ante un `Action` concreto y se remueve ante una `Intention` concreta. Ejecuta `Callback.doAction()`.

### Flujo real de uso

```
CreatureAI.notifyActionAttacked(attacker):
    if canHandleAction():
        clientStartAutoAttack()
        nextAction = getNextAction()
        if (nextAction != null && nextAction.isTriggeredBy(Action.ATTACKED)):
            nextAction.doAction()
```

```
CreatureAI.setIntentionIdle():
    ...
    if (nextAction != null && nextAction.isRemovedBy(Intention.IDLE)):
        setNextAction(null)
```

## 4. Trazabilidad

| Elemento | Path |
|----------|------|
| `Intention` enum | `ai/Intention.java` |
| `Action` enum | `ai/Action.java` |
| `NextAction` + `Callback` | `ai/NextAction.java` |
| Uso de `NextAction` | `ai/CreatureAI.java` (notifyActionAttacked, setIntentionIdle) |
| Campo `_intention` | `ai/AbstractAI.java` |

---
**Status**: VERIFIED  
**Verified**: 2026-08-23