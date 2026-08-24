# SKILL LEARNING

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Skills — adquisición/aprendizaje  
**Source of Truth**: `entity/actor/Player.java`, `mechanics/skill/enums/AcquireSkillType.java`, `data/xml/SkillTreeData.java`, `mechanics/skill/holders/SkillLearn.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (flujo y firmas; schema SQL fuera de alcance)

---

## 1. CATEGORÍAS REALES DE ADQUISICIÓN

`AcquireSkillType` (`mechanics/skill/enums/AcquireSkillType.java`):

```
CLASS, FISHING, PLEDGE, SUBPLEDGE, TRANSFORM, TRANSFER, SUBCLASS, COLLECT
+ getAcquireSkillType(int id)
```

No existen categorías "race" ni "hero" como tipos de adquisición en este enum.

## 2. Almacenamiento runtime

- Las skills conocidas viven en `Creature._skills: Map<Integer, Skill>` (heredado por Player/NPC/Summon).
- Player expone utilidades: `getAllSkills()`, `getKnownSkill(id)`.

## 3. Alta / baja / persistencia (Player)

| Método | Línea aprox. | Función |
|--------|--------------|---------|
| `addSkill(Skill skill, boolean storeInDb?)` | llamadas L2710/2719/2824/2858 | añade al mapa (y opcionalmente persiste) |
| `removeSkill(skill, ...)` | L2165/2219/2237/6808 | elimina (penaltis, wyvern breath…) |
| `restoreSkills()` (private) | ~L8171; invocado en L7347 | carga desde DB al login |
| `storeSkills(List<Skill>, int newClassIndex)` (private) | ~L8140 | guarda lote |
| `storeSkill(newSkill, oldSkill, classIndex)` (private) | ~L8097; usado L8021/10611 | guarda una |
| Constantes SQL | `DELETE_SKILL_SAVE`, `DELETE_CHAR_SKILLS` | limpieza previa al insert (solo conexión) |

> El schema exacto de tablas/columnas NO se documenta en esta fase (límite declarado).

## 4. Árboles de habilidades

- **`SkillTreeData`** (`data/xml/SkillTreeData.java`): singleton con árboles por clase/raza/etc.
  - Método verificado: `getAvailableAutoGetSkills(Player)` usado por `giveAvailableAutoGetSkills()` (L2847).
- **Holders**: `mechanics/skill/holders/SkillLearn.java` y `EnchantSkillLearn.java`.
- `SkillData.reload()` recarga también los Skill Trees (comentado en su código).

## 5. Entrega automática de skills

- `giveAvailableAutoGetSkills()`: obtiene `SkillLearn` disponibles y llama `addSkill(skill, true)`.
- Al subir de nivel / cambiar clase se recalculan (conexión con packets de aprendizaje — fuera de alcance).

## 6. Otras fuentes reales de skills (verificadas)

| Fuente | Mecanismo |
|--------|-----------|
| **Clan** | `_clan.addSkillEffects(this)` (Player L2754) |
| **Items** | `item.getSkills()` → `List<SkillHolder>` (L3524) — equipar/usar otorga |
| **Penaltis automáticos** | weight penalty (id 4270), weapon/armor grade penalty → `addSkill/removeSkill` dinámicos |
| **NPC** | skills definidas en su `NpcTemplate` (conexión — detalle en fase NPC avanzada) |
| **Summon** | skills del servidor/pet data (conexión SUMMON_SYSTEM) |
| **Uso desde AI** | `Playable.useMagic(Skill, forceUse, dontMove)` — abstracto en Playable L327 |

## 7. Evento relacionado

- `OnPlayerSkillLearn` (`mechanics/events/holders/actor/player/OnPlayerSkillLearn.java`) — existe; disparo exacto REQUIRES CODE VERIFICATION.

## 8. Flujo resumido (aprender)

```
Fuente (tree/clan/item/evento)
   ↓ validación propia de la fuente
Player.addSkill(skill, storeInDb)
   ↓ _skills.put(hash, skill)
si storeInDb → storeSkill(...) → DB
al login → restoreSkills() → DB → addSkill sin persistir
```

---
**Status**: VERIFIED (flujo/firmas) · schema DB fuera de alcance  
**Verified**: 2026-08-23