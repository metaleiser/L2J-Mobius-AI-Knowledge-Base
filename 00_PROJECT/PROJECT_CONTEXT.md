# PROJECT_CONTEXT

**Última actualización**: 2026-08-25 (KB v2.0)
**Rol**: contexto de proyecto y mapa de las 4 entidades del workspace.

---

## Qué es este proyecto

Servidor **L2J Mobius CT 2.6 HighFive** (crónica *Lineage 2 High Five*).
- **Type**: MMORPG Server Emulator (Lineage 2)
- **Language**: Java (core) + scripts (Java datapack) + XML/SQL/INI (data/config)
- **Build**: Apache Ant
- **Instalación / juego**: servidor runtime desplegado en este workspace

---

## Las 4 entidades del workspace (rutas reales)

El workspace raíz (`e:\L2J MOBIUS`) contiene **cuatro entidades distintas y NO intercambiables**:

| Entidad | Ruta (relativa a la raíz del workspace) | Qué es | Git |
|---|---|---|---|
| **SERVER_SOURCE** | `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive/` | Código fuente oficial de Mobius H5 (java/, dist/, build.xml, datapack de origen) | Sí (GitLab) |
| **SERVER_RUNTIME** | `L2J_Mobius_CT_2.6_HighFive/` | Servidor desplegado que realmente se usa para jugar/probar (JARs, config, data, geodata) | No |
| **CLIENT** | `Lineage2-TCT-273-client/` | Cliente H5 real (`system/l2.exe`, ver 413) | No |
| **KNOWLEDGE_BASE** | `AI_KNOWLEDGE_BASE/` | Esta documentación / memoria del proyecto | Sí (GitHub) |

> El propio workspace NO es un repositorio único: hay **dos** repositorios Git (KB y UPSTREAM). `SERVER_RUNTIME` y `CLIENT` no tienen Git.

---

## Reglas de autoridad (jerarquía)

1. **SERVER_SOURCE (UPSTREAM)** → autoridad para **implementación y arquitectura del código**.
2. **SERVER_RUNTIME** → autoridad para el **estado actualmente desplegado y observable** del servidor que usamos.
3. **CLIENT** → autoridad para información que **existe exclusivamente en el cliente**.
4. **KB** → memoria estructurada del conocimiento investigado. **Nunca sustituye la evidencia original.**
5. **Wiki / documentación** → referencia secundaria.

**Si SOURCE y RUNTIME difieren**: NO corregir automáticamente ninguno. Registrar la diferencia y su contexto/versionado.

**Si CLIENT y SERVER difieren**: registrar la discrepancia y determinar qué lado produce/consume el comportamiento.

**Nunca mezclar SOURCE, RUNTIME y CLIENT sin indicar la procedencia.**

---

## SOURCE baseline ≠ RUNTIME build

Es **incorrecto** afirmar que el runtime es idéntico al upstream o que representan el mismo commit:

| | SERVER_SOURCE | SERVER_RUNTIME |
|---|---|---|
| Referencia | Commit `e2518ab10872b28cd4c6860e102b493656ba8728` (2026) | Build/runtime **26/05/2024** |
| Procedencia | Repo GitLab oficial `MobiusDevelopment/L2J_Mobius`, rama `master` | Pack H5 compilado descomprimido (resultado de un build) |
| Evidencia del build | `build.xml` (Ant) — genera JARs + ZIP | JARs fechados 26/05/2024, config/data/geodata propios |
| Estado | Source limpio, con historial Git | Sin Git; estado desplegado observable |

El runtime deriva del **mismo ecosistema H5** pero de un **build anterior/diferente** al baseline del source. Sus configs, tablas y datapack difieren (ver `REFERENCE_SOURCES.md` y `VERSIONING/UPSTREAM_BASELINE.md`).

---

## Dónde investigar cada tipo de pregunta

| Si la pregunta es sobre… | Buscar primero en |
|---|---|
| Implementación / arquitectura de código | SOURCE (`UPSTREAM/.../java`, `dist`, `build.xml`) |
| Estado/configuración actualmente ejecutada | RUNTIME (`L2J_Mobius_CT_2.6_HighFive/game/config`, `data`) |
| Texto/recurso/comportamiento específico del cliente | CLIENT (`Lineage2-TCT-273-client/system`, `L2text`, etc.) |
| Conocimiento previamente investigado | KNOWLEDGE_BASE (`AI_KNOWLEDGE_BASE`) |
| Documentación general | REFERENCE SOURCES (`00_PROJECT/REFERENCE_SOURCES.md`) |

---

## SERVER_RUNTIME como laboratorio

`SERVER_RUNTIME` es el **entorno actual de juego/pruebas/observación**. Conceptualmente:

```
SERVER_RUNTIME
  → jugar / probar
  → observar
  → investigar
  → verificar
  → documentar en KB
```

Los **logs son evidencia temporal**, NO se almacenan automáticamente en la KB. No se ha creado aún un `SERVER_TEST/` separado.

---

## Enlaces

- [REFERENCE_SOURCES.md](REFERENCE_SOURCES.md) — fuentes y su autoridad
- [DECISIONS.md](DECISIONS.md) — decisiones registradas
- [ROADMAP.md](ROADMAP.md) — hoja de ruta
- [IDEAS.md](IDEAS.md) — ideas futuras
- [../VERSIONING/UPSTREAM_BASELINE.md](../VERSIONING/UPSTREAM_BASELINE.md) — baseline source vs runtime build
- [../INDEXES/MASTER_INDEX.md](../INDEXES/MASTER_INDEX.md) — punto de entrada
