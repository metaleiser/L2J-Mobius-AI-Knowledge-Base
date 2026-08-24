# COMBAT ARCHITECTURE

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Sistema de combate  
**Source of Truth**: `entity/actor/Creature.java`, `entity/actor/Attackable.java`, `mechanics/stats/Formulas.java`, `mechanics/events/holders/actor/creature/*`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (flujo base); detalles de fórmulas → REQUIRES CODE VERIFICATION

---

## 1. VISIÓN GENERAL

El combate físico/golpe y magia están coordinados principalmente por:

- **`Creature`**: `doAttack`, `onHitTimer`, `doCast`, `reduceCurrentHp`, `doDie`.
- **`CreatureStatus`** / **`PlayerStatus`**: aplicación del daño (HP/CP).
- **`Formulas`**: cálculo de daño (físico/mágico, miss/crit/shield).
- **AI**: inicia el ataque (`AttackableAI`, `PlayerAI`, `SummonAI`).

El flujo de daño se detalla en `COMBAT/DAMAGE_CALCULATION.md` y el flujo de ataque completo en `ATTACK_FLOW.md`.

---

## 2. INICIO DEL ATAQUE

### Físico
```
[Cliente] AttackRequest / RequestActionUse
   → PlayerAI.setIntentionAttack(target)
   → CreatureAI.setIntentionAttack / (AttackableAI override)
   → thinkAttack() (en AttackableAI)
       → if target válido y en rango: _actor.doAttack(getAttackTarget())
```

### Mágico
```
[Cliente] RequestActionUse → PlayerAI.setIntentionCast(skill, target)
   → Creature.doCast(skill[, target[, targets]])
   → beginCast (thread) ... (detalle en fase SKILL; aquí solo conexión)
```

---

## 3. CORE DE COMBATE (clases y métodos)

### `Creature.doAttack(Creature target)`
- **Path**: `entity/actor/Creature.java` (~L1034)
- Flujo:
  1. `_attackLock` (StampedLock) — previene doble golpe simultáneo.
  2. Valida `target == null`, `isAttackDisabled()`, `!target.isTargetable()`.
  3. Eventos `ON_CREATURE_ATTACK` (atacante) y `ON_CREATURE_ATTACKED` (target); si un listener devuelve `TerminateReturn` → cancela ataque (AI Active + ActionFailed).
  4. Validaciones de player: observer, siegeFriend, peaceZone, zona.
  5. Geo `canSeeTarget`; `stopMove`.
  6. Calcula `timeAtk = calculateTimeBetweenAttacks()`, `timeToHit = timeAtk/2`.
  7. switch por arma: BOW/CROSSBOW → `doAttackHitByBow/CrossBow`; POLE → `doAttackHitByPole`; DUAL/DUALFIST/DUALDAGGER/FIST(simple) → `doAttackHitByDual/Simple`; default → simple.
  8. `AttackStanceTaskManager.addAttackStanceTask(player)` (si player fuera de PvP) y `updatePvPStatus`.

### `doAttackHitByBow` (y CrossBow/Pole/Dual/Simple)
- **Path**: `Creature.doAttackHitBy*` (~1360+)
- Cada una hace:
  - `miss = Formulas.calcHitMiss(this,target)`.
  - consume flecha/bolt (`reduceArrowCount`).
  - si no miss: `shld = Formulas.calcShldUse(this,target)`; `crit = Formulas.calcCrit(this,target)`; `damage = Formulas.calcPhysDam(this,target,null,shld,crit,hasSoulshot)`.
  - (Bow: mod distancia `PlayerConfig.CALCULATE_DISTANCE_BOW_DAMAGE`).
  - `ThreadPool.schedule(new HitTask(...), atkSpeed)` → `onHitTimer`.
  - `attack.addHit(...)` y devuelve si no miss.
- Para `DUAL`/`FIST` sencillo: 1 hit.
- Para `DUALFIST`/`DUALDAGGER`: 2 hits encadenados (distintos timings).

