# Packet Threading (Fase 2E)

## 1. Modelo REAL verificado (no asunciones de L2J clásico)

### a) Separation: I/O thread vs execution thread
```java
// ReadHandler.completed (commons)
_buffer.flip();
parseAndExecutePacket(client, buffer);   // sigue en el NIO thread hasta el dispatch
...
_executor.execute(packet);              // ← aquí pasa al ThreadPool
```

- El packet se parsea (`packet.read()` / `readImpl`) en el mismo **NIO client thread** (I/O).
- La **ejecución** (`ClientPacket.run()` → `runImpl()`) ocurre en el **`PacketExecutor`** ThreadPool, **NO** en el client thread.

### b) Runnable chain
- `ClientPacket implements Runnable` → `run()` → `runImpl()`.
- `AttackRequest extends ClientPacket` ⇒ instancia `Runnable`.

## 2. Scheduler / ThreadPool real verificado

| Component | Tipo | Propósito |
|---|---|---|
| `commons.network.packet.PacketExecutor<T>` | executor (ThreadPool) | ejecuta `packet.run()` (Runnable) |
| `commons.threads.ThreadPool` | scheduler singleton | `schedule`/`execute` para tasks de game (ataque, cast, efectos, decay) — usado por `Creature`/AI/Skills |
| NIO client thread (`org.l2jmobius.commons.network.*`) | daemon select-loop | solo I/O: read header/payload, decrypt, dispatch |

### Uso en game packets
- `AttackRequest.runImpl` llama a `player.onActionRequest()` y `target.onAction(player)` / `onForcedAttack(player)` → estos llegan a `CreatureAI`/`AttackStanceTaskManager` → que usa `ThreadPool` internamente.
- `Creature.doAttack` programa `HitTask` vía `ThreadPool.schedule(...)` (verificado en COMBAT/ATTACK_FLOW y COMBAT/COMBAT_TASKS).
- `RequestActionUse` usa helper `useSkill(player, name, targetSelf)` → `Player.useMagic` → `Creature.beginCast` → `MagicUseTask` via `ThreadPool` (verificado).

## 3. Relación con AI/combat/skills/quests (conectores reales)
- `AttackRequest` → `player.onActionRequest()` + `target.onAction/onForcedAttack` → `PlayerAI` / `CreatureAI` → `doAttack` (Combat/ATTACK_FLOW).
- `RequestActionUse` (acción de skill) → `useSkill(player, ...)` → `Player.useMagic(skill)` → `beginCast` → `MagicUseTask` (SKILLS/COMBAT).
- `RequestBypassToServer` → `BypassHandler`/`Quest` events → Quest callbacks (QUESTS).

## 4. Estado de verificación
- `ClientPacket implements Runnable`; `ReadHandler` ejecuta en `_executor.execute`: **VERIFIED** (commons ReadHandler + ClientPacket).
- `PacketExecutor<T>` como ThreadPool ejecutor: **VERIFIED** (referenciado por ReadHandler).
- `ThreadPool` (`commons.threads.ThreadPool`) para tasks de juego: **VERIFIED** (importado por `Creature`, `AttackStanceTaskManager`, `MagicUseTask`).
- Política exacta de thread-safety entre NIO thread y game thread (fences/sincronización por thread): **REQUIRES CODE VERIFICATION**.

Cross-ref: `PACKET_ARCHITECTURE.md`, `PACKET_DISPATCH.md`, `PACKET_PLAYER_FLOW.md`, `COMBAT/COMBAT_TASKS.md`
