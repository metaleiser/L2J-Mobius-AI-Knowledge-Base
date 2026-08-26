# SCHEME_SYSTEM_COMPARISON

**Project**: L2J Mobius CT 2.6 HighFive  
**Domain**: BUFFS  
**Status**: VERIFIED  
**Purpose**: Compare SchemeBuffer NPC and Community Board scheme subsystems to identify shared responsibilities and implementation-specific concerns.

---

## 1. Implementation differences

| Responsibility | SchemeBuffer | Community Board |
|---|---|---|
| **Storage format** | `List<Integer>` (IDs only) | `"id,level;id,level"` string |
| **Persistence** | `SchemeBufferTable._schemesTable` + `buffer_schemes` table (valor confirmado: SOURCE `SchemeBufferTable.java` L64-L66; ver nota ⚠️) | `buffer_schemes` table (RUNTIME `HomeBoard.java` L167-169) |
| **Level resolution** | From `BuffSkillHolder` at execution time | Stored in scheme data |
| **Effector** | NPC (`SchemeBuffer`) | Player (`player`) |
| **Available skills** | `SchemeBufferSkills.xml` categories | `Config.COMMUNITY_AVAILABLE_BUFFS` |
| **Cost system** | Adena or custom item per skill/holder | `COMMUNITYBOARD_CURRENCY` × buff count |
| **Slot counting** | `isDance()` routing, dance/buff counters | `MAX_SCHEME_BUFFS = 24` hard cap |
| **Name validation** | Alphanumeric, ≤14 chars | Sanitize regex, ≤16 chars |
| **Max schemes** | `BUFFER_MAX_SCHEMES` | `CommunityMaxSchemes` (propiedad de `config/Custom/CommunityBoard.ini`, default `5`) |
| **Target selection** | Player or pet via bypass | Player or pet via `resolveTarget()` |

> **Fuente de verdad**: comparación verificada contra **SERVER_RUNTIME** (build 26/05/2024). El `HomeBoard` del baseline upstream SOURCE es otra generación **sin** subsystem de schemes → SOURCE/RUNTIME PATH_DIVERGENCE (detalle en `BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md` §1.1; attribution **PARTIAL**).
>
> ⚠️ **Persistencia NPC (CONFLICT / SOURCE_REQUIRED)**: la única copia de `SchemeBufferTable.java` en el workspace está en SOURCE (`java/org/l2jmobius/gameserver/data/`, L64-L66: `buffer_schemes`). AUDIT-006 había registrado la persistencia runtime del NPC como `custom_buff_schemes`. Sin acceso al core del build 26/05/2024 no se resuelve qué generación aplica al deploy actual → la celda NPC se conserva tal cual y el conflicto queda documentado.

---

## 2. Shared kernel

Both systems use the same runtime APIs:

- `SkillData.getInstance().getSkill(id, level)` — skill resolution
- `Skill.applyEffects(effector, target)` — skill application
- `Skill` object APIs: `isDance()`, `isDebuff()`, `isToggle()`, `isTriggeredSkill()`, `getAbnormalType()`, `getAbnormalLevel()`, `getTargetType()`
- `EffectList.add()` for stacking/replacement at execution time
- `SkillTreeData.getInstance().isSkillAllowed(player, skill)` — neither currently calls it, but both could

---

## 3. Shared responsibilities for a validator

A reusable validator can serve both systems because validation concerns are identical once skills are normalized to resolved `Skill` objects:

- Class/skill availability
- Skill level legality
- Weapon/item conditions
- Target compatibility
- Debuff/category classification
- Same-AbnormalType stacking

The systems differ only in how they produce the list of `(skillId, level)` pairs. The validator should accept normalized input:

```
Player player
Creature target
List<SchemeSkillEntry> entries  // each entry contains a resolved Skill
ValidationContext context
```

---

## 4. Implementation-specific concerns

| Concern | SchemeBuffer-specific | Community Board-specific |
|---|---|---|
| Level source | `BuffSkillHolder.getLevel()` | Parsed from stored string |
| Effector context | NPC equipment/stats | Player equipment/stats |
| Cost validation | `BUFFER_ITEM_ID` vs adena | `COMMUNITYBOARD_CURRENCY` |
| Available skill filter | `_availableBuffs.containsKey(id)` | `COMMUNITY_AVAILABLE_BUFFS.contains(id)` |
| Category model | XML categories (Buffs/Dances/Songs/etc.) | Hardcoded tab categories |

These concerns belong to each system's scheme loader, not to the validator.

---

## 5. Cross-references

- SchemeBuffer: `BUFFS/SCHEME_BUFFER_ANALYSIS.md`
- Community Board schemes: `BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md`
- Validator design: `SKILLS/SCHEME_VALIDATOR_DESIGN.md`
- Validation lifecycle: `SKILLS/SCHEME_VALIDATION_LIFECYCLE.md`
