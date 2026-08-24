# ATTACK FLOW

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: COMBAT — flujo de ataque  
**Source of Truth**: `ai/AttackableAI.java`, `ai/CreatureAI.java`, `entity/actor/Creature.java`, `network/clientpackets/AttackRequest.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## 1. DE DÓNDE VIENEN LOS ATAQUES

Los ataques provienen de:
- **Jugador** → `requestActionUse`/`AttackRequest` (clientpackets) → `PlayerAI.setIntentionAttack`.
- **NPC/Monster** → `AttackableAI.thinkAttack` → `_actor.doAttack`.

**Clases de iniciación verificadas**:
- `PlayerAI.java:362` → `_actor.doAttack(target)`.
- `AttackableAI.java:1305 / 1890` → `_actor.doAttack(getAttackTarget())`.
- `DistrustAI.java:65`, `FortSiegeGuardAI:675`, `SiegeGuardAI:673`, `SummonAI:144` → `_actor.doAttack(...)`.

---

## 2. FLUJO FÍSICO (SECUENCIA)

### 2.1 Player ataca (inicio)
```text
[Cliente] AttackRequest
   ↓
PlayerAI.setIntentionAttack(target)         // validaciones PlayableAI (protección, karma, cursed weapon)
   ↓
CreatureAI.setIntentionAttack(target)       // set _intention = ATTACK; think...
   ↓
AttackableAI.thinkAttack()                  // (solo para hostil)
   → getMostHated() → si null → setIntentionActive; return
   → valida rango/geo; si no rango → moveTo
   → ataque: _actor.doAttack(getAttackTarget())
```

### 2.2 `Creature.doAttack(target)` — lanzamiento
```text
_attackLock.tryWriteLock()
  if target inválida → return
  Evento ON_CREATURE_ATTACK (atacante) → puede cancelar (TerminateReturn)
  Evento ON_CREATURE_ATTACKED (target) → puede cancelar
  Check player (observer/paz/siegeFriend)
  Geo canSeeTarget
  stopMove
  timeAtk = calculateTimeBetweenAttacks()
  new Attack packet (heading, crystal)
  switch weapon:
     BOW/CROSSBOW → doAttackHitByBow/CrossBow (agenda HitTask)
     POLE → doAttackHitByPole
     DUAL/DUALFIST/DUALDAGGER/FIST... → doAttackHitByDual/Simple
  AttackStanceTaskManager.addAttackStanceTask(player)
```
La elección real de `HitTask` se hace en cada `doAttackHitBy*` (se agenda el golpe).

### 2.3 golpe (HitTimer) → daño
```
HitTask.run()
   → onHitTimer(target, damage, crit, miss, shld, soulshot, recharge)
        ├─ miss → notifyActionEvaded (target AI)
        └─ hit → reflection / absorb / reduceCurrentHp / notifyDamageReceived
```

### 2.4 Retorno al atacante
```
target.getAI().notifyActionAttacked(attacker)
```
Significa: el target “sabe” que fue atacado en el siguiente tick (para eventual hate/aggro).

---

## 3. FLUJO COMPLETO (unificado)

```text
Attacker(doAttack)
   ↓
ON_CREATURE_ATTACK / ON_CREATURE_ATTACKED
   ↓
Target validation (dead/targetable/geo/pz)
   ↓
Weapon switch → doAttackHitBy* → HitTask (ThreadPool, retraso atkSpeed)
   ↓
onHitTimer(target, damage, crit, miss, shld, ss)
   ├─ miss → EVADED
   └─ hit → reflection → absorb → reduceCurrentHp → notifyDamageReceived
   ↓
target.reduceCurrentHp → (HP baja / doDie si <0.5)
   ↓
Attackable.calculateRewards / drops / exp (si muerte)
```

---

## 4. REGISTRO DE HIT (hitTask)

- `ThreadPool.schedule(new HitTask(...), atkSpeed)` en cada `doAttackHitBy*`.
- El paquete `Attack` (ServerPacket) se envía por broadcast para animar golpes.
- `SetupGauge` (RED) se envía al player para la barra de recarga (BOW/CrossBow).

---

## 5. NOTA: ataque mágico

`doCast(Skill)` ocurre en paralelo al flujo físico (afecta a `_skillCast`). El detalle completo del daño mágico se documenta en fase `SKILL_SYSTEM`.

---

## 6. TRAZABILIDAD

| Paso | Clase/Método | Path |
|------|--------------|------|
| Intención | `PlayerAI.setIntentionAttack` | `ai/PlayerAI.java` |
| Intención hostil | `AttackableAI.thinkAttack` | `ai/AttackableAI.java` |
| Inicio golpe | `Creature.doAttack` | `entity/actor/Creature.java` |
| Cálculo daño | `Formulas.calcPhysDam` | `mechanics/stats/Formulas.java` |
| programar golpe | `ThreadPool.schedule(HitTask...)` | `entity/actor/Creature.java` |
| Aplicar | `Creature.onHitTimer` | `entity/actor/Creature.java` |
| Reducción HP | `CreatureStatus.reduceHp` | `entity/actor/status/CreatureStatus.java` |

---
**Status**: VERIFIED  
**Verified**: 2026-08-23