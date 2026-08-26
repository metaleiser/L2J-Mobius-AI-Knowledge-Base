# KB_VERSION

```
KB_VERSION          = 2.3
STATUS              = SYNCHRONIZED
SOURCE_BASELINE     = e2518ab10872b28cd4c6860e102b493656ba8728 (upstream master)
RUNTIME_BUILD       = 26/05/2024 (SERVER_RUNTIME desplegado, sin Git)
UPSTREAM_REPOSITORY = MobiusDevelopment/L2J_Mobius
BASELINE_DATE       = 2026-08-22 03:06:03 +0300
LAST_AUDIT          = 2026-08-26 (AUDIT-005: KB v2.3 Skill Semantic Foundation + ABNORMAL_TYPE_REFERENCE + GAPS/INDEX updates)
LAST_COMMIT         = 62a47c3 (KB v2.2: consolidate quest engine and coverage foundations)
PREVIOUS_COMMIT     = 63c9b40 (KB v0.9: PvE vertical slice + party credit + spawn query consolidation)
00_PROJECT_DOCS     = 5
TECHNICAL_DOCS      = 83
SYSTEM_DOCS         = 9 (AI_INSTRUCTIONS=5, VERSIONING=4)
TOTAL_MD            = 97
```

> **Nota sobre "KB v0.9"**: el commit `63c9b40` utilizó **"KB v0.9" como nomenclatura de trabajo del micro-sprint**. La versión canónica de la Knowledge Base es **v2.3**. No reinterpretar el mensaje de ese commit como versión de la KB.

## Cambios de v2.2 → v2.3

- **SKILLS/SKILL_SEMANTIC_REFERENCE.md** (nuevo): referencia central semántica para Scheme Validator (skill identity, categories, beneficial/harmful 4-capas, targets, stacking, restrictions, evidence matrix).
- **SKILLS/ABNORMAL_TYPE_REFERENCE.md** (nuevo): catálogo de 337 valores `AbnormalType` extraídos literalmente del enum SOURCE; reglas de stacking con `SOURCE_REQUIRED` para clasificación individual.
- **SKILLS/EFFECT_SYSTEM.md** (extendido): catálogos `EffectType` (42 valores) y `EffectFlag` (22 flags), buff category routing, stacking/replacement rules, slot management.
- **SKILLS/SKILL_TARGETING.md** (extendido): catálogos `AffectScope` (15 valores) y `AffectObject` (10 valores).
- **SKILLS/SKILL_DATA_MODEL.md** (extendido): categorías de skill (`passive/debuff/triggered/dance/song/toggle/buff`), semántica beneficial/harmful multicapa.
- **GAPS.md** (actualizado): SKILLS, BUFFS/EFFECTS, TARGETS, SCHEME_VALIDATION reflejan cobertura 2.5.
- **INDEXES/MASTER_INDEX.md** (actualizado): 12 docs SKILLS, KB v2.3, AUDIT-005.
- **00_PROJECT/ROADMAP.md** (actualizado): micro-sprint 2.5 completado, 2.6 siguiente.

## Regla canónica de conteo de documentos

```
TÉCNICOS  = todos los .md EXCEPT: 00_PROJECT, AI_INSTRUCTIONS, VERSIONING
SISTEMA    = AI_INSTRUCTIONS + VERSIONING
00_PROJECT = 00_PROJECT (categoría por separado)
TOTAL      = TÉCNICOS + 00_PROJECT + SISTEMA
```

Con esta regla, estado canónico **KB v2.3**: 83 (técnicos) + 5 (00_PROJECT) + 9 (sistema) = **97**. Los documentos `INDEXES/`, `README.md` (root) y `CLIENT_RESEARCH/` se cuentan como **técnicos** (no están excluidos). Conteos históricos conservados: **95** = 81 técnicos + 5 + 9 (KB v2.2, 2026-08-26); **88** = 74 técnicos + 5 + 9 (KB v2.1, 2026-08-25); **81** = 67 técnicos + 5 + 9 (KB v2.0; el `67` de AUDIT-001/AUDIT-002 se conserva como histórico).

## Qué significa KB 2.0

KB v2.0 **actualiza, no reconstruye** la KB v1.0. Se preserva todo el conocimiento histórico. Los cambios de v2.0:

- Nuevo módulo **`00_PROJECT/`** (PROJECT_CONTEXT, REFERENCE_SOURCES, DECISIONS, ROADMAP, IDEAS).
- **Separación de entidades**: SERVER_SOURCE vs SERVER_RUNTIME vs CLIENT vs KNOWLEDGE_BASE (con rutas reales).
- Documentado que **SOURCE baseline ≠ RUNTIME build** (source `e2518ab` vs runtime 26/05/2024).
- **Build documentado: Apache Ant** (flujo real checkRequirements→…→cleanup), sin Gradle.
- **Taxonomía de estados** ampliada: VERIFIED, OBSERVED, UNVERIFIED, ASSUMPTION, UNKNOWN, DEPRECATED, CONFLICT (con compatibilidad histórica).
- MASTER_INDEX actualizado como punto de entrada único.

**2.0 NO significa que "todo esté VERIFIED".** Los estados PARTIAL/UNKNOWN/RCV se conservan donde corresponda; ver `AI_INSTRUCTIONS/VERIFICATION_RULES.md`.

## Política de versionado

| Situación | Versión resultante |
|---|---|
| Auditoría sin cambios relevantes | Se mantiene la versión (se registra en AUDIT_HISTORY) |
| Correcciones documentales menores / erratas / conteos puntuales | 2.0.x (p. ej. 2.0.1) o registro interno en AUDIT_HISTORY |
| Cambio significativo de conocimiento (nueva área documentada, correcciones estructurales de fondo) | 2.1 |
| Cambio mayor de organización/metodología de la KB | 3.0 |

Regla fundamental: **no incrementar la versión solo porque se ejecutó una auditoría.** El versionado refleja contenido, no actividad.

Historial de cambios: [CHANGELOG.md](CHANGELOG.md) · Auditorías: [AUDIT_HISTORY.md](AUDIT_HISTORY.md) · Baselines: [UPSTREAM_BASELINE.md](UPSTREAM_BASELINE.md)