### `onHitTimer` (aplicación del golpe)
- **Path**: `Creature.onHitTimer` (~L5281)
- Descarta si `target == null` / `isAlikeDead()`.
- Si `miss` → target AI `notifyActionEvaded`.
- Si hit:
  - Valor de `damage`.
  - Raid curse (si atacante > nivel+8).
  - Refleja (`Stat.REFLECT_DAMAGE_PERCENT`, no en bow; `calcDamageReflected`).
  - `target.reduceCurrentHp(damage, this, null)`.
  - `target.notifyDamageReceived(damage, this, null, crit, false)`.
  - Absorb HP/MP (`ABSORB_DAMAGE_PERCENT`).
  - `target.getAI().notifyActionAttacked(this)`.
  - `Formulas.calcAtkBreak(target, damage)` → `breakAttack/breakCast`.
  - Trigger skills del atacante (crit/attack).

### `Creature.doCast(Skill skill[, target[, targets]])`
- **Path**: `Creature.doCast` (~L1727/1737)
- Typed:
  - `doCast(Skill)` / `doCast(Skill, Creature, List<WorldObject>)`.
  - Realiza inicio de casteo, maná, `_skillCast`/`_skillCast2` futuros.
- Nota: en `doCast` de `AttackableAI` (clase) se hacen excepciones para evitar buffear a jugadores (ver `Monster.doCast`).

### `Formulas` (cálculo de daño)

Firmas verificadas (ver `mechanics/stats/Formulas.java`):

| Método | Uso |
|--------|-----|
| `calcPhysDam(attacker, target, skill, shld, crit, ss)` | daño físico |
| `calcMagicDam(attacker, target, skill, shld, sps, bss, mcrit)` / cubic | daño mágico |
| `calcManaDam(...)` | daño a mana |
| `calcHitMiss(attacker, target)` | miss |
| `calcCrit(attacker, target)` / (con skill) | critical físico |
| `calcMCrit(double rate)` | crítico mágico |
| `calcShldUse(attacker, target[, skill, sendMsg])` | shield block |
| `calcPAtkSpd(attacker, target, rate)` / `calcAtkSpd` | velocidad ataque |
| `calcDamageReflected(...)` | reflejo |
| `calcAtkBreak(target, damage)` | romper cast/ataque |
| `calcAttributeBonus`, `calcGeneralTraitBonus`, `calcWeaponTraitBonus`, `calcAttackTraitBonus`, `calcLvlBonusMod` | mods |

> Los **cuerpos exactos** no se detallan en esta fase → `REQUIRES CODE VERIFICATION`.

---

## 4. ENTIDADES AFECTADAS

### `CreatureStatus` / `PlayerStatus`
- `CreatureStatus.reduceHp`:
  - checks invul; para player, CP antes de HP (`PlayerStatus`).
  - `_currentHp = max(hp-value, 0)`.
  - si `hp < 0.5 && isMortal()` → `doDie(attacker)`.
- `Creature.doDie` → ver `DEATH_FLOW.md`.

### `AttackStanceTaskManager`
- Marca atacante en combate (15 seg). Relacionado con PvP flag.

---

## 5. EVENTOS DE COMBATE (real)

- `OnCreatureAttack` / `OnCreatureAttacked`
- `OnCreatureDamageDealt` / `OnCreatureDamageReceived`
- `OnCreatureDeath` / `OnCreatureKilled`
- `OnAttackableAttack`, `OnAttackableHate`, `OnAttackableAggroRangeEnter`, `OnAttackableFactionCall`, `OnAttackableKill`

(La mecánica de estos eventos se documenta en su fase; los nombres están verificados.)

---

## 6. TRAZABILIDAD

| Clase/Método | Path |
|--------------|------|
| `Creature.doAttack` | `entity/actor/Creature.java` ~L1034 |
| `Creature.onHitTimer` | `entity/actor/Creature.java` ~L5281 |
| `Creature.doCast` | `entity/actor/Creature.java` ~L1728 |
| `Creature.doAttackHitBy*` | `entity/actor/Creature.java` ~L1360 |
| `CreatureStatus.reduceHp` | `entity/actor/status/CreatureStatus.java` ~L134 |
| `PlayerStatus.reduceHp` (CP) | `entity/actor/status/PlayerStatus.java` ~L71 |
| `Formulas` (calc*) | `mechanics/stats/Formulas.java` |
| `AttackStanceTaskManager` | `taskmanagers/AttackStanceTaskManager.java` |

---
**Status**: VERIFIED (arquitectura) · `REQUIRES CODE VERIFICATION` en fórmulas exactas  
**Verified**: 2026-08-23