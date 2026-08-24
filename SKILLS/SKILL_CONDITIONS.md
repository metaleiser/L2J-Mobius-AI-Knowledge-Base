# SKILL CONDITIONS

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Skills — condiciones  
**Source of Truth**: `mechanics/conditions/` (88 archivos), `mechanics/skill/Skill.checkCondition` (~L946), XML `<conditions>`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (base y evaluación); catálogo completo de las 88 → ver listado de archivos

---

## 1. CLASE BASE

- **Path**: `mechanics/conditions/Condition.java`
- **Declaración**: `public abstract class Condition implements ConditionListener`
- Métodos verificados:

```java
public boolean test(Creature caster, Creature target, Skill skill)          // L107
public boolean test(Creature caster, Creature target, ItemTemplate item)    // L112
public boolean test(Creature caster, Creature target, Skill skill, ItemTemplate item)  // L117
public abstract boolean testImpl(Creature effector, Creature effected, Skill skill, ItemTemplate item); // L137
```

- Los `test(...)` públicos hacen pre-checks comunes y delegan en `testImpl` (que implementa cada condición concreta).

## 2. COMPOSICIÓN LÓGICA (AND / OR / NOT)

Existen clases reales:
- `ConditionLogicAnd`
- `ConditionLogicOr`
- `ConditionLogicNot`

→ Las condiciones se combinan anidando estas clases (el parser XML construye el árbol). Además `ConditionListener` permite notificar cambios que invalidan la condición.

## 3. CATÁLOGO (muestra verificada del directorio)

88 archivos en `mechanics/conditions/`. Muestra de nombres reales:
`Condition`, `ConditionCategoryType`, `ConditionChangeWeapon`, `ConditionGameChance`, `ConditionGameTime`, `ConditionInventory`, `ConditionItemId`, `ConditionListener`, `ConditionLogicAnd/Or/Not`, `ConditionMinDistance`, … más subpaquetes por dominio (`player/*`, `target/*`, etc.).

> Listado íntegro de las 88 → consultar directorio; no se replica aquí para evitar duplicación.

## 4. CUÁNDO Y QUIÉN EVALÚA

### Pre-cast — `Skill.checkCondition(Creature creature, WorldObject object, boolean itemOrWeapon)` (Skill.java ~L946)

```java
if (creature.isFakePlayer() || (isGM && !GeneralConfig.GM_SKILL_RESTRICTION)) return true;
if (player mounted && hasNegativeEffect() && !MountEnabledSkillList.contains(_id)) → mensaje + false
List<Condition> preCondition = itemOrWeapon ? _itemPreCondition : _preCondition;
for (Condition cond : preCondition)
    if (!cond.test(creature, target, this)) {
        msgId != 0 → SystemMessage(msgId) [+ addSkillName si addName]
        else msg → sendMessage(msg)
        return false;
    }
return true;
```

- **Quién**: la propia `Skill`, invocada antes/durante el intento de uso (desde packets/AI vía rutas de validación; conexión con CAST_FLOW §2).
- **Sobre qué**: caster (effector) + target actual + skill.
- **Falla** → el cast NO inicia; solo mensaje al caster.

### En items/armas
- `_itemPreCondition` usa la misma maquinaria con `itemOrWeapon=true`.

## 5. XML ↔ JAVA

| XML | Java |
|-----|------|
| `<conditions msgId="113" addName="1">…</conditions>` | lista `_preCondition` + config de mensajes |
| `<using kind="DUAL" />` | condición de tipo "using weapon kind" |
| combinaciones anidadas | `ConditionLogicAnd/Or/Not` |

El parser es `util/DocumentSkill.java` (detalle de mapeo tag→clase REQUIRES CODE VERIFICATION).

## 6. QUÉ OCURRE CUANDO FALLA

- Mensaje SystemMessage (por id, opcionalmente con nombre de skill) o texto plano.
- Retorno false → interrumpe el flujo ANTES de consumir MP/items o iniciar casting (ver CAST_FLOW: los costes se aplican después de las validaciones).

---

## 7. TRAZABILIDAD

| Elemento | Path |
|----------|------|
| Base | `mechanics/conditions/Condition.java` |
| Composición | `ConditionLogicAnd/Or/Not.java` |
| Evaluación | `mechanics/skill/Skill.java` ~L946–994 |
| Origen XML | `dist/game/data/stats/skills/*.xml` (`<conditions>`) |
| Parser | `util/DocumentSkill.java` |

---
**Status**: VERIFIED (base/evaluación/composición)  
**Verified**: 2026-08-23