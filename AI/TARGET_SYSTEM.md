# TARGET SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: AI — objetivo de entidades  
**Source of Truth**: `entity/actor/Creature.java`, `ai/AbstractAI.java`, `ai/AttackableAI.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## 1. DÓNDE SE ALMACENA

Existen **dos niveles** de target:

1. **`Creature._target`** (`WorldObject`) — objetivo de acción de la entidad.
   - `getTarget()` / `setTarget(WorldObject)`.
   - `setTarget`: si el objeto no está `spawned`, se fija `null`.
2. **`AbstractAI`** — objetivos internos del AI:
   - `_target` (WorldObject) — genérico.
   - `_attackTarget` (Creature) — target de ataque.
   - `_castTarget` (Creature) — target de cast.
   - `_followTarget` (Creature) — target a seguir.
   - Accesos: `getAttackTarget()`, `setAttackTarget()`, `getCastTarget()`, `setCastTarget()`, `getFollowTarget()`, `setFollowTarget()`.

**Nota**: el target mostrado/client vs el target de ataque pueden diferir; `AbstractAI` usa los campos separados.

## 2. Quién lo establece

- **Creature**: `Creature.setTarget(object)` (validación: si no está spawn → `_target = null`).
- **Player**: el packet de ataque/interacción del cliente llama a `setTarget` (p. ej. `Player` / handlers de acción).
- **Monster/Attackable**: `AttackableAI.thinkAttack` hace `setTarget(mostHate)` y `setAttackTarget(mostHate)`.
- Al morir (`Creature.doDie`): `setTarget(null)`.

## 3. Validación (target válido)

### `AttackableAI.checkTarget(WorldObject target)` (lopredominante para monstruos)
1. `target == null` → false.
2. Si `target.isCreature()`:
   - `target.asCreature().isDead()` → false.
   - Si `npc.isMovementDisabled()`: debe estar `npc.isInsideRadius2D(target, physicalAttackRange + colisiones)` y `GeoEngine.canSeeTarget(npc, target)` → en caso contrario false.
   - `!target.isAutoAttackable(npc)` → false.
3. Sin más restricciones → true.

### `Creature.doAttack(target)`
- Valida `target == null || isAttackDisabled() || !target.isTargetable()`.
- Además `GeoEngine.canSeeTarget(this, target)`, y para player: observer/siegeGuard/peace zone.
- Si el target muere entre cálculo y hit, `onHitTimer` lo descarta.

## 4. Cambio de target durante combate

**`AttackableAI.targetReconsider(boolean randomTarget)`**:
- Para `RaidBoss`/`GrandBoss`/minions con `_chaosTime`: recorre `getAggroList()` y valida `checkTarget` para obtener un nuevo candidato.
- Si el target actual muere (`mostHate.isAlikeDead()`) en `thinkAttack` → `npc.stopHating(mostHate)`.
- Si todas las entradas de aggro son inválidas (dead / no spawn / out of region) → `getMostHated()` devuelve `null` y el AI pasa a `setIntentionActive()`.

## 5. Target y el sistema de visitantes

El target siempre debe estar **spawned**; `setTarget` evita objetivos sin spawn. El target puede perderse por:
- Muerte del target.
- Despawn (fuera de región).
- `FORGET_OBJECT` (Action).
- `targetReconsider` (raid/boss).

## 6. Trazabilidad

| Método/Clase | Path |
|--------------|------|
| `Creature.setTarget/getTarget` | `entity/actor/Creature.java` (~L4649) |
| `AbstractAI._target/_attackTarget/...` | `ai/AbstractAI.java` |
| `AttackableAI.checkTarget` | `ai/AttackableAI.java` (~L1308) |
| `AttackableAI.targetReconsider` | `ai/AttackableAI.java` (~L1346) |
| `Creature.doDie` setTarget(null) | `entity/actor/Creature.java` (~L2664) |

---
**Status**: VERIFIED  
**Verified**: 2026-08-23