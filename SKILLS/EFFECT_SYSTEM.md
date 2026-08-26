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

**Status**: VERIFIED (flujo, EffectType, EffectFlag, routing, stacking)  
**Verified**: 2026-08-26
---

## 8. EffectType Catalog (36 values)

Source: `mechanics/effects/EffectType.java`.

| # | Value | # | Value |
|---:|---|---:|---|
| 1 | AGGRESSION | 2 | BUFF |
| 3 | CHARM_OF_LUCK | 4 | CHAT_BLOCK |
| 5 | CPHEAL | 6 | DEBUFF |
| 7 | DISPEL | 8 | DISPEL_BY_SLOT |
| 9 | DMG_OVER_TIME | 10 | DMG_OVER_TIME_PERCENT |
| 11 | DEATH_LINK | 12 | FAKE_DEATH |
| 13 | FEAR | 14 | FISHING |
| 15 | FISHING_START | 16 | HATE |
| 17 | HEAL | 18 | HP_DRAIN |
| 19 | MAGICAL_ATTACK | 20 | MANAHEAL_BY_LEVEL |
| 21 | MANAHEAL_PERCENT | 22 | MUTE |
| 23 | NEVITS_HOURGLASS | 24 | NOBLESSE_BLESSING |
| 25 | NONE | 26 | PARALYZE |
| 27 | PHYSICAL_ATTACK | 28 | PHYSICAL_ATTACK_HP_LINK |
| 29 | REBALANCE_HP | 30 | REFUEL_AIRSHIP |
| 31 | RELAXING | 32 | RESURRECTION |
| 33 | RESURRECTION_SPECIAL | 34 | ROOT |
| 35 | SLEEP | 36 | STEAL_ABNORMAL |
| 37 | STUN | 38 | SUMMON |
| 39 | SUMMON_PET | 40 | SUMMON_NPC |
| 41 | TELEPORT | 42 | TELEPORT_TO_TARGET |

**Count**: 42 values (enum starts at AGGRESSION and ends at TELEPORT_TO_TARGET). Used by individual effect scripts to report their semantic type.

---

## 9. EffectFlag Catalog (22 values + mask)

Source: `mechanics/effects/EffectFlag.java`.

| Value | Mask |
|---|---|
| NONE | 1 << 0 |
| RESURRECTION_SPECIAL | 1 << 1 |
| NOBLESS_BLESSING | 1 << 2 |
| SILENT_MOVE | 1 << 3 |
| PROTECTION_BLESSING | 1 << 4 |
| RELAXING | 1 << 5 |
| FEAR | 1 << 6 |
| CONFUSED | 1 << 7 |
| MUTED | 1 << 8 |
| PSYCHICAL_MUTED | 1 << 9 |
| PSYCHICAL_ATTACK_MUTED | 1 << 10 |
| PASSIVE | 1 << 11 |
| DISARMED | 1 << 12 |
| ROOTED | 1 << 13 |
| SLEEP | 1 << 14 |
| STUNNED | 1 << 15 |
| BETRAYED | 1 << 16 |
| INVUL | 1 << 17 |
| PARALYZED | 1 << 18 |
| BLOCK_RESURRECTION | 1 << 19 |
| SERVITOR_SHARE | 1 << 20 |
| POLEARM_SINGLE_TARGET | 1 << 21 |

Used by `EffectList.updateEffectFlags()` to set creature state flags.

---

## 10. Buff Category Routing

Source: `EffectList.getEffectList(Skill)` L211-241.

Each active skill effect is routed to exactly one queue based on skill category:

| Skill category | Queue | Counts against slot limit? |
|---|---|---|
| `isPassive()` | `_passives` | No |
| `isDebuff()` | `_debuffs` | No |
| `isTriggeredSkill()` | `_triggered` | Own limit |
| `isDance()` (`_magic == 3`) | `_dances` | Own limit |
| `isToggle()` | `_toggles` | No |
| default | `_buffs` | General limit |

This routing happens before stacking checks in `EffectList.add()`.

---

## 11. Stacking & Replacement Rules

Source: `EffectList.add(BuffInfo)` L1414-1567.

| Rule | Evidence |
|---|---|
| Grouping key | `AbnormalType` (`_stackedEffects` map) |
| Replacement | New `abnormalLevel >= old abnormalLevel` → old removed/hidden |
| Ignore weaker | New `abnormalLevel < old abnormalLevel` → new ignored |
| Same skill ID, no abnormal type | Old instance removed, new added at end |
| Herb / abnormalInstant | Old marked `setInUse(false)`, stats removed, still ticking |
| Slot overflow | Oldest buff in the same queue removed |

**Cross-AbnormalType conflicts**: `SOURCE_REQUIRED` — no general rule in SOURCE.

---

**Status**: VERIFIED (flujo, EffectType, EffectFlag, routing, stacking)  
**Verified**: 2026-08-26
