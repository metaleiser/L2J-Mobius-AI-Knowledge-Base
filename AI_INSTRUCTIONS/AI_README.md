# AI_README — Guía de entrada para cualquier IA

**Última revisión**: 2026-08-24 · **Audiencia**: agentes de IA que consulten o modifiquen esta KB

---

## 1. Qué es esta KB

`AI_KNOWLEDGE_BASE/` es una base de conocimiento técnica, versionada y auditable, que documenta el servidor **L2J Mobius CT 2.6 High Five** para consumo por IAs, evitando re-analizar el código completo en cada consulta.

No sustituye al código. Lo documenta.

## 2. Mapa del workspace (KB v2.0)

```
e:\L2J MOBIUS\                                            ← raíz del workspace
├── AI_KNOWLEDGE_BASE\                                    ← KNOWLEDGE_BASE (ESTA KB; única zona modificable por defecto)
│   ├── 00_PROJECT\                                       ← contexto, fuentes, decisiones, roadmap, ideas
│   ├── AI_INSTRUCTIONS\                                  ← sistema de instrucciones para IAs
│   ├── VERSIONING\                                       ← versión, baselines, changelog, auditorías
│   └── (67 documentos técnicos temáticos)
├── L2J_Mobius_CT_2.6_HighFive\                           ← SERVER_RUNTIME (desplegado/ejecutable, sin Git)
│   └── game\, login\, db_installer\, libs\, backup\, ... (JARs 26/05/2024, config, data, geodata)
├── UPSTREAM\L2J_Mobius\                                  ← SERVER_SOURCE (repo Git oficial)
│   └── L2J_Mobius_CT_2.6_HighFive\                       ← source tree completo (java/, dist/, build.xml)
└── Lineage2-TCT-273-client\                              ← CLIENT H5 (system/l2.exe)
```

## 3. Las 4 entidades — distinción OBLIGATORIA

Cada pregunta/afirmación debe indicar de qué entidad proviene. **Nunca mezclar SOURCE, RUNTIME y CLIENT sin señalar la procedencia.**

| Entidad | Ruta | Autoridad para… |
|---|---|---|
| **SERVER_SOURCE** | `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive` | implementación y arquitectura de código |
| **SERVER_RUNTIME** | `L2J_Mobius_CT_2.6_HighFive` | estado actualmente desplegado/observable que usamos |
| **CLIENT** | `Lineage2-TCT-273-client` | información que existe solo en el cliente |
| **KNOWLEDGE_BASE** | `AI_KNOWLEDGE_BASE` | conocimiento previamente investigado (nunca sustituye la evidencia) |

**Cómo elegir la fuente de una pregunta:**

```
implementación                          → SOURCE
estado/configuración actualmente ejecut.→ RUNTIME
texto/recurso/comportamiento de cliente → CLIENT
conocimiento previamente investigado    → KB
documentación general                   → REFERENCE SOURCES (00_PROJECT/REFERENCE_SOURCES.md)
```

Si SOURCE y RUNTIME difieren → **registrar la discrepancia**, no corregir automáticamente ninguno. Ver [../00_PROJECT/PROJECT_CONTEXT.md](../00_PROJECT/PROJECT_CONTEXT.md).

## 4. Baseline vigente

- **UPSTREAM_BASELINE_COMMIT**: `e2518ab10872b28cd4c6860e102b493656ba8728`
- **Fecha del commit**: 2026-08-22 03:06:03 +0300 · rama `master` · repo `MobiusDevelopment/L2J_Mobius`
- El snapshot local fue comparado **estructuralmente** contra ese commit (coincidencia exacta de inventarios). Detalles: [../VERSIONING/UPSTREAM_BASELINE.md](../VERSIONING/UPSTREAM_BASELINE.md)

Antes de modificar cualquier documento: determina **qué versión del código estás observando** y **contra qué commit fue validada la KB**.

## 5. Baseline vigente (SOURCE)

