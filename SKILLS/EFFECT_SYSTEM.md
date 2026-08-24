# EFFECT SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Skills — sistema de efectos  
**Source of Truth**: `mechanics/skill/Skill.applyEffects/applyEffectScope`, `mechanics/skill/BuffInfo.java`, `entity/actor/holders/creature/EffectList.java`, `mechanics/effects/*`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (flujo de aplicación) · reglas internas de stacking/dispel → REQUIRES CODE VERIFICATION

---

## 1. PIEZAS

| Clase | Path | Rol |
|-------|------|-----|
| `BuffInfo` | `mechanics/skill/BuffInfo.java` | Estado de una aplicación: effector, effected, skill, abnormalTime, efectos incluidos |
| `AbstractEffect` | `mechanics/effects/AbstractEffect.java` | Base de los 165 efectos script; callbacks `onStart/onExit/onTick/canStart/calcSuccess/isInstant/getEffectId` |
| `EffectList` | `entity/actor/holders/creature/EffectList.java` | Buffs activos por Creature; acceso vía `getEffectList()` |
| `EffectScope` (enum) | `mechanics/skill/EffectScope.java` | GENERAL, SELF, START, CHANNELING, PVE, PVP |
| `EffectTickTask` | `mechanics/effects/EffectTickTask.java` (`implements Runnable`) | tick periódico de un efecto |
| `EffectTaskInfo` | `mechanics/effects/EffectTaskInfo.java` | wrapper del task/future por efecto |
| `BuffFinishTask` | `mechanics/skill/BuffFinishTask.java` | expiración programada |
| `EffectFlag` / `EffectType` | `mechanics/effects/` | flags semánticos y tipos |

## 2. APLICACIÓN — `Skill.applyEffects(effector, effected, self, passive, instant, abnormalTime)` (L1338)

1. `effected == null` → return.
2. Skill negativa sobre invul / GM sin canGiveDamage → return.
3. `effected.isInvulAgainst(_id,_level)` → return.
4. `addContinuousEffects = !passive && (isToggle() || (isContinuous() && Formulas.calcEffectSuccess(effector, effected, this)))`
   → **aquí se decide la resistencia a debuffs**.
5. Rama principal (!self,!passive):
   - `BuffInfo info = new BuffInfo(...)` (+ setAbnormalTime si custom).
   - `applyEffectScope(GENERAL, info, instant, addContinuousEffects)`.
   - Scope PVP o PVE según effector.playable y tipo de effected.
   - `applyEffectScope(CHANNELING, ...)`.
   - Si continuo → `effected.getEffectList().add(info)`.

### `applyEffectScope(scope, info, applyInstantEffects, addContinuousEffects)` (L1281)
```java
for (AbstractEffect e : getEffects(scope)):
    if e.isInstant():
        if applyInstantEffects && e.calcSuccess(effector, effected, skill)
            → e.onStart(effector, effected, skill)     // daño/heal inmediato
    else if addContinuousEffects && e.canStart(...)
            → info.addEffect(e)                         // buff/debuff persistente
```

## 3. SCOPES VERIFICADOS EN CÓDIGO

- `START` — aplicado en `beginCast` (antes del golpe).
- `GENERAL` — el bloque principal en callSkill.
- `PVP` / `PVE` — extra si effector playable y effected playable/attackable.
- `CHANNELING` — para skills canalizadas.
- `SELF` — rama self (efectos al caster), incluye lógica especial con `hasEffectType(EffectType.BUFF)`.

## 4. DURACIÓN / TICKS / FIN

- Duración: campo `abnormalTime` de BuffInfo (customizable por overload).
- Ticks: cada efecto con periodo agenda `EffectTickTask` (Runnable) envuelto en `EffectTaskInfo`; cancelación al remover el efecto.
- Expiración: `BuffFinishTask` gestiona el fin programado del buff.
- **Muerte**: `doDie` → stopAllEffects (según respawn) — ver COMBAT/DEATH_FLOW.md.
- **Logout**: `Player.storeEffect(boolean)` (~L7759) guarda buffs para restore — conexión con DB (schema fuera de alcance).
- **Dispel/cancel**: operaciones sobre EffectList (cancel/removal); detalle interno de reglas → REQUIRES CODE VERIFICATION.

## 5. STACKING / REEMPLAZO

- Ocurre dentro de `EffectList.add(info)` (agrupación por `AbnormalType`, reemplazo por nivel/tiempo).
- Reglas exactas no verificadas línea a línea en esta fase → REQUIRES CODE VERIFICATION.

## 6. CLASIFICACIÓN DE EFECTOS REALES

Los **165 scripts** en `dist/game/data/scripts/handlers/skill/effects/` cubren categorías observadas por nombres verificados:
daño (`PhysicalDamage`, `Backstab`), control (`Betray`), aggro (`AddHate`), aplicación compuesta (`ApplySkillEffects`), traits (`AttackTrait`), etc.

Categorías pedidas que SÍ existen como efectos/scripts o flags: daño, heal, buff/debuff, root/stun/sleep/silence/paralysis/poison/bleed (vía EffectType/AbnormalType), stat modifiers (FuncTemplate), transformación, summon, teleport.
→ El catálogo completo de 165 nombres vive en el directorio; no se replica aquí.

## 7. TRAZABILIDAD

| Elemento | Path |
|----------|------|
| Motor | `mechanics/skill/Skill.java` L1281–1400+ |
| Estado | `mechanics/skill/BuffInfo.java` |
| Contenedor | `entity/actor/holders/creature/EffectList.java` |
| Ticks | `mechanics/effects/{EffectTickTask,EffectTaskInfo}.java` |
| Fin | `mechanics/skill/BuffFinishTask.java` |
| Implementaciones | `dist/game/data/scripts/handlers/skill/effects/*.java` (165) |

---
**Status**: VERIFIED (flujo de aplicación) · stacking/dispel internos REQUIRES  
**Verified**: 2026-08-23