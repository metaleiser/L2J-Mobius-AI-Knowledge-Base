# KB_VERSION

```
KB_VERSION          = 2.1
STATUS              = SYNCHRONIZED
SOURCE_BASELINE     = e2518ab10872b28cd4c6860e102b493656ba8728 (upstream master)
RUNTIME_BUILD       = 26/05/2024 (SERVER_RUNTIME desplegado, sin Git)
UPSTREAM_REPOSITORY = MobiusDevelopment/L2J_Mobius
BASELINE_DATE       = 2026-08-22 03:06:03 +0300
LAST_AUDIT          = 2026-08-25 (AUDIT-003: KB v2.1 metadata reconciliation + Q00005)
00_PROJECT_DOCS     = 5
TECHNICAL_DOCS      = 74
SYSTEM_DOCS         = 9 (AI_INSTRUCTIONS=5, VERSIONING=4)
TOTAL_MD            = 88
```

## Regla canónica de conteo de documentos

```
TÉCNICOS  = todos los .md EXCEPT: 00_PROJECT, AI_INSTRUCTIONS, VERSIONING
SISTEMA    = AI_INSTRUCTIONS + VERSIONING
00_PROJECT = 00_PROJECT (categoría por separado)
TOTAL      = TÉCNICOS + 00_PROJECT + SISTEMA
```

Con esta regla: 74 (técnicos) + 5 (00_PROJECT) + 9 (sistema) = **88**. Los documentos `INDEXES/`, `README.md` (root) y `CLIENT_RESEARCH/` se cuentan como **técnicos** (no están excluidos). El número `67` registrado en AUDIT-001/AUDIT-002 era correcto para el **snapshot v2.0** (81 docs) y se conserva como histórico.

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
