# SKILL SEMANTIC REFERENCE

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Skills — referencia semántica central para Scheme Validator  
**Source of Truth**: `mechanics/skill/Skill.java`, `mechanics/skill/EffectScope.java`, `mechanics/skill/targets/{TargetType,AffectScope,AffectObject}.java`, `mechanics/effects/{AbstractEffect,EffectType,EffectFlag}.java`, `entity/actor/holders/creature/EffectList.java`  
**Verified**: 2026-08-26  
**Status**: VERIFIED (firmas, enums y reglas contra SOURCE)

> Propósito: proporcionar a los micro-sprints 2.6 (Scheme Validator) y 2.7 (Community Board) una capa semántica verificable para cada skill, sin reemplazar los documentos arquitectónicos existentes.

---

## 1. Skill Identity

| Propiedad | Clase/Field | Semántica |
|---|---|---|
| `id` | `Skill._id` L72 | Identificador numérico de la skill. |
| `level` | `Skill._level` L74 | Nivel de la skill. |
| `displayId` | `Skill._displayId` L76 | ID mostrado al cliente. |
| `displayLevel` | `Skill._displayLevel` L78 | Nivel mostrado al cliente. |
| `name` | `Skill._name` L80 | Nombre interno. |

Una `Skill` en runtime **es la definición**: hay una instancia por cada nivel/enchant (SkillData la cachea).

---

## 2. Skill Categories

`EffectList.getEffectList(Skill)` enruta cada skill a una de seis colas (`EffectList.java` L211-241):

| Categoría | Condición SOURCE | Cola | Semántica |
|---|---|---|---|
| **Passive** | `Skill.isPassive()` (operateType=P) | `_passives` | Siempre activa; sin slot. |
| **Debuff** | `Skill.isDebuff()` (`_isDebuff`) | `_debuffs` | Efectos negativos; no cuentan contra slots normales. |
| **Triggered** | `Skill.isTriggeredSkill()` (`_isTriggeredSkill`) | `_triggered` | Buffs de activación; slot separado. |
| **Dance/Song** | `Skill.isDance()` (`_magic == 3`) | `_dances` | Cubre ambas (`isSong()` no existe como método separado). |
| **Toggle** | `Skill.isToggle()` (operateType=T) | `_toggles` | Buff encendido/apagado; no cuentan contra slots normales. |
| **Normal buff** | default | `_buffs` | Resto de efectos continuos. |

Además:
- `Skill.isPhysical()` → `_magic == 0`
- `Skill.isMagic()` → `_magic == 1`
- `Skill.isStatic()` → `_magic == 2`
- `Skill.isDance()` → `_magic == 3`

---

## 3. Beneficial vs Harmful Semantics

**No existe un único flag "beneficial"**. La determinación es multicapa y debe combinarse:

### Capa 1 — Skill-level explícito
| Método | Fuente | Semántica |
|---|---|---|
| `isDebuff()` | `_isDebuff` (XML `isDebuff="true"`) L194 | Debuff declarado explícitamente. |
| `is7Signs()` | `_isSevenSigns` L883 | Skill de Seven Signs; exento de slots. |
| `isRecoveryHerb()` | `_abnormalType == HP_RECOVER` L647 | Herb de recuperación. |

### Capa 2 — Negatividad derivada
| Método | Fuente | Semántica |
|---|---|---|
| `hasNegativeEffect()` | `_effectPoint < 0 && targetType != SELF` L941 | Usada para bloqueos de mount, peace, siege, invul. |
| `getEffectPoint()` | `_effectPoint` L847 | Negativo ⇒ tendencia negativa. |

### Capa 3 — Effect-level
| Método | Fuente | Semántica |
|---|---|---|
| `AbstractEffect.getEffectType()` | `EffectType` enum | BUFF, DEBUFF, STUN, SLEEP, HEAL, etc. |
| `Skill.hasEffectType(...)` | Itera `_effects` | Verifica si la skill contiene un tipo de efecto concreto. |

### Capa 4 — Control flags
| Método | Fuente | Semántica |
|---|---|---|
| `EffectFlag.*` | `EffectFlag` enum | Estados de control (MUTED, SLEEP, STUNNED, CONFUSED, etc.). |

