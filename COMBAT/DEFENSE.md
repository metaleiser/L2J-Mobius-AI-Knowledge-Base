# DEFENSE

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: COMBAT — defensa y mitigación  
**Source of Truth**: `mechanics/stats/Formulas.java`, `entity/actor/Creature.java`, `entity/actor/status/CreatureStatus.java`, `entity/actor/status/PlayerStatus.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (mecanismos y puntos de aplicación) · fórmulas internas → REQUIRES CODE VERIFICATION

---

## 1. Mecanismos de defensa verificados

| Mecanismo | Dónde se aplica | Detalle verificado |
|-----------|-----------------|--------------------|
| **Escudo (block)** | `Formulas.calcShldUse(attacker, target)` → byte; llamado en `doAttackHitBy*` antes de calcular daño | El valor se pasa a `calcPhysDam`; el byte indica uso/no uso de escudo |
| **Invulnerabilidad** | `CreatureStatus.reduceHp`: `if (_creature.isInvul() && !(isDOT \|\| isHPConsumption)) return;` | Ignora DOT y consumo propio de HP |
| **CP antes que HP (Player)** | `PlayerStatus.reduceHp(...)` (sobrecarga con `ignoreCP`) | El jugador consume CP primero; luego HP. Detalle completo del reparto → REQUIRES |
| **Reflejo de daño** | `onHitTimer`: `Stat.REFLECT_DAMAGE_PERCENT`; se excluyen bows; límite anti-raid (`nivel+8`) | Daño reflejado aplica `reduceCurrentHp(reflectedDamage, target, true, false, null)` al atacante |
| **Absorción HP/MP** | `onHitTimer`: `Stat.ABSORB_DAMAGE_PERCENT` / `Stat.ABSORB_MANA_DAMAGE_PERCENT` | Cap por `_stat.getMaxRecoverableHp()/Mp()` |
| **Interrupción por daño** | `CreatureStatus.reduceHp`: `stopEffectsOnDamage(awake)`; si `isStunned()` y `Rnd.get(10)==0` → `stopStunning(true)` | Solo cuando no es DOT/consumo |
| **Ruptura de ataque/cast** | `Formulas.calcAtkBreak(target, damage)` → `target.breakAttack(); target.breakCast();` | No aplica a raids (`!target.isRaid()`) |
| **Inmunidad offline trade** | `PlayerStatus.reduceHp`: si `OFFLINE_MODE_NO_DAMAGE` y cliente detached en store/craft | Solo jugadores |
| **GM sin permiso de daño** | `CreatureStatus.reduceHp`: attacker GM sin `canGiveDamage()` → ignora daño | Seguridad admin |
| **Champion monsters** | `Creature.reduceCurrentHp`: divide el daño entre `CHAMPION_HP` | Config `ChampionMonstersConfig` |
| **LOS para skills a distancia** | `Creature.reduceCurrentHp`: si player + skill castRange>0 + sin línea de visión → `amount = 0` | Uso de `GeoEngine.canSeeTarget` |

---

## 2. Lo que NO está desglosado aquí (REQUIRES CODE VERIFICATION)

- Fórmula interna de reducción por **PDef / defensa** dentro de `calcPhysDam`.
- Cálculo de **evasión** interno de `calcHitMiss` (atributos usados, caps).
- Reparto exacto **CP↔HP** en `PlayerStatus.reduceHp` (transferencia parcial vista en ~L198-207).
- Valores de `calcShldUse` (constantes SHIELD_*) y su efecto numérico.

---

## 3. TRAZABILIDAD

| Mecanismo | Path |
|-----------|------|
| `calcShldUse` | `mechanics/stats/Formulas.java` |
| `reduceHp` (genérica) | `entity/actor/status/CreatureStatus.java` (~L134-187) |
| `reduceHp` (con CP, player) | `entity/actor/status/PlayerStatus.java` (~L71+) |
| Reflejo/absorción/break | `entity/actor/Creature.onHitTimer` (~L5281-5460) |
| `calcAtkBreak` | `mechanics/stats/Formulas.java` |
| Champion divisor | `entity/actor/Creature.reduceCurrentHp` (~L6826) |

---
**Status**: VERIFIED (puntos de aplicación) · fórmulas internas marcadas  
**Verified**: 2026-08-23