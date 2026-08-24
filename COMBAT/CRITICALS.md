# CRITICALS

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: COMBAT — críticos físicos y mágicos  
**Source of Truth**: `mechanics/stats/Formulas.java`, `entity/actor/Creature.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (flujo y puntos de decisión) · fórmulas internas → REQUIRES CODE VERIFICATION

---

## 1. Crítico físico

**Decisión**: ocurre ANTES de programar el golpe, dentro de cada `doAttackHitBy*`:

```java
crit = Formulas.calcCrit(this, target);            // sobrecarga simple
crit = Formulas.calcCrit(this, target, skill);     // sobrecarga con skill
damage = Formulas.calcPhysDam(this, target, null, shld, crit, hasSoulshot);
```

- El boolean `crit` viaja dentro del `HitTask` hasta `onHitTimer`.
- En `onHitTimer` se reutiliza para:
  - `notifyDamageReceived(damage, attacker, null, crit, false)` (evento `ON_CREATURE_DAMAGE_DEALT/RECEIVED` llevan el flag).
  - Trigger skills: si `holder.getSkillType() == OptionSkillType.CRITICAL && crit && Rnd.get(100) < chance` → `makeTriggerCast`.

**Métodos verificados** (`mechanics/stats/Formulas.java`):
- `public static boolean calcCrit(Creature attacker, Creature target)`
- `public static boolean calcCrit(Creature attacker, Creature target, Skill skill)`

> La fórmula interna (rate base, modificadores por arma/clase/nivel) → `REQUIRES CODE VERIFICATION`.

## 2. Crítico mágico

**Método verificado**:
- `public static boolean calcMCrit(double mRate)` — recibe una tasa ya calculada.
- Se usa en el pipeline mágico (`calcMagicDam` recibe `mcrit` como parámetro en su firma).

> De dónde sale `mRate` (stat de caster) → `REQUIRES CODE VERIFICATION` (fase SKILL).

## 3. Mensajes de crítico/miss

- `Creature.sendDamageMessage(Creature target, int damage, boolean mcrit, boolean pcrit, boolean miss)`:
  - Base verificada solo envía mensaje de evasión (`SystemMessageId.C1_HAS_EVADED_C2_S_ATTACK`) cuando `miss && target.isPlayer()`.
  - Los overrides concretos (mensajes de crítico) viven en subclases → no leídos en esta fase (`UNKNOWN` detalle).

## 4. Resumen del flujo

```text
doAttackHitBy*
   ├─ miss  = calcHitMiss(atk,tgt)
   └─ !miss → crit = calcCrit(atk,tgt[,skill])
                ↓
        damage = calcPhysDam(..., crit, ss)
                ↓
HitTask → onHitTimer(..., crit, ...)
   ├─ mensajes / eventos con flag crit
   └─ trigger skills CRITICAL (probabilidad por holder)
```

---

## 5. TRAZABILIDAD

| Elemento | Path |
|----------|------|
| `calcCrit` / `calcMCrit` | `mechanics/stats/Formulas.java` |
| Decisión pre-golpe | `entity/actor/Creature.doAttackHitByBow/CrossBow/Pole/Dual/Simple` |
| Uso en impacto | `entity/actor/Creature.onHitTimer` |
| Trigger skills críticos | `entity/actor/Creature.onHitTimer` (`_triggerSkills`, `OptionSkillHolder`) |
| `sendDamageMessage` | `entity/actor/Creature.java` |

---
**Status**: VERIFIED (flujo/puntos de decisión) · tasas internas UNKNOWN  
**Verified**: 2026-08-23