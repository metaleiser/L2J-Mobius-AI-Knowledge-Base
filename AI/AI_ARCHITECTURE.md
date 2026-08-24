# AI ARCHITECTURE

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Sistema de Inteligencia Artificial  
**Source of Truth**: `java/org/l2jmobius/gameserver/ai/`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (declaraciones `extends`/`implements` verificadas contra el código)

> Este documento es la referencia para `AI`/`COMBAT`. No reemplaza el código; el código es la fuente de verdad.

---

## 1. ÁRBOL DE CLASES (REAL, verificado)

Todas las declaraciones provienen de leer la línea `public ...` de cada archivo.

```
gameserver/ai/
├── AbstractAI (abstract class)
│   └── CreatureAI
│       ├── PlayableAI (abstract) ──► PlayerAI
│       │                            └─► SummonAI implements Runnable
│       ├── AttackableAI ──► DistrustAI
│       ├── KindAI (BoatAI, AirShipAI, DoorAI)
│       ├── SiegeGuardAI implements Runnable ──► SpecialSiegeGuardAI
│       └── FortSiegeGuardAI implements Runnable
├── Intention (enum)
├── Action (enum)
└── NextAction (class; contiene interface Callback)
```

**Tabla de declaraciones verificadas:**

| Archivo | Declaración |
|---------|-------------|
| `AbstractAI.java` | `public abstract class AbstractAI` |
| `CreatureAI.java` | `public class CreatureAI extends AbstractAI` |
| `PlayableAI.java` | `public abstract class PlayableAI extends CreatureAI` |
| `PlayerAI.java` | `public class PlayerAI extends PlayableAI` |
| `SummonAI.java` | `public class SummonAI extends PlayableAI implements Runnable` |
| `AttackableAI.java` | `public class AttackableAI extends CreatureAI` |
| `DistrustAI.java` | `public class DistrustAI extends AttackableAI` |
| `BoatAI.java` | `public class BoatAI extends CreatureAI` |
| `AirShipAI.java` | `public class AirShipAI extends CreatureAI` |
| `DoorAI.java` | `public class DoorAI extends CreatureAI` |
| `SiegeGuardAI.java` | `public class SiegeGuardAI extends CreatureAI implements Runnable` |
| `SpecialSiegeGuardAI.java` | `public class SpecialSiegeGuardAI extends SiegeGuardAI` |
| `FortSiegeGuardAI.java` | `public class FortSiegeGuardAI extends CreatureAI implements Runnable` |
| `Intention.java` | `public enum Intention` |
| `Action.java` | `public enum Action` |
| `NextAction.java` | `public class NextAction` |

---

## 2. RESPONSABILIDADES

### `AbstractAI`
- **Package**: `org.l2jmobius.gameserver.ai`
- **Path**: `ai/AbstractAI.java`
- **Extends**: (ninguna)
- **Responsibility**: clase madre de todos los AI del mundo.
  - `_actor` (Creature) — la criatura controlada.
  - `_intention` (Intention) — objetivo a largo plazo.
  - `_target`, `_attackTarget`, `_castTarget`, `_followTarget` — objetivos desacoplados.
  - `_nextAction` (NextAction).
  - `_skill` (Skill) — skill del CAST en curso.
- Métodos abstractos de entrada (notifs) que el mundo usa para comunicar eventos al AI sin acoplar clases: `notifyActionAttacked`, `notifyActionAggression`, `notifyActionEvaded`, `notifyActionDeath`, `notifyActionReadyToAct`, etc.
- También expone `getIntention()`, `getActor()`.

### `CreatureAI`
- **Path**: `ai/CreatureAI.java`
- **Extends**: `AbstractAI`
- **Responsibility**: implementación base concreta del manejo de intenciones (IDLE, ACTIVE, ATTACK, CAST, MOVE_TO, FOLLOW, ...) y el dispatch del `NextAction`.
  - `setIntentionIdle/Active/Rest/Attack/Cast/MoveTo/Follow/PickUp/Interact`.
  - `canHandleAction()` → `_actor.isSpawned() || _actor.isTeleporting() && _actor.hasAI()`.
  - `notifyActionAttacked` → ejecuta `NextAction` si `isTriggeredBy(Action.ATTACKED)`.
  - `setIntentionIdle` → limpia attack/cast target, StopMove, `clientStopAutoAttack`, y si `NextAction` se elimina (`isRemovedBy(Intention.IDLE)`).
  - `setIntentionActive` → `stopFollow`, cambia a ACTIVE y dispara think.

