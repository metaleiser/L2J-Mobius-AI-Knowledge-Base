# COMMUNITY_BOARD_SCHEME_ANALYSIS

**Project**: L2J Mobius CT 2.6 HighFive  
**Domain**: BUFFS  
**System**: Community Board HomeBoard scheme subsystem  
**Status**: VERIFIED (hechos verificables contra **SERVER_RUNTIME**) · atribución y paridad de interfaz: PARTIAL  
**Analyzed implementation**: **SERVER_RUNTIME** — `L2J_Mobius_CT_2.6_HighFive/game/data/scripts/handlers/communityboard/HomeBoard.java`  
**Upstream SOURCE counterpart**: `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive/dist/game/data/scripts/handlers/bypass/communityboard/HomeBoard.java` (generación distinta, SIN motor de schemes — ver §1.1)

---

## 1. Component overview

The Community Board scheme system is implemented in `HomeBoard`, a community board handler registered for `_bbshome` / `_bbstop` and custom buffer commands. Schemes are stored in the **`buffer_schemes`** database table (persistencia verificada en runtime `HomeBoard.java` L167-169) and edited through the Community Board HTML interface.

Delivery chain verificada (runtime `HomeBoard.java` L633-640): `returnHtml` → `CommunityBoardHandler.separateAndSend(returnHtml, player)`.

### 1.1 Divergencia SOURCE/RUNTIME (PATH_DIVERGENCE — attribution PARTIAL)

- El motor de schemes **existe solo en SERVER_RUNTIME** (`game/data/scripts/handlers/communityboard/HomeBoard.java`, build 26/05/2024).
- En el baseline upstream **SOURCE** (`e2518ab`), el homólogo `dist/game/data/scripts/handlers/bypass/communityboard/HomeBoard.java` es una **generación distinta** sin subsystem de schemes; su interfaz `IParseBoardHandler` no coincide con la del handler runtime → **mismatch de interfaz respecto al baseline marcado PARTIAL**.
- El origen exacto de la personalización runtime **no se afirma como probado** (clasificación **PARTIAL**; posible port/adaptación de otra fuente, sin evidencia concluyente en este sprint).
- Registrado también en [GAPS.md](../GAPS.md) bajo COMMUNITY BOARD (**PARTIAL**) como SOURCE/RUNTIME PATH_DIVERGENCE.

---

## 2. Scheme storage model

Storage: **`buffer_schemes`** table with columns `object_id`, `scheme_name`, `skills`.

The `skills` column stores a delimited string:

```
"id,level;id,level;id,level"
```

Both **skill ID and level** are persisted. This differs from SchemeBuffer, which stores only IDs.

---

## 3. Loading and parsing

- `loadSchemes(objectId)` L989-1009: returns `Map<String, String>` (schemeName → skills string).
- `loadSchemeSkills(objectId, schemeName)` L1017-1022: returns the skills string for one scheme.
- `parseSchemeSkills(skills)` L937-945: splits by `;` into a list of `"id,level"` entries.
- `addBuffToScheme()` L953-967: appends `"id,level"` to the stored string, checks for duplicates and max scheme size.
- `removeBuffFromScheme()` L975-982: removes a matching `"id,level"` entry.

---

## 4. Scheme creation and recording

Handler: `_bbsscheme_create` in `HomeBoard.parseCommunityBoardCommand()` L372-404.

1. Sanitize name: letters, digits, underscore, dash, max 16 characters.
2. Check maximum schemes count (`CommunityMaxSchemes` — propiedad de `config/Custom/CommunityBoard.ini`, default `5`; carga verificada en runtime `HomeBoard.java` L1090-1102; INI físico verificado).
3. Save empty scheme to database.
4. Set `VAR_ACTIVE_SCHEME` player variable to enable recording mode.

In recording mode, every individual buff click via **`_bbsbuffsingle`** (runtime `HomeBoard.java` L285-330) is added to the active scheme through `addBuffToScheme()` (llamada verificada en **L319**).

> ⚠️ Corrected (Micro-Sprint 2.7): **`_bbsbuff` NO realiza grabación de esquemas** — es únicamente el hook de aplicación directa (§6). El único punto de grabación verificado es `_bbsbuffsingle`.

---

## 5. Scheme execution flow

Handler: `_bbsscheme_apply` in `HomeBoard.parseCommunityBoardCommand()` L444-501.

1. Sanitize scheme name.
2. Load skills string from DB.
3. Parse into `"id,level"` entries.
4. For each entry:
   - Split by `,` into id and level
   - Resolve `Skill` via `SkillData.getInstance().getSkill(id, level)`
   - Filter against `Config.COMMUNITY_AVAILABLE_BUFFS`
