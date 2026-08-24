# CAST FLOW

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Skills — flujo de casteo  
**Source of Truth**: `entity/actor/Creature.java` (doCast L1727/1737, beginCast L1886, onMagic*Timer, callSkill L6238), `entity/actor/tasks/creature/MagicUseTask.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## 1. PUNTOS DE ENTRADA

```java
public void doCast(Skill skill)                                  // L1727 → beginCast(skill, false)
public void doCast(Skill skill, Creature target, List<WorldObject> targets)  // L1737
   → beginCast(skill, false|true, target, targets)               // L1769/1756
```

Llamadores verificados:
- **Player**: packets de uso de skill → `PlayerAI`/`Playable.useMagic(...)` → `doCast`.
- **Monster/AttackableAI**: `thinkCast()` / `tryCast(...)` → `doCast`.
- **SummonAI**: `setIntentionCast(skill, target)` → cast.
- **Trigger skills** (Fase 2B): `Creature.makeTriggerCast` → `doCast`.

## 2. VALIDACIÓN PREVIA: `checkDoCastConditions(Skill)` (L2215)

Gate genérico antes del cast (muerto, ya casteando, skills deshabilitadas, muted, etc.). Detalle íntegro no enumerado línea a línea → `REQUIRES CODE VERIFICATION` si se requiere lista exacta.

## 3. `beginCast(skill, simultaneously, target, targets)` (L1886)

### 3.1 Checks iniciales
1. `target == null` → limpia flags de casting + `ActionFailed` + AI Active.
2. **Evento `ON_CREATURE_SKILL_USE`** (`OnCreatureSkillUse`, campos caster/skill/simultaneously/target/targets) → un listener puede devolver `TerminateReturn` y cancelar.
3. Bloqueo duro: skills de `EffectType.RESURRECTION` si `isResurrectionBlocked()` en caster o target.

### 3.2 Cálculo del tiempo de casteo
```
skillTime = hitTime + coolTime
si !channeling-sin-id:
    si !skill.isStatic():      skillTime = Formulas.calcAtkSpd(this, skill, skillTime)
    si isMagic() y SPS/BSPS cargado:  skillTime *= 0.6
mínimos de animación cliente:
    magic base>550 → floor 550ms ; física base>=500 → floor 500ms
```

### 3.3 Cola de consumibles
Si ya hay `_isCastingSimultaneouslyNow` y es simultaneous → reprograma a sí mismo en 100ms (`ThreadPool.schedule`) y retorna.

### 3.4 Flags e interrupción
- `setCastingSimultaneouslyNow(true)` o `setCastingNow(true)`.
- No-simultaneous: `_castInterruptTime = -2 + gameTicks + skillTime/MILLIS_IN_TICK`.
- `setLastSkillCast` / `setLastSimultaneousSkillCast`.

### 3.5 Cooldown (reuse)
```
base = getReuseDelay()
static/staticReuse → sin stat
isMagic()          → × Stat.MAGIC_REUSE_RATE
isPhysical()       → × Stat.P_REUSE
else               → × Stat.DANCE_REUSE
playable           → × ClassBalanceConfig.SKILL_REUSE_MULTIPLIERS[classId]
Formulas.calcSkillMastery → reuseDelay=100 (+mensaje ready)
reuseDelay > 1000 → addTimeStamp(skill, reuseDelay)     // TimeStamp map
10 < reuse ≤ 1000 → disableSkill(skill, delay)
```
Métodos verificados: `addTimeStamp(Skill,long)` L2438, `addTimeStamp(Skill,long,long)` L2450, `getSkillReuseTimeStamps()` L2428, `disableSkill(Skill,long)` L2533.

### 3.6 Costes y requisitos
- `MpInitialConsume` se descuenta YA al inicio (`_status.reduceMp`) + StatusUpdate.
- Player: item consume (`getItemConsumeId/Count`) — si falta → abort.
- Talisman (BodyPart.DECO): reduce mana del item; sin mana → abort.

### 3.7 Presentación
- Heading + `ExRotation` hacia target.
- `FlyType` → broadcast `FlyToLocation` + `setXYZ(target)`.
- Si no toggle: broadcast `MagicSkillUse` y `MagicSkillLaunched` (via `broadcastSkillPacket`).
- Player: mensaje `USE_S1` (casos especiales fishing 1312 / wolf collar 2046).

### 3.8 Efectos START
```java
if (skill.hasEffects(EffectScope.START))
    skill.applyEffectScope(EffectScope.START, new BuffInfo(this, target, skill), true, false);
