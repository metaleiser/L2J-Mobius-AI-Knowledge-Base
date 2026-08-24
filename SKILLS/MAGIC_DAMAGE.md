# MAGIC DAMAGE

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Skills — daño mágico y resistencias (resuelve UNKNOWNs de Fase 2B)  
**Source of Truth**: `mechanics/stats/Formulas.java`, `mechanics/skill/Skill.callSkill→applyEffects`, `entity/actor/Creature.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (firmas y puntos de llamada) · cuerpos matemáticos → REQUIRES CODE VERIFICATION

---

## 1. FIRMA VERIFICADA DEL CÁLCULO MÁGICO

```java
// Formulas.java
public static double calcMagicDam(Creature attacker, Creature target, Skill skill,
                                  byte shld, boolean sps, boolean bss, boolean mcrit)
public static double calcMagicDam(Cubic attacker, Creature target, Skill skill,
                                  boolean mcrit, byte shld)
public static double calcManaDam(Creature attacker, Creature target, Skill skill,
                                 byte shld, boolean sps, boolean bss, boolean mcrit)
```

- Existe variante para **Cubics** (atacante Cubic).
- `calcManaDam`: daño directo a MP.

## 2. CRÍTICO MÁGICO

```java
public static boolean calcMCrit(double mRate)
```
- Recibe una tasa ya calculada (`mRate`). El origen exacto de `mRate` (stat del caster) NO se verificó en esta fase → `REQUIRES CODE VERIFICATION`.
- El resultado entra como parámetro `mcrit` a `calcMagicDam`.

## 3. RESISTENCIA / ÉXITO MÁGICO

Métodos verificados en `Formulas`:
- `public static boolean calcMagicSuccess(Creature attacker, Creature target, Skill skill)` — éxito del efecto mágico.
- `public static boolean calcEffectSuccess(Creature attacker, Creature target, Skill skill)` — usado por `Skill.applyEffects` para decidir si un efecto continuo/debuff se aplica (ver EFFECT_SYSTEM §2.4).
- `public static boolean calcCubicSkillSuccess(Cubic attacker, Creature target, Skill skill, byte shld)`.

> Cuerpos internos (fórmulas con mAtk/mRes/level difference) → REQUIRES CODE VERIFICATION.

## 4. ATRIBUTOS / ELEMENTOS

```java
public static double calcAttributeBonus(Creature attacker, Creature target, Skill skill)
public static double calcGeneralTraitBonus(Creature attacker, Creature target, TraitType traitType, boolean ignoreResistance)
public static double calcAttackTraitBonus(...)
public static double calcWeaponTraitBonus(...)
```
- Conectan los elementos del XML (`element`, `elementPower`) y traits con el daño final.
- Punto de aplicación dentro de `calcMagicDam/calcPhysDam` → REQUIRES CODE VERIFICATION.

## 5. REFLECT / ABSORB (conexión Fase 2B)

- Reflejo: `Formulas.calcDamageReflected(attacker, target, skill, crit)` + lógica de `onHitTimer` para golpes físicos; para mágico el reflejo ocurre vía efectos/stats (`Stat.REFLECT_*`) — punto exacto REQUIRES.
- Absorción: `Stat.ABSORB_DAMAGE_PERCENT` / `ABSORB_MANA_DAMAGE_PERCENT` aplicados en `onHitTimer` físico (ya documentado); equivalente mágico vía efectos.

---

## 6. FLUJO REAL DE DAÑO POR SKILL (verificado hasta Formulas)

```text
callSkill(skill, targets)                       [Creature L6238]
   ↓
skill.applyEffects(effector, effected, ...)      [Skill L1338]
   ↓
applyEffectScope(scope, info, instant=true, ...)
   ↓ efecto INSTANT (p.ej. script MDam/PhysicalDamage)
   ↓   e.calcSuccess(...)  → resistencia/éxito
   ↓   e.onStart(...)       → dentro: Formulas.calcMagicDam / calcPhysDam
   ↓                            (+ attribute/trait bonuses, shield, crit)
reduceCurrentHp(damage, effector, skill)          [Creature]
   ↓
CreatureStatus.reduceHp → doDie si <0.5           [COMBAT/DEATH_FLOW.md]
```

Notas verificadas:
- El daño instantáneo lo ejecuta el **script del efecto** llamando a las fórmulas; el core no "sabe" de tipos de daño.
- `shld` (bloqueo) también participa en daño mágico (`calcShldUse`).
- SPS/BSPS afectan tiempo de cast (beginCast) y entran como flags a `calcMagicDam`.

## 7. TRAZABILIDAD

| Elemento | Path |
|----------|------|
| Fórmulas | `mechanics/stats/Formulas.java` |
| Disparo | `Creature.callSkill` (~L6238) |
| Motor efectos | `mechanics/skill/Skill.applyEffects` (~L1338) |
| Scripts de ejemplo | `dist/game/data/scripts/handlers/skill/effects/*.java` |

---
**Status**: VERIFIED (firmas/ruta) · cuerpos numéricos REQUIRES CODE VERIFICATION  
**Verified**: 2026-08-23