5. Check currency: `COMMUNITYBOARD_CURRENCY` × skill count.
6. Deduct currency.
7. For each resolved skill:
   - Call `skill.applyEffects(player, target)` where `player` is the effector
   - Optionally send `MagicSkillUse` packet if `COMMUNITYBOARD_CAST_ANIMATIONS` is enabled

The **player** is the effector, not an NPC. This means weapon/item conditions in `Skill.checkCondition()` are evaluated against the player's actual equipment.

---

## 6. Individual buff flow (`_bbsbuff` — NO graba esquemas)

Handler: `_bbsbuff` in `HomeBoard.parseCommunityBoardCommand()` L516-556. Aplica buffs directamente y **no persiste nada en los schemes**; la grabación ocurre exclusivamente en `_bbsbuffsingle` (§4, L285-330).

1. Parse bypass string: `id,level;id,level;...;page`
2. Check currency.
3. For each entry:
   - Resolve `Skill`
   - Filter against `COMMUNITY_AVAILABLE_BUFFS`
   - Call `skill.applyEffects(player, target)`

Same `applyEffects` application path as scheme casting, but **without any scheme write** — recording lives exclusively in `_bbsbuffsingle` (L285-330, `addBuffToScheme()` at L319).

---

## 7. Existing validation

| Validation | Status |
|---|---|
| Name sanitization | Yes |
| Max schemes | Yes |
| Max buffs per scheme (`MAX_SCHEME_BUFFS = 24`) | Yes |
| Duplicate entries | Yes |
| `COMMUNITY_AVAILABLE_BUFFS` whitelist | Yes |
| Currency check | Yes |
| Class availability | No |
| Skill level legality | No |
| Weapon/item conditions | No |
| Target compatibility | No |
| Debuff/category | No |
| AbnormalType conflicts | No |

---

## 8. Critical difference from SchemeBuffer

| Aspect | SchemeBuffer | Community Board |
|---|---|---|
| Storage | In-memory + DB, IDs only | DB, `id,level` pairs |
| Level resolution | From `BuffSkillHolder` XML | Stored in scheme |
| Effector | NPC (`SchemeBuffer`) | Player |
| Available skills | `SchemeBufferSkills.xml` categories | `COMMUNITY_AVAILABLE_BUFFS` config |
| Cost | Per skill / static | Per buff count |
| Slot counting at edit | Dance/buff counters | Hard cap 24 |

---

## 9. Runtime anchors verificados (Micro-Sprint 2.7)

| Ancla en RUNTIME `HomeBoard.java` | Líneas | Verificación |
|---|---|---|
| Constantes SQL sobre tabla `buffer_schemes` | L167-169 | VERIFIED (sprint 2.7) |
| Registro bypass `_bbsbuffsingle` | L83 | VERIFIED (sprint 2.7) |
| Handler `_bbsbuffsingle` — grabación de scheme (`addBuffToScheme()` en L319) | L285-330 | VERIFIED (sprint 2.7) |
| Handler `_bbsscheme_create` | L372-404 | VERIFIED (AUDIT-006) |
| Handler `_bbsscheme_apply` | L444-501 | VERIFIED (AUDIT-006) |
| Handler `_bbsbuff` — aplicación directa, sin escritura de schemes | L516-556 | VERIFIED (AUDIT-006 + sprint 2.7) |
| Entrega HTML `CommunityBoardHandler.separateAndSend(returnHtml, player)` | L633-640 (send en L639) | VERIFIED (sprint 2.7) |
| `parseSchemeSkills()` | L937-945 | VERIFIED (AUDIT-006) |
| `addBuffToScheme()` | L953-967 | VERIFIED (AUDIT-006, firma en L953) |
| `removeBuffFromScheme()` | L975-982 | VERIFIED (AUDIT-006) |
| `loadSchemes()` | L989-1009 | VERIFIED (AUDIT-006) |
| `loadSchemeSkills()` | L1017-1022 | VERIFIED (AUDIT-006) |
| Carga `CommunityMaxSchemes` desde `config/Custom/CommunityBoard.ini` (default 5) | L1090-1102 (getProperty en L1096) | VERIFIED (sprint 2.7) |

## 10. Cross-references

- SchemeBuffer analysis: `BUFFS/SCHEME_BUFFER_ANALYSIS.md`
- System comparison: `BUFFS/SCHEME_SYSTEM_COMPARISON.md`
- Skill semantics: `SKILLS/SKILL_SEMANTIC_REFERENCE.md`
- SchemeValidator design: `SKILLS/SCHEME_VALIDATOR_DESIGN.md`
- Validation lifecycle: `SKILLS/SCHEME_VALIDATION_LIFECYCLE.md`
