# SKILL TARGETING

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Skills — selección de objetivos  
**Source of Truth**: `mechanics/skill/targets/{TargetType,AffectScope,AffectObject}.java`, `handler/TargetHandler.java`, `dist/game/data/scripts/handlers/skill/targets/` (34 scripts), `mechanics/skill/Skill.getTargetList`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (estructura y catálogo)

---

## 1. DECISIÓN COMBINADA (respuesta directa)

¿Quién decide el target? **Una combinación**, verificada:

1. El **XML** declara la categoría (`<targetType>`, `<affectScope>`, `affectObject`).
2. El **packet/AI** aporta solo el *objetivo semilla* (`creature.getTarget()`).
3. `Skill.getTargetList(Creature caster, boolean onlyFirst)` toma esa semilla y delega en el **TargetHandler script** correspondiente al `targetType`, que calcula la lista final usando `affectScope`/`affectObject`.

Ni la Skill "decide sola", ni el AI reparte: el reparto final lo hace el handler script.

## 2. `TargetType` — 38 valores verificados

```
AREA, AREA_CORPSE_MOB, AREA_FRIENDLY, AREA_SUMMON, AREA_UNDEAD,
AURA, AURA_CORPSE_MOB, AURA_FRIENDLY,
BEHIND_AREA, BEHIND_AURA,
CLAN, CLAN_MEMBER, COMMAND_CHANNEL,
CORPSE, CORPSE_CLAN, CORPSE_MOB,
ENEMY_SUMMON, FLAGPOLE,
FRONT_AREA, FRONT_AURA,
GROUND, HOLY, NONE, ONE,
OWNER_PET, PARTY, PARTY_CLAN, PARTY_MEMBER, PARTY_NOTME, PARTY_OTHER,
PC_BODY, PET, SELF, SERVITOR, SUMMON,
TARGET_PARTY, UNDEAD, UNLOCKABLE
```

Cobertura de conceptos pedidos:
- único → `ONE`; self → `SELF`; área → `AREA*`/`AURA*`; party/clan → `PARTY*`/`CLAN*`/`COMMAND_CHANNEL`;
- enemigo/mascota → `ENEMY_SUMMON`, `PET`, `SERVITOR`, `SUMMON`, `OWNER_PET`;
- muertos/cadáver → `CORPSE*`; undead → `UNDEAD`, `AREA_UNDEAD`; ground → `GROUND`.

## 3. FILTROS: `AffectScope` y `AffectObject`

- Existen como enums en `mechanics/skill/targets/`.
- Refinan QUÉ entidades dentro del área (aliados/enemigos/muertos/undead…).
- Detalle de valores individuales no enumerado en esta fase → `REQUIRES CODE VERIFICATION` si se requiere catálogo completo.

## 4. HANDLERS SCRIPT (34)

- Ubicación real: `dist/game/data/scripts/handlers/skill/targets/*.java`.
- Registrados contra `handler/TargetHandler.java` (mismo patrón que EffectHandler).
- Cada script implementa la lógica de UN TargetType (ej.: cómo se expande un AURA, qué hace GROUND).

## 5. MÉTODO DE ENTRADA VERIFICADO

`Skill.getTargetList(Creature creature, boolean onlyFirst)` (Skill.java ~L996):
```java
WorldObject objTarget = creature.getTarget();
Creature target = objTarget != null && objTarget.isCreature() ? objTarget.asCreature() : null;
return getTargetList(creature, onlyFirst, target);   // delega al handler según targetType
```
Overloads adicionales aceptan objetivo explícito (usado por AI cuando castea sobre un target concreto).

## 6. RELACIÓN CON DEAD TARGETS

- Tipos CORPSE* operan sobre cadáveres (resurrect/sweep).
- Para skills ofensivas, un target muerto se filtra más tarde en `onMagicHitTimer`/`applyEffects` (ver CAST_FLOW §6 y EFFECT_SYSTEM).

## 7. TRAZABILIDAD

| Elemento | Path |
|----------|------|
| Enums | `mechanics/skill/targets/{TargetType,AffectScope,AffectObject}.java` |
| Registro runtime | `handler/TargetHandler.java` |
| Implementaciones | `dist/game/data/scripts/handlers/skill/targets/*.java` (34) |
| Punto de entrada | `mechanics/skill/Skill.getTargetList(...)` (~L996) |

---
**Status**: VERIFIED (catálogo/estructura) · valores internos de AffectScope/Object → REQUIRES  
**Verified**: 2026-08-23