# KB_VERSION

```
KB_VERSION          = 1.0
STATUS              = PARTIALLY SYNCHRONIZED
UPSTREAM_COMMIT     = e2518ab10872b28cd4c6860e102b493656ba8728
UPSTREAM_BRANCH     = master
UPSTREAM_REPOSITORY = MobiusDevelopment/L2J_Mobius
BASELINE_DATE       = 2026-08-22 03:06:03 +0300
LAST_AUDIT          = 2026-08-24 (AUDIT-001)
TECHNICAL_DOCS      = 67
SYSTEM_DOCS         = 9 (AI_INSTRUCTIONS=5, VERSIONING=4)
```

## Qué significa KB 1.0

Snapshot documental consolidado y parcialmente sincronizado: las áreas priorizadas (CONFIGURATION, DATABASE core, MANAGERS catálogo/firmas, PACKETS estructura/conteos, BUILD/layout, arranque GameServer) fueron verificadas contra el código del commit baseline y corregidas; el resto de los 67 documentos técnicos conserva estados PARTIAL/UNKNOWN/RCV declarados y NO está declarado como VERIFIED global.

**1.0 NO significa "todo VERIFIED".**

## Política de versionado

| Situación | Versión resultante |
|---|---|
| Auditoría sin cambios relevantes | Se mantiene 1.0 (se registra en AUDIT_HISTORY) |
| Correcciones documentales menores / erratas / conteos puntuales | 1.0.x (p. ej. 1.0.1) o registro interno en AUDIT_HISTORY |
| Cambio significativo de conocimiento (nueva área documentada, correcciones estructurales de fondo) | 1.1 |
| Cambio mayor de organización/metodología de la KB | 2.0 |

Regla fundamental: **no incrementar la versión solo porque se ejecutó una auditoría.** El versionado refleja contenido, no actividad.

Historial de cambios: [CHANGELOG.md](CHANGELOG.md) · Auditorías: [AUDIT_HISTORY.md](AUDIT_HISTORY.md) · Baseline técnica: [UPSTREAM_BASELINE.md](UPSTREAM_BASELINE.md)