### Regla de decisión para el Scheme Validator
```
if isDebuff()                    → HARMFUL
else if hasNegativeEffect()      → LIKELY_HARMFUL (SOURCE_REQUIRED si targetType no es enemigo)
else if effectType == BUFF/HEAL  → BENEFICIAL
else if effectType == DEBUFF     → HARMFUL
else if isDance() || isToggle()  → BENEFICIAL_BY_DEFAULT (verificar skill XML)
else                              → NEUTRAL / REQUIRES_REVIEW
```

---

## 4. Target Classification

### 4.1 TargetType — 38 valores
Catálogo completo en [`SKILL_TARGETING.md`](SKILL_TARGETING.md) §2.

### 4.2 AffectScope — 15 valores (geometría)
| Valor | Semántica SOURCE |
|---|---|
| `SINGLE` | Un solo objetivo. |
| `POINT_BLANK` | Área desde el caster. |
| `RANGE` | Área desde el target seleccionado. |
| `RING_RANGE` | Anillo alrededor del target. |
| `FAN` | Área en abanico. |
| `SQUARE` | Cuadrado desde el target. |
| `SQUARE_PB` | Cuadrado desde el caster. |
| `PARTY` | Miembros de party. |
| `PLEDGE` | Miembros de clan. |
| `PARTY_PLEDGE` | Party + clan. |
| `DEAD_PLEDGE` | Clan muertos. |
| `VALAKAS_SCOPE` | Especial Valakas. |
| `WYVERN_SCOPE` | Especial wyvern. |
| `STATIC_OBJECT_SCOPE` | Objetos estáticos. |
| `NONE` | No afecta nada. |

### 4.3 AffectObject — 10 valores (filtro de entidad)
| Valor | Semántica SOURCE |
|---|---|
| `ALL` | Todo. |
| `CLAN` | Clan. |
| `FRIEND` | Amistoso. |
| `NOT_FRIEND` | No amistoso. |
| `NOE` | No Ofensa/Enemigo (según contexto). |
| `OBJECT_DEAD_NPC_BODY` | Cadáveres NPC. |
| `UNDEAD_REAL_ENEMY` | Undead enemigo. |
| `INVISIBLE` | Invisibles. |
| `HIDDEN_PLACE` | Lugares ocultos. |
| `WYVERN_OBJECT` | Objetos wyvern. |


---

## 5. Stacking Rules

Fuente: `EffectList.add(BuffInfo)` L1414-1567.

### 5.1 Agrupación
- Efectos continuos se agrupan por **`AbnormalType`** en `_stackedEffects: Map<AbnormalType, BuffInfo>`.

### 5.2 Reemplazo
- Si llega una skill con el **mismo** `AbnormalType`:
  - Y `newSkill.abnormalLevel >= oldSkill.abnormalLevel` → reemplaza.
  - Y `newSkill.abnormalLevel < oldSkill.abnormalLevel` → **no se añade**.

### 5.3 Herb / abnormalInstant
- Si la nueva skill es `isAbnormalInstant()` (herbs):
  - La anterior se marca `setInUse(false)` y se remueven stats, pero sigue "viva" (ticks continúan).
- Si la anterior es `isAbnormalInstant()` y la nueva no → se remueve la anterior.

### 5.4 Sin AbnormalType
- Si `skill.getAbnormalType().isNone()` → se remueve cualquier instancia previa de la **misma skill ID**.

### 5.5 Cross-AbnormalType
> **SOURCE_REQUIRED**: la SOURCE no demuestra reglas generales de conflicto entre `AbnormalType` diferentes. Cada par de skills debe verificarse contra su XML.

---

## 6. Buff Slot Management

Fuente: `EffectList.add()` L1531-1562.

Solo se comprueban slots si **NO** es debuff, toggle, 7Signs ni stacked (`doesStack`).

| Tipo | Límite SOURCE |
|---|---|
| Dance/Song | `PlayerConfig.DANCES_MAX_AMOUNT` |
| Triggered | `PlayerConfig.TRIGGERED_BUFFS_MAX_AMOUNT` |
| Buff normal | `_owner.getStat().getMaxBuffCount()` |
| Healing potion | Exento (`isHealingPotionSkill()`) |
| Debuff | Exento |
| Toggle | Exento |
| 7Signs | Exento |

Si se excede el límite, se remueve el buff más antiguo de la cola correspondiente.

---

## 7. Restrictions & Applicability

### 7.1 Pre-cast (`Skill.checkCondition` L946-994)
- GM bypass (`GM_SKILL_RESTRICTION`).
- Fake player bypass.
- Mount restriction para skills con `hasNegativeEffect()` (`MountEnabledSkillList`).
- Lista `_preCondition` / `_itemPreCondition` (88 clases de conditions).