### `PlayableAI` (abstract)
- **Path**: `ai/PlayableAI.java`
- **Extends**: `CreatureAI`
- **Responsibility**: para `Player` y `Summon`. Valida restricciones anti-protección de playable target (ej. `isProtectionBlessingAffected`, karma, cursed weapon) antes de `setIntentionAttack`.

### `PlayerAI`
- **Path**: `ai/PlayerAI.java`
- **Extends**: `PlayableAI`
- **Responsibility**: AI del jugador (envía `setIntentionAttack`, `clientNotifyDead`, auto-ataque). Es quien llama `_actor.doAttack(target)` en respuesta a órdenes.

### `SummonAI`
- **Path**: `ai/SummonAI.java`
- **Extends**: `PlayableAI`
- **Implements**: `Runnable`
- **Responsibility**: AI del summon; verifica pathfinding (geodata) antes de atacar, gestiona follow del owner y `run()` (task periódico).

### `AttackableAI`
- **Path**: `ai/AttackableAI.java`
- **Extends**: `CreatureAI`
- **Responsibility**: AI de NPC atacables (monstruos, NPC hostiles). Core del comportamiento de caza: `thinkActive`, `thinkAttack`, `thinkCast`, `notifyActionAggression`, `targetReconsider`.
- Usa `getMostHated()` como candidato; gestión de `_globalAggro`, `rollGlobalAggro`, `setGlobalAggro`, `_attackTimeout`, `_chaosTime`.

### `DistrustAI`
- **Path**: `ai/DistrustAI.java`
- **Extends**: `AttackableAI`
- **Responsibility**: NPC que traiciona/ataca a un objetivo forzado (`_forcedTarget`).

### `Vehicle/Prop AI`
- `BoatAI` / `AirShipAI` / `DoorAI` — manejo de barcos, naves y puertas (no combate estándar).

### `SiegeGuardAI` / `FortSiegeGuardAI` / `SpecialSiegeGuardAI`
- **Path**: `ai/SiegeGuardAI.java`, `ai/FortSiegeGuardAI.java`, `ai/SpecialSiegeGuardAI.java`
- **Implements Runnable** (excepto Special que hereda de SiegeGuardAI).
- Responsibilidad: guardias de asedio (castillo/fortaleza); comportamiento de ronda con lógica propia de `setIntentionAttack`.

---

## 3. CLASIFICACIÓN RÁPIDA

| Rol | Clases |
|-----|--------|
| Base | `AbstractAI`, `CreatureAI` |
| Jugable | `PlayableAI`, `PlayerAI`, `SummonAI` |
| Ataque/hostil | `AttackableAI`, `DistrustAI` |
| Vehículo | `BoatAI`, `AirShipAI` |
| Objeto | `DoorAI` |
| Asedio | `SiegeGuardAI`, `FortSiegeGuardAI`, `SpecialSiegeGuardAI` |
| Datos de estado | `Intention`, `Action`, `NextAction` |

---
## 4. CONEXIONES CON OTROS SISTEMAS

- **Creature**: `Creature._ai` es el `CreatureAI` asignado (getter `getAI()`); el AI controla el `Creature` (movimiento/ataque/estado).
- **World**: los AI usan `World` (búsqueda de visibles, regiones).
- **TaskManager**: `AttackableThinkTaskManager`, `AttackStanceTaskManager`, etc. (ver `AI/AI_TASKS.md`).
- **Combate**: `AttackableAI` inicia `_actor.doAttack(...)` (ver `COMBAT/ATTACK_FLOW.md`).

## 5. NOTA IMPORTANTE (no duplicar)

El tema AI **no** se documenta en `SYSTEMS/AI_SYSTEM.md`; toda la referencia AI vive en `AI/`. (En fases previas `MASTER_INDEX` tenía marcado `SYSTEMS/AI_SYSTEM.md` como planificado; se sustituye por `AI/AI_ARCHITECTURE.md`.)

---
**Fuente**: `java/org/l2jmobius/gameserver/ai/**`  
**Status**: VERIFIED  
**Verified**: 2026-08-23