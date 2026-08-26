# KB_VERSION

```
KB_VERSION          = 2.2
STATUS              = SYNCHRONIZED
SOURCE_BASELINE     = e2518ab10872b28cd4c6860e102b493656ba8728 (upstream master)
RUNTIME_BUILD       = 26/05/2024 (SERVER_RUNTIME desplegado, sin Git)
UPSTREAM_REPOSITORY = MobiusDevelopment/L2J_Mobius
BASELINE_DATE       = 2026-08-22 03:06:03 +0300
LAST_AUDIT          = 2026-08-26 (AUDIT-004: KB v2.2 consolidación quest-engine + gaps map + hardening)
LAST_COMMIT         = 63c9b40 (KB v0.9: PvE vertical slice + party credit + spawn query consolidation)
PREVIOUS_COMMIT     = d029681 (KB: complete Q00005 PvE vertical slice)
00_PROJECT_DOCS     = 5
TECHNICAL_DOCS      = 81
SYSTEM_DOCS         = 9 (AI_INSTRUCTIONS=5, VERSIONING=4)
TOTAL_MD            = 95
```

> **Nota sobre "KB v0.9"**: el commit `63c9b40` utilizó **"KB v0.9" como nomenclatura de trabajo del micro-sprint**. La versión canónica de la Knowledge Base es **v2.2**. No reinterpretar el mensaje de ese commit como versión de la KB.

## Cambios de v2.1 → v2.2

- **QUESTS/QUEST_ENGINE_REFERENCE.md** (nuevo): referencia central de APIs del engine (`Quest#*`, `QuestState#*`), 15 categorías.
- **GAPS.md** (nuevo): mapa central de cobertura COVERED/PARTIAL/MISSING/SOURCE_REQUIRED.
- **QUESTS/QUEST_PARTY_CREDIT.md**, **QUESTS/QUEST_VERTICAL_SLICE_TEMPLATE.md**, **WORLD/SPAWN_QUERY_GUIDE.md**, slices **Q00003/Q00005**: incorporados desde micro-sprints 2.2–2.3 (ya existían en disco; ahora contabilizados).
- **QUEST_REWARDS.md**: sección de consultas de inventario + patrones reutilizables (ENGINE PATTERN vs QUEST-SPECIFIC).
- **QUEST_ARCHITECTURE.md**: conteo quests corregido 543/511 → **532/510** (filesystem 2026-08-26).
- **Cross-links** AGGRO_SYSTEM/DEATH_FLOW ↔ QUEST_PARTY_CREDIT (quest credit ≠ drop normal).
- **VERIFICATION_RULES.md**: taxonomía ampliada (DIRECTLY_SUPPORTED, ORIENTATION_ONLY, SOURCE_REQUIRED, OUTDATED, CONTRADICTED, UNKNOWN_CLIENT) + metadata estándar.

## Regla canónica de conteo de documentos

```
TÉCNICOS  = todos los .md EXCEPT: 00_PROJECT, AI_INSTRUCTIONS, VERSIONING
SISTEMA    = AI_INSTRUCTIONS + VERSIONING
00_PROJECT = 00_PROJECT (categoría por separado)
TOTAL      = TÉCNICOS + 00_PROJECT + SISTEMA
```

Con esta regla, estado canónico **KB v2.2**: 81 (técnicos) + 5 (00_PROJECT) + 9 (sistema) = **95**. Los documentos `INDEXES/`, `README.md` (root) y `CLIENT_RESEARCH/` se cuentan como **técnicos** (no están excluidos). Conteos históricos conservados: **88** = 74 técnicos + 5 + 9 (KB v2.1, 2026-08-25) y **81** = 67 técnicos + 5 + 9 (KB v2.0; el `67` de AUDIT-001/AUDIT-002 se conserva como histórico).

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
