# AI TASKS

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: AI — tareas programadas  
**Source of Truth**: `taskmanagers/AttackStanceTaskManager.java`, `taskmanagers/AttackableThinkTaskManager.java`, `taskmanagers/RespawnTaskManager.java`, `taskmanagers/DecayTaskManager.java`, `taskmanagers/MovementTaskManager.java`, `commons/threads/ThreadPool.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## 1. Visión general de ThreadPool

- `commons.threads.ThreadPool` provee:
  - `schedule(Runnable, delay)` → one-shot `ScheduledFuture`.
  - `scheduleAtFixedRate(Runnable, delay, period)` → tarea periódica.
  - `schedulePriorityTaskAtFixedRate(...)` → tarea periódica con prioridad.
- Las tareas programadas devuelven `Future<?>` que pueden cancelarse con `future.cancel(true)`.

## 2. `HitTask` — el golpe físico

- Se agenda dentro de `Creature.doAttackHitByBow/CrossBow/Pole/Dual`:
  - `ThreadPool.schedule(new HitTask(this, target, damage1, crit1, miss1, shld1, attack.hasSoulshot(), rechargeShots), sAtk)`
- `HitTask.run()` → `onHitTimer(...)` (aplica el daño al target).
- **No se almacena como campo** → no se puede cancelar directamente; la cancelación efectiva ocurre en `onHitTimer` (si el target murió / se desapareció).

## 3. `AttackStanceTaskManager` (Runnable, Singleton)

- **Path**: `taskmanagers/AttackStanceTaskManager.java`
- Declaración: `implements Runnable`.
- Estado: `Map<Creature, Long> CREATURE_ATTACK_STANCES` + `COMBAT_TIME = 15000` (ms).
- Programación: `ThreadPool.schedulePriorityTaskAtFixedRate(this, 0, 1000)`.
- `addAttackStanceTask(creature)` — marca que entró en ataque.
- `removeAttackStanceTask(creature)` — elimina.
- `hasAttackStanceTask(creature)` — consulta.
- En `run()`: elimina las entradas donde `currentTime - entry.getValue() > COMBAT_TIME` (15s).
- **Relación con combate**: `Creature.doAttack` llama a `AttackStanceTaskManager.getInstance().addAttackStanceTask(player)` cuando un player ataca fuera de PvP zone (marca de atacante).

## 4. `AttackableThinkTaskManager` (Singleton)

- **Lugar**: `taskmanagers/AttackableThinkTaskManager.java`
- **Tipo**: `private int POOL_SIZE = 1000`, `TASK_DELAY = 1000`.
- Mantiene `POOLS: Set<Set<Attackable>>`.
- `add(Attackable)` / `remove(Attackable)`.
- Corre funciones `AttackableThink` para recordar/piensar.

## 5. `RespawnTaskManager`

- **Línea**: `scheduleAtFixedRate(this, 0, 1000)`.
- Estado: `PENDING_RESPAWNS: Map<Npc, Long>`.
- `add(npc, time)`, `run()` itera y cuando `currentTime > entry.getValue()` → `spawn.respawnNpc(npc)`.
- Conexión con `DeathFlow`: cuando un NPC muere, se agenda respawn si `Spawn._doRespawn` está activo (ver `Spawn.java`).

## 6. `DecayTaskManager` — desaparición de cuerpo

- **Métodos**: `add(creature)` (programa decay), `cancel(creature)` (cancelar — usado en `Player.doRevive`), `getRemainingTime`.
- Se dispara tras la muerte (decay del "cuerpo" del NPC/monstruo).

## 7. `MovementTaskManager` — movimiento al 0.1s

- `MOVING_OBJECTS`: lista de creatures en movimiento.
- Invoca `updatePosition()` ~0.1s. Relacionado con `Creature.moveTo`/`moveToPawn` y cambios de región (ver `Creature.moveTo` y `Creature.getPosition`).

---

## 8. Cómo se programan / detienen / cancelan

| Operación | Cómo se hace |
|-----------|--------------|
| Programar golpe | `ThreadPool.schedule(HitTask(...), atkSpeed)` |
| Programar skill cast | `ThreadPool.schedule(mut, skillTime-400)` → `_skillCast/_skillCast2 (Future)` |
| Cancelar cast | `abortCast()` → `_skillCast.cancel(true)` + `_skillCast2.cancel(true)` |
| Cancelar ataque | `abortAttack()` → solo envía `ActionFailed` (NO cancela HitTask) |
| Marcar entrada en combate | `AttackStanceTaskManager.addAttackStanceTask(creature)` |
| Dejar combate | alcanzar `COMBAT_TIME=15s` (o `removeAttackStanceTask` vía zona/duelo) |
| Respawn | `RespawnTaskManager.add(npc, time)` + `respawnNpc` cuando llega el tick |

---

## 9. Qué ocurre si...

- **cambia el target**: el `HitTask` ya programado va al target capturado (una referencia); si el atacante cambia su `getAttackTarget()` antes de `run()` el golpe puede **no** aplicarse si valida `target != attackTarget` en algunos caminos → marcado `REQUIRES CODE VERIFICATION`.
- **muere el atacante**: de genera `doDie` → target del muerto se limpia, `abortAttack/abortCast` (para cast). El `HitTask` del muerto si ya programado podría ejecutarse, pero `onHitTimer` revertiría por `isAlikeDead()`.
- **muere el objetivo**: `onHitTimer` detecta target dead / `isAlikeDead` → no aplica daño (solo recharge).

---
**Fuente**: código en `taskmanagers/*` y `commons/threads/ThreadPool.java`  
**Status**: VERIFIED (con notas marcadas REQUIRES CODE VERIFICATION)  
**Verified**: 2026-08-23