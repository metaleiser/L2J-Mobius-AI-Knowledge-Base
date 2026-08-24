# UPSTREAM_BASELINE

## Registro vigente

| Campo | Valor |
|---|---|
| Repository | https://gitlab.com/MobiusDevelopment/L2J_Mobius.git |
| Branch | master |
| Commit | `e2518ab10872b28cd4c6860e102b493656ba8728` |
| Commit date | 2026-08-22 03:06:03 +0300 |
| Snapshot documentado | `L2J_Mobius_CT_2.6_HighFive` |
| Clone local de referencia | `UPSTREAM\L2J_Mobius` (sparse-checkout de esa carpeta) |

## Verificación snapshot ↔ commit

El snapshot local (ZIP sin metadatos Git) fue comparado **estructuralmente** contra este commit el 2026-08-24 mediante recuentos recursivos de filesystem sobre ambos árboles. Resultado: coincidencia exacta en todas las categorías medidas.

| Métrica | Snapshot | Upstream @ e2518ab |
|---|---|---|
| Total archivos | 27.657 | 27.657 |
| .java | 3.194 | 3.194 |
| .xml | 2.907 | 2.907 |
| .sql | 116 | 116 |
| .ini | 73 | 73 |
| .properties | 0 | 0 |

También coinciden: estructura superior (`java/dist/launcher/build.xml/.project/.classpath/.settings/readme.txt`) y las rutas críticas `dist/game/config`, `dist/game/data`, `java/org/l2jmobius`.

## Alcance exacto de la verificación

- ✅ Realizado: comparación estructural por inventario (recuento total + por extensión + estructura de directorios + rutas clave).
- ❌ NO realizado: comparación byte a byte / hash de los 27.657 archivos.
- Conclusión legítima: el snapshot corresponde a este commit con altísima confianza. Toda afirmación de contenido específico debe verificarse igualmente contra el código antes de usarse.

## Rotación

Ver [../AI_INSTRUCTIONS/CHANGE_DETECTION.md](../AI_INSTRUCTIONS/CHANGE_DETECTION.md): tras una actualización aprobada y documentada se establece `NEW_UPSTREAM_BASELINE_COMMIT`, conservando aquí el histórico:

| Desde | Hasta | Fecha rotación | Motivo |
|---|---|---|---|
| — | `e2518ab10872b28cd4c6860e102b493656ba8728` | 2026-08-24 | Baseline inicial KB 1.0 |