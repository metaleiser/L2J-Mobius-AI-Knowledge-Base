# DAMAGE CALCULATION

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: COMBAT — cálculo de daño  
**Source of Truth**: `mechanics/stats/Formulas.java`, `entity/actor/Creature.java`, `entity/actor/status/PlayerStatus.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (métodos y flujo) · cuerpos exactos de fórmulas → REQUIRES CODE VERIFICATION

---

## 1. CLASE CENTRAL: `Formulas`

**Path**: `mechanics/stats/Formulas.java`  
**Package**: `org.l2jmobius.gameserver.mechanics.stats`

**Métodos verificados (firmas):**

| Método | Firma | Descripción |
|--------|-------|-------------|
| `calcPhysDam` | `(Creature atk, Creature tgt, Skill skill, byte shld, boolean crit, boolean ss)` | daño físico final |
| `calcBlowDamage` / `calcBackstabDamage` | (ataque, target, skill, shld, ss) | golpes especiales tipo blow/backstab |
| `calcMagicDam` | `(atk, tgt, skill, shld, sps, bss, mcrit)` y overload `(Cubic,...)` | daño mágico |
| `calcManaDam` | `(atk, tgt, skill, shld, sps, bss, mcrit)` | daño directo a MP |
| `calcHitMiss` | `(atk, tgt)` → boolean | evasión/miss |
| `calcCrit` | `(atk, tgt[,skill])` → boolean | crítico físico |
| `calcMCrit` | `(double rate)` → boolean | crítico mágico |
| `calcShldUse` | `(atk, tgt[, skill, sendMsg])` → byte | escudo (bloqueo) |
| `calcPAtkSpd` | `(atk, tgt, rate)` → int | velocidad de ataque físico |
| `calcAtkSpd` | `(atk, skill, skillTime)` → int | velocidad de casteo |
| `calcDamageReflected` | `(atk, tgt, skill, crit)` → void | cálculo de reflejo |
| `calcAtkBreak` | `(target, damage)` → boolean | ruptura de ataque/break |
| `calcAttributeBonus` | `(atk, tgt, skill)` | bonus de atributo |
| `calcGeneralTraitBonus` | `(atk, tgt, traitType, ignoreResistance)` | resistencia a traits |
| `calcLvlBonusMod` | `(atk, tgt, skill)` | mod de nivel |
| `calcWeaponTraitBonus` / `calcAttackTraitBonus` | `(atk,tgt)` | bonus de arma/ataque |

> La **implementación** (fórmulas matemáticas) no se leyó íntegramente en esta fase → todos los detalles numéricos quedan `REQUIRES CODE VERIFICATION`.

---

## 2. PIPELINE FÍSICO (dónde se aplica el cálculo)

```text
doAttackHitByBow/CrossBow/Pole/Dual/Simple (Creature ~L1360+)
   → Formulas.calcHitMiss(attacker, target)
   → Formulas.calcShldUse(...)
   → Formulas.calcCrit(...)
   → Formulas.calcPhysDam(...)
   → HitTask → onHitTimer
```

### Modificadores que ocurren en `onHitTimer` (no en Formulas)
- **Distancia (bow)**: `damage *= (calculateDistance3D(target) / 4000) + 0.8` (solo si `PlayerConfig.CALCULATE_DISTANCE_BOW_DAMAGE`).
- **Raid curse**: si `target.isRaid() && giveRaidCurse() && nivel+8 > target` → `damage=0` y `skill.applyEffects` (petrifica al atacante).
- **Reflejo** (`Stat.REFLECT_DAMAGE_PERCENT`) → se refleja daño al atacante (excluye bows). Llam a `calcDamageReflected`.
- **Absorción HP/MP** (`Stat.ABSORB_DAMAGE_PERCENT`, `ABSORB_MANA_DAMAGE_PERCENT`).
- **Break**: `Formulas.calcAtkBreak(target, damage)` → `breakAttack/breakCast`.

---

## 3. PIPELINE MÁGICO (esquema, conexión a Skill)

```text
Creature.doCast(skill[, target[, targets]])
  → validación (mana, reuse, range, geo)
  → beginCast
  → Formule calcMagicSuccess / calcMagicDam / calcMCrit
  → applyEffects / daño
```
> El detalle completo (efectos, `applyEffects`, elementos) se documenta en la fase **SKILL**. Aquí se deja la conexión visible, `REQUIRES CODE VERIFICATION` el path completo.

---

## 4. CÓMO EL IMPACTO AFECTA AL RECEPTOR

Finalmente el daño llega vía `reduceCurrentHp` (Creature/PlayerStatus).

### `CreatureStatus.reduceHp` (genérico)
```java
if (_creature.isDead()) return;
if (_creature.isInvul() && !(isDOT || isHPConsumption)) return;
...
setCurrentHp(Math.max(_currentHp - value, 0));
if ((_creature.getCurrentHp() < 0.5) && _creature.isMortal()) doDie(attacker);
```

### `PlayerStatus.reduceHp` (con CP)
```java
// PlayerStatus.java ~L71
reduceHp(value, attacker, awake, isDOT, isHPConsumption, ignoreCP)
  ...
  // CP se consume ANTES que HP (reduceCp)
  // luego reduceHp(restante)
```
Ver también `reduceCp(int)` y `setCurrentCp`.

---

## 5. CRITICALS / MISS / SHIELD

- `calcCrit(attacker,target)` decide crítico físico (doble línea en `onHitTimer`).
- `calcMCrit(rate)` decide crítico mágico.
- `calcHitMiss(attacker,target)` decide miss; se llama ANTES de calcular daño; si miss, el daño es 0 y `onHitTimer` no aplica daño.
- `calcShldUse(attacker,target)` → byte: `SHIELD_REJECT`, `SHIELD_BLOCK`, `SHIELD_NONE`; reduce daño (en `calcPhysDam`) y determina efecto de bloqueo.

> Fórmulas exactas de protección (PDef), `EVASION`, `Accuracy`, y `cálculo de reducción` **no están desmenuzadas** en esta fase → marcadas `REQUIRES CODE VERIFICATION`.

---

## 6. TRAZABILIDAD

| Método | Path |
|--------|------|
| `Formulas.calcPhysDam/calcMagicDam/calcCrit/calcHitMiss/...` | `mechanics/stats/Formulas.java` |
| `doAttackHitBy*` | `entity/actor/Creature.java` |
| `CreatureStatus.reduceHp` | `entity/actor/status/CreatureStatus.java` |
| `PlayerStatus.reduceHp` | `entity/actor/status/PlayerStatus.java` |
| `notifyDamageReceived` | `entity/actor/Creature.java` ~L7214 |

---
**Status**: VERIFIED (estructura de cálculos y pipeline)  
**Verified**: 2026-08-23