### 7.2 Zonas
- Debuffs bloqueados en zonas PEACE (`target.isInsideZone(ZoneId.PEACE)`).
- Debuffs con `hasNegativeEffect()` restringidos en siege según estado del player.

### 7.3 Invulnerabilidad
- `effected.isInvul()` o GM sin `canGiveDamage()` → skills negativas no aplican (L1347).
- `effected.isInvulAgainst(_id, _level)` → skill totalmente inmune.

### 7.4 Pledge / class
- `minPledgeClass` (L867) — mínimo rango de clan.
- **Class ID restrictivo por skill NO existe en Skill.java** — las restricciones de clase vienen del SkillTree (qué skills puede aprender), no del skill en sí.

---

## 8. Scheme Validator Evidence Matrix

| Dimensión | Estado | Evidencia SOURCE |
|---|---|---|
| skill identity | ✅ | Skill fields L72-80 |
| skill level | ✅ | Skill._level |
| target | ✅ | TargetType + AffectScope + AffectObject |
| beneficial/harmful | ✅ | 4-capas documentadas arriba |
| duration | ✅ | BuffInfo._abnormalTime |
| stacking | ✅ | EffectList.add() L1414-1567 |
| conflicts (mismo AbnormalType) | ✅ | `_stackedEffects` |
| conflicts (cross-AbnormalType) | ⚠️ SOURCE_REQUIRED | Necesita skill XML |
| applicability | ✅ | checkCondition + zones + invul |
| class restriction | ⚠️ PARTIAL | Solo pledge; aprendizaje vía SkillTree |
| weapon restriction | ⚠️ PARTIAL | Conditions `<using kind="..."/>` |
| summon/pet restriction | ✅ | TargetType PET/SERVITOR/SUMMON/OWNER_PET |
| party/self restriction | ✅ | TargetType PARTY/SELF/etc |
| augmentation | ⚠️ SOURCE_REQUIRED | `OptionSkillType`, item skills |
| skill source | ✅ | AcquireSkillType |
| skill availability | ✅ | SkillTreeData |
| effect compatibility | ✅ | EffectType + EffectFlag |
| ordering/priority | ✅ | abnormalLevel + reemplazo |

---

## 9. Cross-links

- Motor de aplicación de efectos → [`EFFECT_SYSTEM.md`](EFFECT_SYSTEM.md)
- Targeting detallado → [`SKILL_TARGETING.md`](SKILL_TARGETING.md)
- Modelo de datos → [`SKILL_DATA_MODEL.md`](SKILL_DATA_MODEL.md)
- Catálogo completo de AbnormalType → [`../REFERENCE/ABNORMAL_TYPE_CATALOG.md`](../REFERENCE/ABNORMAL_TYPE_CATALOG.md)
- Aprendizaje → [`SKILL_LEARNING.md`](SKILL_LEARNING.md)
- Casteo → [`CAST_FLOW.md`](CAST_FLOW.md)
- Arquitectura → [`SKILL_ARCHITECTURE.md`](SKILL_ARCHITECTURE.md)
- Diseño del Scheme Validator (consolidado) → [`SCHEME_VALIDATION.md`](SCHEME_VALIDATION.md)
- Ciclo de vida de validación (consolidado en SCHEME_VALIDATION.md) → [`SCHEME_VALIDATION.md`](SCHEME_VALIDATION.md)
- Análisis de SchemeBuffer → [`../BUFFS/SCHEME_BUFFER_ANALYSIS.md`](../BUFFS/SCHEME_BUFFER_ANALYSIS.md)
- Análisis de Community Board schemes → [`../BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md`](../BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md)
- Comparación de sistemas de scheme → [`../BUFFS/SCHEME_SYSTEM_COMPARISON.md`](../BUFFS/SCHEME_SYSTEM_COMPARISON.md)
- Requisitos del validador → [`../00_PROJECT/ROADMAP.md`](../00_PROJECT/ROADMAP.md)

---

**Fuente**: `mechanics/skill/Skill.java`, `mechanics/effects/*`, `mechanics/skill/targets/*`, `entity/actor/holders/creature/EffectList.java`  
**Status**: VERIFIED (con cross-AbnormalType y augmentation marcados SOURCE_REQUIRED)
