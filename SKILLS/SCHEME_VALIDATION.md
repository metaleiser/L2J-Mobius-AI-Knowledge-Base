# SCHEME VALIDATION

**Project**: L2J Mobius CT 2.6 HighFive  
**Domain**: SKILLS, BUFFS  
**Status**: VERIFIED (design-only; implementation deferred to future sprint)  
**Sprint 0.6B**: Merged from \SCHEME_VALIDATOR_DESIGN.md\ + \SCHEME_VALIDATION_LIFECYCLE.md\

---

## 1. Validator scope

The validator validates schemes. It does **not** cast skills, modify effects, or manage scheme storage. It accepts normalized input from either SchemeBuffer or Community Board and returns issues.

---

## 2. Proposed contract

\\\java
interface SchemeValidator {
    List<SchemeValidationIssue> validate(
        Player player,
        Creature target,
        List<SchemeSkillEntry> skills,
        ValidationContext context
    );
}

class SchemeSkillEntry {
    Skill skill;
    int desiredLevel;
    int skillId;
}

class SchemeValidationIssue {
    IssueSeverity severity;    // ERROR, WARNING, INFO
    String skillId;
    String skillName;
    String category;           // CLASS, LEVEL, WEAPON, TARGET, STACKING, CATEGORY
    String message;
}

enum ValidationContext { CREATION, EDITING, EXECUTION }
enum IssueSeverity { ERROR, WARNING, INFO }
\\\

---

## 3. Validation dimensions

| Dimension | Evidence | APIs |
|-----------|----------|------|
| Class availability | VERIFIED | \SkillTreeData.isSkillAllowed(player, skill)\ |
| Skill level legality | VERIFIED | \Skill.getLevel()\, \SkillData.getMaxLevel()\, \SkillTreeData.getCompleteClassSkillTree()\ |
| Weapon/item conditions | VERIFIED (caveats) | \Skill.checkCondition(creature, target, itemOrWeapon)\ — sends SystemMessage, must capture |
| Target compatibility | VERIFIED | \Skill.getTargetType()\ — SELF→pet warning, PET→player warning |
| Category classification | VERIFIED | \Skill.isDebuff()\, \isDance()\, \isToggle()\, \isPassive()\, etc. |
| Same-AbnormalType stacking | VERIFIED | \EffectList.add()\ — replacement by \bnormalLevel\ |
| Cross-AbnormalType conflicts | SOURCE_REQUIRED | No engine-level conflict map exists |

---

## 4. Lifecycle stages

| Stage | Trigger | What to validate |
|-------|---------|-----------------|
| **Creation** | \_bbsscheme_create\ or \createscheme\ | Name format, max schemes |
| **Editing** | Adding/removing a skill | Duplicates, slots, category warnings, structural limits |
| **Execution** | Applying scheme to target | Class, level, weapons, target, stacking |
| **Periodic** | Login, subclass change, equipment change | Re-validate stored scheme against current state |

**Execution time is the authoritative validation point** — only then is class, equipment, and target known.

---

## 5. Severity matrix

| Dimension | Creation | Editing | Execution | Periodic |
|-----------|----------|---------|-----------|----------|
| Name format | ERROR | ERROR | — | — |
| Max schemes | ERROR | ERROR | — | — |
| Max skills per scheme | — | ERROR | — | — |
| Duplicate skills | — | ERROR | WARNING | — |
| Slot counts | — | ERROR | WARNING | — |
| Passive skill | — | ERROR | ERROR | ERROR |
| Debuff in self scheme | — | WARNING | WARNING | WARNING |
| Class availability | — | WARNING | ERROR | ERROR |
| Skill level legality | — | WARNING | ERROR | ERROR |
| Weapon/item conditions | — | INFO | WARNING | WARNING |
| Target compatibility | — | WARNING | ERROR | ERROR |
| Same AbnormalType conflict | — | INFO | WARNING | WARNING |
| Cross AbnormalType conflict | — | SOURCE_REQUIRED | SOURCE_REQUIRED | SOURCE_REQUIRED |

---

## 6. Re-validation triggers

- Player changes class or subclass
- Player equips/unequips weapon or armor affecting skill conditions
- Skill XML or skill tree update changes availability
- Scheme is imported from a different system

---

## 7. Implementation notes

- A validator implementation should accept a \ValidationContext\ enum to apply stage-specific severity.
- Warnings at edit time become errors at execution time for the same issue.
- SOURCE_REQUIRED dimensions must not be implemented with hardcoded assumptions.
- \Skill.checkCondition()\ sends SystemMessage on failure — a validator must suppress or capture these messages.
- Passives should never appear in a scheme (ERROR across all stages).
- The validator knows nothing about SchemeBuffer XML categories or Community Board bypass strings.

---

## 8. Cross-references

- SchemeBuffer analysis: [BUFFS/SCHEME_BUFFER_ANALYSIS.md](../BUFFS/SCHEME_BUFFER_ANALYSIS.md)
- Community Board analysis: [BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md](../BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md)
- System comparison: [BUFFS/SCHEME_SYSTEM_COMPARISON.md](../BUFFS/SCHEME_SYSTEM_COMPARISON.md)
- Skill semantics: [SKILL_SEMANTIC_REFERENCE.md](SKILL_SEMANTIC_REFERENCE.md)
- EffectList stacking: [EFFECT_SYSTEM.md](EFFECT_SYSTEM.md) §5
- Design archive: [ARCHIVE/PRE_SPRINT_0_6B/SCHEME_VALIDATOR/SCHEME_VALIDATOR_DESIGN.md](../ARCHIVE/PRE_SPRINT_0_6B/SCHEME_VALIDATOR/SCHEME_VALIDATOR_DESIGN.md)
- Lifecycle archive: [ARCHIVE/PRE_SPRINT_0_6B/SCHEME_VALIDATOR/SCHEME_VALIDATION_LIFECYCLE.md](../ARCHIVE/PRE_SPRINT_0_6B/SCHEME_VALIDATOR/SCHEME_VALIDATION_LIFECYCLE.md)

---

**Documentation only** — no implementation in server/source code.  
**Status**: VERIFIED (design-only)
