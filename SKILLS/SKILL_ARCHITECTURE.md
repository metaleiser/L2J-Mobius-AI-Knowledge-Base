# SKILL ARCHITECTURE

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Sistema de Skills  
**Source of Truth**: `gameserver/mechanics/skill/`, `gameserver/handler/EffectHandler.java`, `gameserver/handler/TargetHandler.java`, `dist/game/data/scripts/handlers/skill/`, `data/xml/SkillData.java`, `util/DocumentSkill.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (arquitectura)

> El código fuente es la única fuente de verdad. Este documento describe la estructura REAL verificada; no reemplaza al código.

---

## 1. LAS TRES CAPAS DEL SISTEMA (hallazgo clave)

A diferencia de otros forks de L2J, en este proyecto el comportamiento concreto de efectos y targeting **NO está compilado en el core**: vive como **scripts** dentro del datapack.

| Capa | Ubicación | Contenido |
|------|-----------|-----------|
| **Core Java** | `java/org/l2jmobius/gameserver/mechanics/skill/` + `mechanics/effects/` + `mechanics/conditions/` | Modelo runtime: `Skill`, `BuffInfo`, `AbstractEffect`, `Condition`, enums, tareas |
| **Registros Java** | `gameserver/handler/EffectHandler.java`, `handler/TargetHandler.java` | Mapas nombre→clase que los scripts alimentan en arranque |
| **Scripts datapack** | `dist/game/data/scripts/handlers/skill/effects/` (**165 archivos**) y `handlers/skill/targets/` (**34 archivos**) | Implementaciones concretas de cada efecto y target handler |

Consecuencia práctica: cambiar un efecto concreto = editar un script en `dist/` (recargable por ScriptEngine), no recompilar el core.

---

## 2. ÁRBOL DE PAQUETES VERIFICADO

```
gameserver/
├── mechanics/
│   ├── skill/
│   │   ├── Skill.java                 ← clase central
│   │   ├── BuffInfo.java              ← estado de un efecto aplicado
│   │   ├── BuffFinishTask.java        ← expiración de buff
│   │   ├── EffectScope.java (enum)
│   │   ├── SkillOperateType.java (enum: A1,A2,A3,A4,CA1,CA5,DA1,DA2,P,T)
│   │   ├── AbnormalType.java / AbnormalVisualEffect.java
│   │   ├── CommonSkill.java           ← skills "well-known"
│   │   ├── MountEnabledSkillList.java
│   │   ├── SkillChannelizer.java / SkillChannelized.java
│   │   ├── holders/  → SkillHolder, SkillLearn, EnchantSkillLearn
│   │   ├── enums/    → AcquireSkillType, EffectCalculationType,
│   │   │                EnchantSkillType, FlyType, SkillFinishType
│   │   └── targets/  → TargetType, AffectScope, AffectObject
│   ├── effects/
│   │   ├── AbstractEffect.java        ← base de los 165 efectos
│   │   ├── EffectFlag.java / EffectType.java
│   │   └── EffectTaskInfo.java / EffectTickTask.java
│   └── conditions/                    ← 88 archivos (base Condition + concretas)
├── handler/
│   ├── EffectHandler.java             ← registro runtime de efectos
│   └── TargetHandler.java             ← registro runtime de targets
├── data/xml/SkillData.java            ← loader/caché de skills
├── util/DocumentSkill.java            ← parser XML → objetos Skill
└── entity/actor/tasks/creature/MagicUseTask.java (+ QueuedMagicUseTask)

dist/game/data/
├── stats/skills/*.xml (+ custom/)     ← definición declarativa
├── xsd/skills.xsd                     ← validación
└── scripts/handlers/
    ├── EffectMasterHandler.java       ← registra los 165 efectos
    └── skill/{effects,targets}/       ← implementaciones script
```

---

## 3. DECLARACIONES CLAVE VERIFICADAS

| Archivo | Declaración |
|---------|-------------|
| `Skill.java` | `public class Skill` |
| `BuffInfo.java` | `public class BuffInfo` |
| `SkillOperateType.java` | `public enum SkillOperateType` |
| `EffectScope.java` | `public enum EffectScope` |
| `holders/SkillHolder.java` | `public class SkillHolder` |
| `targets/TargetType.java` | `public enum TargetType` (38 valores) |
| `conditions/Condition.java` | `public abstract class Condition implements ConditionListener` |
| `effects/EffectTickTask.java` | `public class EffectTickTask implements Runnable` |
| `handler/EffectHandler.java` | `public class EffectHandler implements IHandler<Class<? extends AbstractEffect>, String>` |

**NO existen**: `SkillHandler`, `SkillTree.java`, clases de efectos compiladas en core, ni operate types "CHANCE/TRIGGERED/AURA" (los triggered son OptionSkills — ver COMBAT/CRITICALS.md).
## 4. DEPENDENCIAS DEL SISTEMA

```
Skill
├── SkillData / DocumentSkill      ← carga XML
├── EffectHandler (scripts 165)    ← resolución de <effect name="...">
├── TargetHandler (scripts 34)     ← selección de targets
├── Conditions (88 clases)         ├── Formulas (daño/success)
├── BuffInfo + EffectList          ├── ThreadPool (_skillCast/_skillCast2, EffectTickTask)
├── Creature (doCast/beginCast/callSkill/onMagic*Timer)
├── AI (useMagic / thinkCast)      ├── Events (OnCreatureSkillUse…)
├── Items (getSkills → SkillHolder)
└── DB (restore/store skills — solo conexión)
```

## 5. NOTA DE NO-DUPLICACIÓN

La referencia de skills vive **solo** en `SKILLS/` (esta carpeta). No se crea `SYSTEMS/SKILL_SYSTEM.md`; el enlace planificado en `MASTER_INDEX.md` fue sustituido.

---
**Fuente**: `mechanics/skill/**`, `handler/{EffectHandler,TargetHandler}.java`, `dist/game/data/scripts/handlers/skill/**`, `data/xml/SkillData.java`, `util/DocumentSkill.java`  
**Status**: VERIFIED  
**Verified**: 2026-08-23