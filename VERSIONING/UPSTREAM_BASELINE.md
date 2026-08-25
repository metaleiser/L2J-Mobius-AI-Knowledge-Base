# UPSTREAM_BASELINE

## Registro vigente — SERVER_SOURCE (upstream)

| Campo | Valor |
|---|---|
| Repository | https://gitlab.com/MobiusDevelopment/L2J_Mobius.git |
| Branch | master |
| Commit | `e2518ab10872b28cd4c6860e102b493656ba8728` |
| Commit date | 2026-08-22 03:06:03 +0300 |
| Clone local | `UPSTREAM/L2J_Mobius` (partial clone `blob:none`, sparse-checkout `L2J_Mobius_CT_2.6_HighFive/`) |
| Snapshot documentado | `L2J_Mobius_CT_2.6_HighFive` (source tree completo en el clone) |
| Build | Apache Ant (`build.xml`) |

> **IMPORTANTE**: este baseline describe el **SERVER_SOURCE** (código upstream). NO describe el `SERVER_RUNTIME`.

---

## REGISTRO — SERVER_RUNTIME (no es upstream)

El `SERVER_RUNTIME` (`L2J_Mobius_CT_2.6_HighFive/` en la raíz del workspace) es un **despliegue** del servidor H5, NO el source:

| Campo | Valor |
|---|---|
| Nº de build/runtime | **26/05/2024** (JARs `GameServer.jar`/`LoginServer.jar` con esa fecha de compilación) |
| Git | **Sin `.git`** |
| Procedencia | Resultado descomprimido de un build/distribución (JARs + datapack + geodata + config) |
| Relación con baseline | Deriva del mismo ecosistema H5, pero **NO coincide** con `e2518ab` (configs, tablas y datapack difieren) |

**NO afirmar** que el runtime es idéntico al upstream ni que ambos representan el mismo commit.
La relación correcta es **SOURCE BASELINE vs RUNTIME BUILD**.

---

## Verificación snapshot source ↔ commit (histórico KB v1.0)

Comparación estructural realizada el 2026-08-24 sobre el source:

| Métrica | Snapshot | Upstream @ e2518ab |
|---|---|---|
| Total archivos | 27.657 | 27.657 |
| .java | 3.194 | 3.194 |
| .xml | 2.907 | 2.907 |
| .sql | 116 | 116 |
| .ini | 73 | 73 |
| .properties | 0 | 0 |

Alcance: comparación estructural por inventario; **no** byte a byte. Conclusión: el source corresponde al commit con altísima confianza; toda afirmación de contenido específico se verifica contra código antes de usarse.

> Nota de auditoría 2026-08-25: el **runtime** reporta 25.075 archivos, `.java`=1.320 (solo scripts de datapack en `game/data/scripts`), `.xml`=2.390, `.sql`=118, `.ini`=58 (18 en `game/config`) y **sí** incluye geodata — valores que difieren del source y confirman que runtime ≠ source.

---

## Rotación

Ver [../AI_INSTRUCTIONS/CHANGE_DETECTION.md](../AI_INSTRUCTIONS/CHANGE_DETECTION.md). Tras una actualización aprobada y documentada se establece `NEW_UPSTREAM_BASELINE_COMMIT`, conservando el histórico:

| Desde | Hasta | Fecha rotación | Motivo |
|---|---|---|---|
| — | `e2518ab10872b28cd4c6860e102b493656ba8728` | 2026-08-24 | Baseline inicial KB 1.0 (source) |
