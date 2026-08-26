# SKILL DATA MODEL

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Skills — modelo de datos  
**Source of Truth**: `mechanics/skill/Skill.java`, `dist/game/data/stats/skills/*.xml`, `util/DocumentSkill.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (atributos XML reales observados; campos internos completos de Skill.java no enumerados exhaustivamente)

---

## 1. PRINCIPIO

Una **skill** se declara en XML y se representa en runtime como una instancia de `org.l2jmobius.gameserver.mechanics.skill.Skill` **por cada nivel** (y variante de enchant). No hay "template + instance" como en items: el objeto `Skill` ES la definición.

## 2. ATRIBUTOS XML REALES (observados en `00000-00099.xml`, skill id=1)

### Atributos del tag `<skill>`
| Atributo | Ejemplo | Propósito |
|----------|---------|-----------|
| `id` | `1` | identificador |
| `levels` | `37` | cantidad de niveles |
| `name` | `Triple Slash` | nombre |
| `enchantGroup1..7` | `2` | rutas de enchant habilitadas |

### Tablas `<table name="#...">`
Arrays indexados por nivel (o enchant): `#effectPoints`, `#magicLevel`, `#mpConsume`, `#power`, `#ench1Power`, `#enchDuel`, `#enchElementPower`, etc.

### Elementos hoja verificados
| Elemento | Ejemplo |
|----------|---------|
| `icon` | `icon.skill0001` |
| `operateType` | `A1` (ver `SkillOperateType`) |
| `targetType` | `ONE` |
| `baseCritRate` | `15` |
| `castRange` / `effectRange` | `40` / `400` |
| `hitTime` / `coolTime` | `1730` / `167` |
| `reuseDelay` | `3000` |
| `magicLevel` | tabla |
| `mpConsume` | tabla |
| `power` | tabla |
| `ignoreShld` | `true` |
| `nextActionAttack` | `true` |
| `overHit` | `true` |
| `enchant1..7 name="..."` | sobrescriben atributos por ruta de enchant |

> Otros atributos que el parser soporta (element/attribute/etc.) existen en otros skills del datapack; los listados aquí son los verificados directamente.

### Bloque `<conditions>`
```xml
<conditions msgId="113" addName="1">
    <using kind="DUAL" />
</conditions>
```
- Se parsean a objetos `Condition` (`mechanics/conditions/`).
- `msgId` = SystemMessageId si falla; `addName="1"` añade el nombre de la skill al mensaje.

### Bloque `<effects>` con scopes
```xml
<effects>
    <effect name="PhysicalDamage" />
</effects>
```
- `name` debe existir en `EffectHandler` (registrado por scripts).
- Existen scopes adicionales según posición en el XML (`<effects>`, self, start, channeling, pvp/pve — ver `EFFECT_SYSTEM.md`).

---

## 3. REPRESENTACIÓN RUNTIME (`Skill`)

Campos/métodos verificados por uso directo:

| Miembro | Uso verificado |
|---------|----------------|
| `_id`, `_level` | identidad |
| `_operateType` | `SkillOperateType` (isActive/isMagic/isPhysical/isToggle/isChanneling/isStatic) |
| `_targetType` | `TargetType` |
| `_effectPoint` | negativo ⇒ debuff (`hasNegativeEffect()`: `_effectPoint < 0 && targetType != SELF`) |
| `_isDebuff` | flag explícito XML `isDebuff="true"` |
| `_abnormalType`, `_abnormalLevel`, `_abnormalTime` | stacking, duración |
| `_isTriggeredSkill` | buffs de activación |
| `_magic` | 0=physical, 1=magic, 2=static, 3=dance |
| `_preCondition`, `_itemPreCondition` | listas `List<Condition>` |
| `getHitTime()/getCoolTime()` | cálculo de skillTime en `beginCast` |
| `getReuseDelay()/isStaticReuse()` | cooldowns |
| `getDisplayId()/getDisplayLevel()` | packets MagicSkillUse |
| `getFlyType()` | FlyToLocation |
| `hasEffects(EffectScope)/getEffects(scope)` | aplicación de efectos |
| `checkCondition(...)`, `applyEffects(...)`, `applyEffectScope(...)`, `getTargetList(...)` | motor |

> Enumeración completa de campos privados de `Skill.java` no realizada línea a línea → `REQUIRES CODE VERIFICATION` si se requiere inventario total.

## 4. Skill Categories & Beneficial/Harmful Semantics

### 4.1 Categories (rutas de `EffectList.getEffectList`)

| Category | Detection | Queue |
|---|---|---|
| Passive | `operateType == P` | `_passives` |
| Debuff | `_isDebuff == true` | `_debuffs` |
| Triggered | `_isTriggeredSkill == true` | `_triggered` |
| Dance/Song | `_magic == 3` | `_dances` |
| Toggle | `operateType == T` | `_toggles` |
| Normal buff | default | `_buffs` |

### 4.2 Beneficial / harmful — 4 layers

No single "beneficial" flag exists. Combine:

1. **Skill-level explicit**: `isDebuff()` (XML `isDebuff`), `is7Signs()`, `isRecoveryHerb()`.
2. **Derived negative**: `hasNegativeEffect() = _effectPoint < 0 && _targetType != SELF`.
3. **Effect-level**: `AbstractEffect.getEffectType()` returns `EffectType.BUFF/DEBUFF/STUN/SLEEP/HEAL/...`.
4. **Control flags**: `EffectFlag.MUTED/SLEEP/STUNNED/CONFUSED/...`.

For Scheme Validator decision tree see [`SKILL_SEMANTIC_REFERENCE.md`](SKILL_SEMANTIC_REFERENCE.md) §3.

## 5. QUÉ PERTENECE A QUÉ

| Dato | Dónde vive |
|------|------------|
| id/levels/name/tablas/tiempos/power/costes/targetType/operateType/enchants | **XML** |
| iconos/rutas | XML (cliente muestra su propio icono) |
| comportamiento de efecto concreto | **script** (`handlers/skill/effects/*`) referenciado por `name` |
| selección de targets concreta | **script** (`handlers/skill/targets/*`) según `targetType` |
| reglas de cast/cooldown/interrupción | **Java core** (`Creature`, `Formulas`) |
| mensajes SystemMessageId | enums Java; ids referenciados desde XML conditions |
| XSD validación | `dist/game/data/xsd/skills.xsd` |

---
**Status**: VERIFIED (atributos, categorías y semántica beneficial/harmful) · inventario total de campos → REQUIRES  
**Verified**: 2026-08-26