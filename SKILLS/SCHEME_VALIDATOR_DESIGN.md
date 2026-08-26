# SCHEME_VALIDATOR_DESIGN

**Project**: L2J Mobius CT 2.6 HighFive  
**Domain**: SKILLS, BUFFS  
**Status**: VERIFIED  
**Purpose**: Define a reusable validator contract for scheme validation, with evidence levels for each validation dimension.

This document is design-only. Implementation is deferred to a future sprint.

---

## 1. Validator scope

The validator validates schemes. It does not cast skills, modify effects, or manage scheme storage. It accepts normalized input from either SchemeBuffer or Community Board and returns issues.

---

## 2. Proposed contract

```java
interface SchemeValidator {
    List<SchemeValidationIssue> validate(
        Player player,
        Creature target,
        List<SchemeSkillEntry> skills,
        ValidationContext context
    );
}

class SchemeSkillEntry {
    Skill skill;       // resolved Skill object
    int desiredLevel;  // level that will be applied
    int skillId;       // for messages
}

class SchemeValidationIssue {
    IssueSeverity severity;        // ERROR, WARNING, INFO
    String skillId;
    String skillName;
    String category;               // CLASS, LEVEL, WEAPON, TARGET, STACKING, CATEGORY
    String message;
}

enum ValidationContext { CREATION, EDITING, EXECUTION }
enum IssueSeverity { ERROR, WARNING, INFO }
```

---

## 3. Validation dimensions

### 3.1 Class availability

**Evidence**: VERIFIED.

API: `SkillTreeData.getInstance().isSkillAllowed(player, skill)` at `SkillTreeData.java:1332-1369`.

- Validates skill and level against class, race, transfer, common, fishing, and transformation trees.
- Must be checked at execution time because the player's class may have changed since scheme creation.

### 3.2 Skill level legality

**Evidence**: VERIFIED.

APIs: `Skill.getLevel()`, `SkillData.getInstance().getMaxLevel(skillId)`, `SkillTreeData.getCompleteClassSkillTree(class)`.

- Global max level can be checked against `SkillData`.
- Class-specific max level requires the complete class skill tree.
- `isSkillAllowed()` effectively accepts any level up to the class-specific max by using `Math.min(skill.getLevel(), maxLevel)`.

### 3.3 Weapon and item conditions

**Evidence**: VERIFIED with caveats.

API: `Skill.checkCondition(creature, target, itemOrWeapon)` at `Skill.java:946-994`.

- `_preCondition` (pass `false`) and `_itemPreCondition` (pass `true`) are private; only accessible through `checkCondition()`.
- The method sends `SystemMessage` on failure. A validator implementation must suppress or capture these messages.
- GM and fake-player bypasses apply.
- Only meaningful at execution time when the player has actual equipment.

### 3.4 Target compatibility

**Evidence**: VERIFIED.

API: `Skill.getTargetType()` returns `TargetType` enum.

- `SELF` skills applied to pet: warning/error.
- `PET`, `SERVITOR`, `SUMMON`, `OWNER_PET` skills applied to player: warning/error.
- `PARTY`/`CLAN` target types: informational note if used outside party/clan context.

### 3.5 Category classification

**Evidence**: VERIFIED.

APIs: `Skill.isDebuff()`, `isDance()`, `isToggle()`, `isTriggeredSkill()`, `isPassive()`, `isHealingPotionSkill()`, `is7Signs()`.

- Passives should never appear in a scheme.
- Debuffs in a self-buff scheme are semantically suspicious.
- Dance/song limits already enforced by SchemeBuffer at edit time, not by Community Board.

### 3.6 Same-AbnormalType stacking

**Evidence**: VERIFIED.

Source: `EffectList.add()` at `EffectList.java:1475-1524`.

- Engine groups continuous effects by `AbnormalType`.
- Replacement rule: incoming skill replaces existing one if `newSkill.abnormalLevel >= oldSkill.abnormalLevel`.
- A validator can pre-compute which skill survives when two skills share the same `AbnormalType`.

### 3.7 Cross-AbnormalType conflicts

**Evidence**: SOURCE_REQUIRED.

- No engine-level conflict map exists for different `AbnormalType` values.
- Each pair of skills with different `AbnormalType` requires individual XML analysis.
- A validator must not infer cross-type conflicts without evidence.

---

## 4. Validation severity guidelines

| Dimension | CREATION severity | EDITING severity | EXECUTION severity |
|---|---|---|---|
| Class availability | WARNING | WARNING | ERROR |
| Skill level | WARNING | WARNING | ERROR |
| Weapon condition | INFO | INFO | WARNING/ERROR |
| Target compatibility | WARNING | WARNING | ERROR |
| Passive in scheme | ERROR | ERROR | ERROR |
| Debuff in self scheme | WARNING | WARNING | WARNING |
| Same AbnormalType conflict | INFO | INFO | WARNING |
| Cross AbnormalType conflict | SOURCE_REQUIRED | SOURCE_REQUIRED | SOURCE_REQUIRED |

---

## 5. Decoupling pattern

```
SchemeBuffer                    CommunityBoard
     |                                |
ormalize to          normalize to
SchemeSkillEntry      SchemeSkillEntry
     |                                |
     +-------------+------------------+
                   |
            SchemeValidator
                   |
     List<SchemeValidationIssue>
                   |
            UI display / logging
```

The validator knows nothing about SchemeBuffer XML categories or Community Board bypass strings.

---

## 6. What belongs to implementation sprint

- Message suppression for `Skill.checkCondition()`.
- Concrete `SchemeValidator` implementation.
- Integration points in `SchemeBuffer` and `HomeBoard`.
- Cross-AbnormalType conflict data gathering.

---

## 7. Cross-references

- SchemeBuffer analysis: `BUFFS/SCHEME_BUFFER_ANALYSIS.md`
- Community Board analysis: `BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md`
- System comparison: `BUFFS/SCHEME_SYSTEM_COMPARISON.md`
- Validation lifecycle: `SKILLS/SCHEME_VALIDATION_LIFECYCLE.md`
- Skill semantics: `SKILLS/SKILL_SEMANTIC_REFERENCE.md`
- EffectList stacking: `SKILLS/EFFECT_SYSTEM.md` §5
