# COMBAT TASKS

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: COMBAT — tareas y threading  
**Source of Truth**: `commons/threads/ThreadPool.java`, `taskmanagers/*.java`, `entity/actor/Creature.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## 1. `ThreadPool` (foundation)

- **Path**: `commons/threads/ThreadPool.java`
- Métodos de planificación:
  - `schedule(Runnable, long delay)` → one-shot (devuelve `ScheduledFuture`).
  - `scheduleAtFixedRate(Runnable, delay, period)` → periódica.
  - `schedulePriorityTaskAtFixedRate(...)` → periódica prioritaria.
- Uso en combate:
  - `ThreadPool.schedule(new HitTask(...), atkSpeed)` — golpes.
  - `ThreadPool.schedule(mut, skillTime-400)` → `_skillCast` (futures guardados).

---

## 2. Tareas periódicas del combate

| Manager | Tipo | Scheduling | Responsabilidad |
|---------|------|------------|-----------------|
| `AttackStanceTaskManager` | Runnable+Singleton | `schedulePriorityTaskAtFixedRate(this, 0, 1000)` | marcar/limpiar `AttackStance` (15s) |
| `AttackableThinkTaskManager` | Singleton | (pool periódico) | think del AI de atacables |
| `RespawnTaskManager` | Runnable+Singleton | `scheduleAtFixedRate(this, 0, 1000)` | respawn NPC |
| `DecayTaskManager` | Runnable+Singleton | `scheduleAtFixedRate` | decay de cuerpos |
| `MovementTaskManager` | (manager) | ~0.1s | updatePosition de creatures móviles |
| `GameTimeTaskManager` | manager | ticks | tiempo del juego |

---

## 3. El flujo de schedule dentro del combate

### Programación de golpe
```
Creature.doAttackHitByBow(...)
   → ThreadPool.schedule(new HitTask(...), atkSpeed)
   HitTask.run() → onHitTimer(...)
```
- El hit **no** se guarda en campo; por tanto `abortAttack()` no lo cancela.

### Programación de cast (skill)
```
Creature.doCast(...) → beginCast(...)
   _skillCast (Future) = ThreadPool.schedule(mut, skillTime - 400)
   _skillCast2 ...
```
- `abortCast()` → `future.cancel(true)` + `_skillCast = null`.

---

## 4. `AttackStanceTaskManager` en detalle

- `CREATURE_ATTACK_STANCES: Map<Creature, Long>` (timestamp).
- `COMBAT_TIME = 15000` ms.
- `run()`:
```
iterate entries
    if (currentTime - entry.getValue() > COMBAT_TIME)
        remove(creature)
        on combat stop (hook)
```
- Se usa para PvP flag: `addAttackStanceTask(player)` desde `doAttack`.

---

## 5. `AttackableThinkTaskManager`

- Pool fijo de `POOL_SIZE=1000` tareas `AttackableThink` con `TASK_DELAY=1000`.
- Cada set recorre `attackables` para `notifyActionThink`.

---

## 6. Cómo se detienen / cancelan tareas

| Evento | Qué se cancela | Cómo |
|--------|----------------|------|
| Muere el atacante | `abortAttack/abortCast` | `_skillCast.cancel(true)` para cast; `abortAttack` solo `ActionFailed` |
| Target muere antes del hit | `HitTask` efectivo | `onHitTimer` detecta target dead → rechaza |
| Cambio de intención a IDLE | `AttackStance`? no (sí se hace en AI) | `setIntentionIdle` → `clientStopAutoAttack` + stopMove |
| Revive de Player | `DecayTaskManager.cancel(this)` | deja de decae cuerpo |

---

## 7. Trazabilidad

| Clase/Método | Path |
|--------------|------|
| `ThreadPool.schedule` | `commons/threads/ThreadPool.java` |
| `AttackStanceTaskManager` | `taskmanagers/AttackStanceTaskManager.java` |
| `AttackableThinkTaskManager` | `taskmanagers/AttackableThinkTaskManager.java` |
| `RespawnTaskManager` | `taskmanagers/RespawnTaskManager.java` |
| `DecayTaskManager` | `taskmanagers/DecayTaskManager.java` |
| `MovementTaskManager` | `taskmanagers/MovementTaskManager.java` |
| `HitTask` | `entity/actor/Creature.java` (inner) |
| `_skillCast` futuros | `entity/actor/Creature.java` |
| `abortAttack/abortCast` | `entity/actor/Creature.java` |

---
**Status**: VERIFIED  
**Verified**: 2026-08-23