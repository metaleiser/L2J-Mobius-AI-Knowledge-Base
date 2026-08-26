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

## 3. `AffectScope` — 15 valores (geometría)

Source: `mechanics/skill/targets/AffectScope.java`.

| # | Value | Semántica |
|---:|---|---|
| 1 | `NONE` | No afecta nada. |
| 2 | `SINGLE` | Un solo objetivo. |
| 3 | `POINT_BLANK` | Área desde el caster. |
| 4 | `RANGE` | Área desde el target seleccionado. |
| 5 | `RING_RANGE` | Anillo alrededor del target. |
| 6 | `FAN` | Área en abanico. |
| 7 | `SQUARE` | Cuadrado desde el target. |
| 8 | `SQUARE_PB` | Cuadrado desde el caster. |
| 9 | `PARTY` | Miembros de party. |
| 10 | `PLEDGE` | Miembros de clan. |
| 11 | `PARTY_PLEDGE` | Party + clan. |
| 12 | `DEAD_PLEDGE` | Clan muertos. |
| 13 | `VALAKAS_SCOPE` | Especial Valakas. |
| 14 | `WYVERN_SCOPE` | Especial wyvern. |
| 15 | `STATIC_OBJECT_SCOPE` | Objetos estáticos. |

## 4. `AffectObject` — 10 valores (filtro de entidad)

Source: `mechanics/skill/targets/AffectObject.java`.

| # | Value | Semántica |
|---:|---|---|
| 1 | `ALL` | Cualquier entidad. |
| 2 | `CLAN` | Clan. |
| 3 | `FRIEND` | Amistoso. |
| 4 | `NOT_FRIEND` | No amistoso. |
| 5 | `NOE` | No Ofensa/Enemigo (según contexto). |
| 6 | `OBJECT_DEAD_NPC_BODY` | Cadáveres NPC. |
| 7 | `UNDEAD_REAL_ENEMY` | Undead enemigo. |
| 8 | `INVISIBLE` | Entidades invisibles. |
| 9 | `HIDDEN_PLACE` | Lugares ocultos. |
| 10 | `WYVERN_OBJECT` | Objetos wyvern. |

## 5. HANDLERS SCRIPT (34)

- Ubicación real: `dist/game/data/scripts/handlers/skill/targets/*.java`.
- Registrados contra `handler/TargetHandler.java` (mismo patrón que EffectHandler).
- Cada script implementa la lógica de UN TargetType (ej.: cómo se expande un AURA, qué hace GROUND).

## 6. MÉTODO DE ENTRADA VERIFICADO

`Skill.getTargetList(Creature creature, boolean onlyFirst)` (Skill.java ~L996):
```java
WorldObject objTarget = creature.getTarget();
Creature target = objTarget != null && objTarget.isCreature() ? objTarget.asCreature() : null;
return getTargetList(creature, onlyFirst, target);   // delega al handler según targetType
```
Overloads adicionales aceptan objetivo explícito (usado por AI cuando castea sobre un target concreto).

## 7. RELACIÓN CON DEAD TARGETS

- Tipos CORPSE* operan sobre cadáveres (resurrect/sweep).
- Para skills ofensivas, un target muerto se filtra más tarde en `onMagicHitTimer`/`applyEffects` (ver CAST_FLOW §6 y EFFECT_SYSTEM).

## 8. TRAZABILIDAD

| Elemento | Path |
|----------|------|
| Enums | `mechanics/skill/targets/{TargetType,AffectScope,AffectObject}.java` |
| Registro runtime | `handler/TargetHandler.java` |
| Implementaciones | `dist/game/data/scripts/handlers/skill/targets/*.java` (34) |
| Punto de entrada | `mechanics/skill/Skill.getTargetList(...)` (~L996) |

---
**Status**: VERIFIED (catálogos TargetType/AffectScope/AffectObject y estructura)  
**Verified**: 2026-08-26