```

### 3.9 Programación del golpe mágico
```java
MagicUseTask mut = new MagicUseTask(this, targets, skill, skillTime, simultaneously);
if skillTime > 0:
    player && !simultaneous → SetupGauge(BLUE, skillTime)
    channeling → getSkillChannelizer().startChanneling(skill)
    cancela Future previo (_skillCast2 o _skillCast)
    _skillCast(X) = ThreadPool.schedule(mut, max(0, skillTime - 400))   // 400ms antes por animación
else:
    mut.setSkillTime(0); onMagicLaunchedTimer(mut);
```

## 4. CADENA DE TAREAS (verificada)

```
MagicUseTask.run()                        [tasks/creature/MagicUseTask.java]
   → Creature.onMagicLaunchedTimer(mut)   L5866   (broadcast Launched; validaciones de rango/target)
   → Creature.onMagicHitTimer(mut)        L6003   (checks finales: dead, invul, range…)
        → callSkill(skill, targets)       L6100→L6238  (aplica efectos reales)
   → Creature.onMagicFinalizer(mut)       L6131   (limpia flags, reuse ya aplicado, enable next)
```
- `callSkill(Skill, List<WorldObject>)` es quien dispara `applyEffects`/daño (conexión con EFFECT_SYSTEM y MAGIC_DAMAGE).

## 5. INTERRUPCIÓN / CANCELACIÓN

| Situación | Efecto | Verificado en |
|-----------|--------|---------------|
| Listener cancela | flags off + ActionFailed | beginCast (OnCreatureSkillUse) |
| Falta item/talisman | `abortCast()` + return | beginCast |
| Stun/sleep/paralyze/etc. | `abortCast()` | llamadas L2086/2099/3477/3509/3524 |
| Daño recibido (prob.) | `_isCastingNow && canAbortCast()…` → `abortCast()` | L5498 |
| Muerte | `abortAttack()+abortCast()` antes de doDie | CreatureStatus.reduceHp |
| Manual | `Creature.abortCast()` L4274 → `future.cancel(true)` sobre `_skillCast/_skillCast2`; `canAbortCast()` L4224 | — |

## 6. CASOS ESPECIALES

- **Target muerto durante cast**: validado en `onMagicHitTimer` (no aplica; detalles exactos de cada check REQUIRES).
- **Caster muere**: `reduceHp` fuerza abort antes de morir; el Future queda cancelado.
- **Cambia el target**: el cast usa los targets capturados en `mut`; cambiar `getTarget()` después no redirige el golpe ya agendado.
- **Se mueve**: el movimiento puede romper el cast vía reglas estándar (detalle REQUIRES; el gauge sigue la animación).
- **Simultaneous** (pociones/hierbas): usa `_skillCast2` independiente y permite cola con delay 100ms.

## 7. TRAZABILIDAD

| Paso | Path |
|------|------|
| Entradas | `Playable.useMagic`, packets, AI thinkCast |
| Orquestador | `entity/actor/Creature.java` (L1727–2208) |
| Tasks | `entity/actor/tasks/creature/MagicUseTask.java`, `QueuedMagicUseTask.java` |
| Timers | `onMagicLaunchedTimer/onMagicHitTimer/callSkill/onMagicFinalizer` |
| Relación AI | `AI/INTENTION_ACTION.md` (Intention.CAST), `AI/AI_TASKS.md`, `COMBAT/COMBAT_TASKS.md` |

---
**Status**: VERIFIED  
**Verified**: 2026-08-23