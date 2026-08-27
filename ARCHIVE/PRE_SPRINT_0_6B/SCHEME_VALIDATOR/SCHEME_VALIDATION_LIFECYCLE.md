# SCHEME_VALIDATION_LIFECYCLE

**Project**: L2J Mobius CT 2.6 HighFive  
**Domain**: SKILLS, BUFFS  
**Status**: VERIFIED  
**Purpose**: Define when scheme validation should occur and what each stage should check.

---

## 1. Lifecycle stages

| Stage | Trigger | What to validate | Rationale |
|---|---|---|---|
| **Creation** | `_bbsscheme_create` or `createscheme` | Structural limits only: name, max schemes | Scheme starts empty; no skills to validate |
| **Editing** | Adding or removing a skill | Duplicate detection, slot counting, category warnings, structural limits | Early feedback; player's class may change later |
| **Execution** | Applying a scheme to a target | Class availability, skill level, weapon conditions, target compatibility, AbnormalType stacking | Only at execution is the definitive context known |
| **Periodic** | Login, subclass change, equipment change | Re-validate stored scheme against current player state | Catch invalidations caused by state changes |

---

## 2. Execution-time as primary validation

Execution time is the authoritative validation point because:

- The player's class may have changed since scheme creation.
- The player's equipment, which affects weapon/item conditions, is only known at execution time.
- The target (player or summon) is selected at execution time.
- `SkillTreeData.isSkillAllowed()` and `Skill.checkCondition()` require live `Player` and `Creature` contexts.

Edit-time validation can provide warnings but must not reject skills that might become valid under a different class or equipment state.

---

## 3. Validation dimensions by stage

| Dimension | Creation | Editing | Execution | Periodic |
|---|---|---|---|---|
| Name format | ERROR | ERROR | - | - |
| Max schemes | ERROR | ERROR | - | - |
| Max skills per scheme | - | ERROR | - | - |
| Duplicate skills | - | ERROR | WARNING | - |
| Slot counts | - | ERROR | WARNING | - |
| Passive skill | - | ERROR | ERROR | ERROR |
| Debuff in self scheme | - | WARNING | WARNING | WARNING |
| Class availability | - | WARNING | ERROR | ERROR |
| Skill level legality | - | WARNING | ERROR | ERROR |
| Weapon/item conditions | - | INFO | WARNING | WARNING |
| Target compatibility | - | WARNING | ERROR | ERROR |
| Same AbnormalType conflict | - | INFO | WARNING | WARNING |
| Cross AbnormalType conflict | - | SOURCE_REQUIRED | SOURCE_REQUIRED | SOURCE_REQUIRED |

---

## 4. Re-validation triggers

Re-validate a stored scheme when:

- Player changes class or subclass.
- Player equips or unequips a weapon or armor piece that affects skill conditions.
- A skill XML or skill tree update changes availability.
- The scheme is imported or loaded from a different system.

---

## 5. Implementation notes

- A validator implementation should accept a `ValidationContext` enum to apply stage-specific severity.
- Warnings at edit time become errors at execution time for the same issue.
- SOURCE_REQUIRED dimensions must not be implemented with hardcoded assumptions.

---

## 6. Cross-references

- Validator design: `SKILLS/SCHEME_VALIDATOR_DESIGN.md`
- SchemeBuffer analysis: `BUFFS/SCHEME_BUFFER_ANALYSIS.md`
- Community Board analysis: `BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md`
- System comparison: `BUFFS/SCHEME_SYSTEM_COMPARISON.md`