- **SOURCE_BASELINE_COMMIT**: `e2518ab10872b28cd4c6860e102b493656ba8728`
- **Fecha del commit**: 2026-08-22 03:06:03 +0300 · rama `master` · repo `MobiusDevelopment/L2J_Mobius` (GitLab oficial)
- El `SERVER_SOURCE` local (`UPSTREAM/...`) corresponde a este commit.
- El `SERVER_RUNTIME` es un **build distinto (26/05/2024)** y NO coincide con este baseline (SOURCE baseline ≠ RUNTIME build). Detalles: [../VERSIONING/UPSTREAM_BASELINE.md](../VERSIONING/UPSTREAM_BASELINE.md)

Antes de modificar cualquier documento: determina **qué entidad estás observando** (SOURCE/RUNTIME/CLIENT), **qué versión del código** y **contra qué baseline fue validada la KB**.

## 6. Taxonomía de estados (KB v2.0)

| Estado | Significado | Compatibilidad |
|---|---|---|
| **VERIFIED** | Confirmado mediante código, cliente, ejecución reproducible o evidencia suficiente (idealmente `archivo:línea`) | equivale a VERIFIED |
| **OBSERVED** | Observado durante una prueba/uso del servidor o cliente, aún no completamente explicado | nuevo |
| **UNVERIFIED** | Afirmación encontrada en una fuente que aún no fue comprobada | releva a REQUIRES CODE VERIFICATION / afines |
| **ASSUMPTION** | Hipótesis de trabajo | nuevo |
| **UNKNOWN** | No existe información suficiente | equivale a UNKNOWN |
| **DEPRECATED** | Información anteriormente válida pero sustituida | releva a OUTDATED |
| **CONFLICT** | Existen fuentes o evidencias contradictorias | releva a CONTRADICTED |
| *(PARTIAL)* | Parte demostrada, parte no (histórico, se conserva) | histórico |
| *(PLANNED ⧗)* | Documentado como planificado; el doc aún no existe | histórico |

- **No renombrar destructivamente** documentos históricos que usen estados antiguos; documentar el mapeo y usarlo a partir de v2.0.
- Los conteos que puedan cambiar llevan fecha/commit de referencia.

## 7. Qué puede modificar una IA

- Únicamente archivos dentro de `AI_KNOWLEDGE_BASE/`.
- Solo tras verificar contra código y, para correcciones no triviales, tras presentar propuesta y obtener autorización.

## 8. Qué NO debe modificar (nunca)

- **SERVER_SOURCE / SERVER_RUNTIME** (Java/XML/SQL/INI/config/build/scripts/datapack)
- **CLIENT** (`Lineage2-TCT-273-client`)
- Clone `UPSTREAM/L2J_Mobius` y su `.git`
- Cualquier archivo fuera de `AI_KNOWLEDGE_BASE/` sin autorización explícita

## 9. Antes de corregir documentación

1. Leer [../INDEXES/MASTER_INDEX.md](../INDEXES/MASTER_INDEX.md) y [../VERSIONING/KB_VERSION.md](../VERSIONING/KB_VERSION.md).
2. Comprobar estado Git del clone upstream y si hay commits nuevos ([CHANGE_DETECTION.md](CHANGE_DETECTION.md)).
3. Verificar cada afirmación contra el código actual.
4. Clasificar hallazgo (CRITICAL/HIGH/MEDIUM/LOW/INFO) con evidencia.
5. Presentar propuesta → esperar autorización → aplicar → validar (ver [AUDIT_PROTOCOL.md](AUDIT_PROTOCOL.md)).

## 10. Preguntas que esta KB debe poder responder en todo momento

- ¿Qué versión de Mobius documenta? → baseline en [../VERSIONING/UPSTREAM_BASELINE.md](../VERSIONING/UPSTREAM_BASELINE.md)
- ¿Último commit auditado? → [../VERSIONING/AUDIT_HISTORY.md](../VERSIONING/AUDIT_HISTORY.md)
- ¿Qué está VERIFIED / PARTIAL / UNKNOWN? → mismo archivo, sección Result
- ¿Qué cambió desde la última auditoría? → [../VERSIONING/CHANGELOG.md](../VERSIONING/CHANGELOG.md)
- ¿Qué documentos podrían necesitar actualización? → salida de la última auditoría (Impact Analysis)

Regla mnemotécnica: **primero versión del código observado, después documento**.