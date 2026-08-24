# AI_README — Guía de entrada para cualquier IA

**Última revisión**: 2026-08-24 · **Audiencia**: agentes de IA que consulten o modifiquen esta KB

---

## 1. Qué es esta KB

`AI_KNOWLEDGE_BASE/` es una base de conocimiento técnica, versionada y auditable, que documenta el servidor **L2J Mobius CT 2.6 High Five** para consumo por IAs, evitando re-analizar el código completo en cada consulta.

No sustituye al código. Lo documenta.

## 2. Mapa del workspace

```
C:\AI_KNOWLEDGE_BASE\                                   ← raíz del workspace
├── AI_KNOWLEDGE_BASE\                                  ← ESTA KB (única zona modificable por defecto)
│   ├── *.md y carpetas temáticas                        (67 documentos técnicos)
│   ├── AI_INSTRUCTIONS\                                ← este sistema de instrucciones
│   └── VERSIONING\                                     ← versión, baseline, changelog, auditorías
├── L2 H5\                                              ← CLIENTE del juego (solo registro; sin auditoría aún)
├── L2J_Mobius-master-L2J_Mobius_CT_2.6_HighFive\
│   └── L2J_Mobius-master-L2J_Mobius_CT_2.6_HighFive\
│       └── L2J_Mobius_CT_2.6_HighFive\                 ← SERVIDOR: código real (snapshot ZIP)
│           ├── java\org\l2jmobius\{commons,gameserver,loginserver,...}
│           └── dist\game\{config,data}, dist\db_installer, launcher, build.xml
└── UPSTREAM\L2J_Mobius\                                ← clone Git de referencia (rama master,
    └── L2J_Mobius_CT_2.6_HighFive\                        sparse-checkout de esta carpeta)
```

## 3. Jerarquía de verdad (OBLIGATORIA)

```
CÓDIGO > CONFIGURACIÓN REAL > ESTRUCTURA REAL (SQL/XML/filesystem) > KB > INFERENCIA
```

Si la KB contradice al código: **gana el código**. Una inferencia **nunca** se convierte en hecho VERIFIED.

## 4. Baseline vigente

- **UPSTREAM_BASELINE_COMMIT**: `e2518ab10872b28cd4c6860e102b493656ba8728`
- **Fecha del commit**: 2026-08-22 03:06:03 +0300 · rama `master` · repo `MobiusDevelopment/L2J_Mobius`
- El snapshot local fue comparado **estructuralmente** contra ese commit (coincidencia exacta de inventarios). Detalles: [../VERSIONING/UPSTREAM_BASELINE.md](../VERSIONING/UPSTREAM_BASELINE.md)

Antes de modificar cualquier documento: determina **qué versión del código estás observando** y **contra qué commit fue validada la KB**.

## 5. Diferencia servidor / cliente / upstream

| Fuente | Qué es | Uso permitido |
|---|---|---|
| **SERVIDOR** (snapshot local) | Código que realmente ejecuta el servidor | Fuente primaria para comportamiento actual |
| **UPSTREAM** (clone) | Historial Git oficial de Mobius | Detectar evolución, diffs, commits nuevos |
| **CLIENTE** (`L2 H5`) | Cliente del juego | Solo registro; auditoría CLIENT↔SERVER es fase futura |

Las afirmaciones sobre el cliente deben marcarse como **CLIENT** y nunca mezclarse con las del servidor.

## 6. Estados documentales

| Estado | Significado |
|---|---|
| **VERIFIED** | Existe evidencia concreta en código/filesystem (idealmente `archivo:línea`) que lo demuestra |
| **PARTIAL** | Parte demostrada, parte no; se indica cuál |
| **UNKNOWN** | No se pudo determinar con la evidencia disponible |
| **REQUIRES CODE VERIFICATION** | Requiere inspección adicional antes de usarse como cierto |
| **OUTDATED** | El código avanzó; la afirmación correspondía a un estado anterior |
| **CONTRADICTED** | El código demuestra explícitamente que la afirmación es incorrecta |
| **PLANNED (⧗)** | Documentado como planificado; el documento NO existe todavía |

Los conteos que puedan cambiar llevan fecha/commit de referencia.

## 7. Qué puede modificar una IA

- Únicamente archivos dentro de `AI_KNOWLEDGE_BASE/`.
- Solo tras verificar contra código y, para correcciones no triviales, tras presentar propuesta y obtener autorización.

## 8. Qué NO debe modificar (nunca)

- Servidor local (Java/XML/SQL/INI/config/build/scripts/datapack)
- Cliente `L2 H5`
- Clone `UPSTREAM\L2J_Mobius` y sus `.git`
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