# SCHEME_BUFFER_ANALYSIS

**Project**: L2J Mobius CT 2.6 HighFive  
**Domain**: BUFFS  
**System**: SchemeBuffer NPC  
**Status**: VERIFIED  
**Source**: `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive/java/org/l2jmobius/gameserver/entity/actor/instance/SchemeBuffer.java`, `UPSTREAM/.../data/SchemeBufferTable.java`

---

## 1. Component overview

The SchemeBuffer is an NPC class that provides player-defined buff schemes and manual buff casting. It is implemented as a Java class extending `Npc` and uses `SchemeBufferTable` for persistence and skill lookup.

---

## 2. Scheme storage model

Schemes are stored in `SchemeBufferTable._schemesTable` as:

```
Map<Integer, Map<String, List<Integer>>>
objectId → schemeName → List<skillId>
```

Only **skill IDs** are stored. Skill levels are not persisted per scheme. The level is resolved at execution time from the `BuffSkillHolder` loaded from `SchemeBufferSkills.xml`.

---

## 3. Skill resolution

Available skills are loaded from `SchemeBufferSkills.xml` into `SchemeBufferTable._availableBuffs` and `_availableBuffsByType`. Each entry contains:

- `skillId`
- `level`
- `price`
- `category`
- `description`

At execution time:

```java
BuffSkillHolder holder = SchemeBufferTable.getInstance().getAvailableBuff(skillId);
Skill skill = SkillData.getInstance().getSkill(skillId, holder.getLevel());
```

---

## 4. Scheme creation flow

Handler: `createscheme` in `SchemeBuffer.onBypassFeedback()` L292-332.

Validated:
- Name length ≤ 14 characters
- Name is alphanumeric after normalization
- Maximum schemes count (`BUFFER_MAX_SCHEMES`)
- Duplicate scheme name

Not validated:
- Skill availability
- Class restrictions
- Level restrictions
- Weapon restrictions
- Debuff/category restrictions

The scheme is created empty. Skill validation happens only at edit time.

---

## 5. Scheme editing flow

Handler: `skillselect` / `skillunselect` in `SchemeBuffer.onBypassFeedback()` L246-282.

When adding a skill:

1. Resolve skill via `SkillData.getSkill(skillId, 1)` — level 1 is used only for classification.
2. Check `skill.isDance()`:
   - If dance: count against `PlayerConfig.DANCES_MAX_AMOUNT`
   - If not dance: count against `player.getStat().getMaxBuffCount()`
3. Prevent duplicate skill IDs in the same scheme.

Existing validation is limited to slot counting and duplicate prevention. No class, level, weapon, target, or debuff validation is performed.

---

## 6. Scheme execution flow

Handler: `givebuffs` in `SchemeBuffer.onBypassFeedback()` L143-199.

1. Retrieve scheme via `SchemeBufferTable.getInstance().getScheme(objectId, schemeName)`.
2. Calculate fee via `getFee(scheme)` using `BuffSkillHolder` prices.
3. Deduct cost from player (adena or configured item).
4. For each `skillId` in the scheme:
   - Resolve `BuffSkillHolder`
   - Resolve `Skill` at holder level
   - Call `skill.applyEffects(this, target)` where `this` is the SchemeBuffer NPC

The NPC is the effector, not the player. This affects `checkCondition()` evaluation and effect success calculations.

---

## 7. Manual cast and autobuff flows

- `manual` / `castbuff`: single skill cast from a category page. Uses same resolution and `applyEffects(this, target)`.
- `autobuff`: applies `MAGE_GROUP` or `FIGHTER_GROUP` category skills based on `player.isMageClass()`. Same resolution and application path.

---

## 8. What is not validated

| Validation dimension | Status | Reason |
|---|---|---|
| Class availability | Not validated | `SkillTreeData.isSkillAllowed()` not called |
| Skill level legality | Not validated | Level resolved from XML, not checked against player class |
| Weapon/item conditions | Not validated | `Skill.checkCondition()` not called |
| Target compatibility | Not validated | Target type not checked against selected target |
| Debuff/category | Not validated | Only `isDance()` used for slot routing |
| AbnormalType conflicts | Not validated | Same-type stacking not checked at edit time |

---

## 9. Cross-references

- Skill semantics and APIs: `SKILLS/SKILL_SEMANTIC_REFERENCE.md`
- SchemeValidator design: `SKILLS/SCHEME_VALIDATOR_DESIGN.md`
- Validation lifecycle: `SKILLS/SCHEME_VALIDATION_LIFECYCLE.md`
- Community Board scheme system: `BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md`
- System comparison: `BUFFS/SCHEME_SYSTEM_COMPARISON.md`
