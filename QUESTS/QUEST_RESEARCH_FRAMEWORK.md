# QUEST RESEARCH FRAMEWORK

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Metodología — investigación de quests  
**Casos de estudio modelo**: Q00001_LettersOfLove (simple), Q00039_RedEyedInvaders (compleja)  
**Baseline de referencia**: SERVER_SOURCE `e2518ab10872b28cd4c6860e102b493656ba8728` · SERVER_RUNTIME build **26/05/2024**  
**Verified**: 2026-08-25 (KB v2.0)  
**Status**: VERIFIED — metodología modelada contra Q00001 + Q00039

---

## INTRODUCCIÓN

Este documento define la **metodología estándar** para investigar cualquier quest de `L2J_Mobius_CT_2.6_HighFive`. Se basa en los aprendizajes de dos casos de estudio ya investigados:

- **Q00001_LettersOfLove** — quest simple: diálogo lineal, entrega de items entre NPCs, sin mobs.
- **Q00039_RedEyedInvaders** — quest compleja: múltiples mobs, drops en party con weighting, branching de drop, colecciones cruzadas, triggers por límite.

El framework **NO** asume que todas las quests tienen todos los componentes. Cada sección de la plantilla A-W puede ser **PRESENT**, **ABSENT**, **NOT_APPLICABLE**, **UNKNOWN** o **UNVERIFIED**.

### Principio rector

> **Generalizar la metodología, no los resultados concretos.**  
> Q00001 y Q00039 presentan CONFLICTs distintos. El hecho de que ambos tengan conflictos no significa que toda quest tenga conflictos — solo que **siempre se compara SOURCE vs RUNTIME**.

---

## CASOS DE ESTUDIO MODELO

### Q00001_LettersOfLove — Quest simple (delivery)

| Característica | Valor | Estado |
|---|---|---|
| Quest ID | 1 | VERIFIED · SOURCE/RUNTIME |
| Tipo | Diálogo lineal, entrega de items | VERIFIED · SOURCE/RUNTIME |
| NPCs | Darin (30048), Roxxy (30006), Baulro (30033) — 3 interactivos | VERIFIED · SOURCE/RUNTIME |
| Mobs | — | ABSENT |
| Items quest | 687, 688, 1079, 1080 | VERIFIED · SOURCE/RUNTIME |
| Condiciones | cond 1–4 (State: CREATED→STARTED→COMPLETED) | VERIFIED · SOURCE/RUNTIME |
| HTML files | 15 (14 `.html` + 1 `.htm`) | VERIFIED · RUNTIME/SOURCE (mismo conteo) |
| Recompensa | Item 906, EXP 5672, SP 446, Adena 2466 | VERIFIED · SOURCE/RUNTIME |
| SOURCE↔RUNTIME | NewGuide integración en SOURCE, no en RUNTIME | CONFLICT |
| CLIENT | Textos en `questname-e.dat`, `NpcName-e.dat`, `itemname-e.dat` | UNKNOWN (cifrado) |

**Lección clave**: Las diferencias SOURCE↔RUNTIME pueden ser en **lógica específica de la quest** (NewbieGuide en Q00001), no solo en packages o licencias.

### Q00039_RedEyedInvaders — Quest compleja (kill + collection)

| Característica | Valor | Estado |
|---|---|---|
| Quest ID | 39 | VERIFIED · SOURCE/RUNTIME |
| Tipo | Kill + colección doble con cross-check | VERIFIED · SOURCE/RUNTIME |
| NPCs | Babenco (30334), Bathis (30332) — 2 interactivos | VERIFIED · SOURCE/RUNTIME |
| NPCs narrativos | Magister Rohmer (no interactivo) | VERIFIED · HTML/LORE |
| Mobs | 20919, 20920, 20921, 20925 — 4 tipos | VERIFIED · SOURCE/RUNTIME |
| Items quest | 7178, 7179, 7180, 7181 (pares A/B, límites 100/30) | VERIFIED · SOURCE/RUNTIME |
| Condiciones | cond 1–5 | VERIFIED · SOURCE/RUNTIME |
| HTML files | 14 (10 `.html` + 4 `.htm`) | VERIFIED · RUNTIME/SOURCE |
| Recompensa | Items 6521×60, 6529×1, 6535×500, EXP 62366, SP 2783 | VERIFIED · SOURCE/RUNTIME |
| SOURCE↔RUNTIME | Packages, licencia, @author | CONFLICT (lógica idéntica) |
| CLIENT | Textos en `*.dat` cifrados | UNKNOWN (cifrado) |

**Lección clave**: Las diferencias SOURCE↔RUNTIME pueden ser en **aspectos estéticos/formato** (packages, licencia) sin afectar la lógica. Siempre documentar la naturaleza del conflicto.

---

## PATRONES COMUNES (GENERALIZABLES)

Los siguientes patrones aparecen en ambas quests y son el núcleo del framework:

| # | Patrón | Descripción | Q00001 | Q00039 |
|---|--------|-------------|--------|--------|
| C1 | Identificación | `super(questId)` en constructor; clase nombrada `Q000XX_Name` | ✅ | ✅ |
| C2 | Localización script | `quests/Q000XX_Name/Q000XX_Name.java` en `dist/` (SOURCE) y `game/` (RUNTIME) | ✅ | ✅ |
| C3 | Registro NPCs | `addStartNpc(…)` + `addTalkId(…)` | ✅ | ✅ |
| C4 | Registro kills | `addKillId(…)` | — | ✅ |
| C5 | Registro items | `registerQuestItems(…)` | ✅ | ✅ |
| C6 | State machine base | `State.CREATED / STARTED / COMPLETED` vía `startQuest()` / `exitQuest(false, true)` | ✅ | ✅ |
| C7 | Condiciones numéricas | `qs.setCond(n)` / `qs.getCond()` como fases | ✅ | ✅ |
| C8 | Callbacks | `onEvent(String, Npc, Player)`, `onTalk(Npc, Player)`; `onKill` si hay mobs | ✅ | ✅ |
| C9 | HTML naming | `npcId-page.html` / `.htm`; bypass: `bypass -h Quest QuestName npcId-page.html` | ✅ | ✅ |
| C10 | HTML en quest dir | Archivos HTML/HTM en el mismo directorio que el `.java` | ✅ | ✅ |
| C11 | MIN_LEVEL | Constante + chequeo `player.getLevel() >= MIN_LEVEL` | ✅ | ✅ |
| C12 | Recompensas estándar | `giveItems`, `giveAdena`, `addExpAndSp`, `exitQuest(false, true)` | ✅ | ✅ |
| C13 | NPCs vs lore refs | Distinguir interactivos (`addTalkId`) de referencias narrativas (HTML) | ✅ | ✅ |
| C14 | Estado de conocimiento | Cada afirmación: Estado + Procedencia + Evidencia | ✅ | ✅ |
| C15 | Comparación SOURCE↔RUNTIME | Siempre comparar; registrar CONFLICT con detalle | ✅ | ✅ |
| C16 | Cliente | Buscar `questname-e.dat`, `NpcName-e.dat`, `itemname-e.dat`; UNKNOWN si cifrado | ✅ | ✅ |
| C17 | Trazabilidad | Archivo + clase + método + líneas + procedencia | ✅ | ✅ |

---

## PATRONES ESPECÍFICOS (NO GENERALIZABLES)

| Patrón | Quest | ¿Generalizable? | Razón |
|--------|-------|-----------------|-------|
| Integración NewbieGuide | Q00001 (solo SOURCE) | ❌ | Mecánica específica de esta quest; no es patrón de quest estándar |
| Party drop ponderado (playerChance) | Q00039 | ⚠️ Solo kill quests en party | No todas las quests tienen mobs ni party |
| Branching de drop (`getRandomBoolean`) | Q00039 | ⚠️ Solo mobs multi-ruta | Específico; no todas usan branching |
| Collection-set cross-check | Q00039 | ⚠️ Solo quests de doble colección | ABSENT en Q00001 |
| Limit-aware `giveItemRandomly` return | Q00039 | ⚠️ Solo quests con colecciones | No applicable a delivery quests |
| Count-aware `hasItem` con ItemHolder | Q00039 | ⚠️ Solo cuando se usan ItemHolders | Q00001 usa `int` + `hasQuestItems` |
| Flujo lineal de entrega | Q00001 | ⚠️ Solo delivery quests | Específico |
| Número de estados numéricos | Variable por quest | ❌ | Q00001: 4 conds; Q00039: 5 conds |


---

## METODOLOGÍA DE INVESTIGACIÓN (TOKEN-EFFICIENT)

```
Pregunta → MASTER_INDEX → doc relevante → script quest → deps core → RUNTIME → CLIENT → conclusión → documentación
```

### Fases

| Fase | Acción | Token strategy |
|------|--------|---------------|
| 1. Identificar | MASTER_INDEX → sección QUESTS → doc existente si hay | No releer docs ya documentados |
| 2. Localizar | Encontrar script `.java` en RUNTIME (`game/data/scripts/quests/Q000XX_*/`) y SOURCE (`UPSTREAM/.../dist/`) | Leer Java completo solo si el script es corto (<200 líneas) o contiene lógica crítica. Si es largo, localizar callbacks relevantes y leer bloques específicos. |
| 3. Descomponer | Extraer constantes: NPCs, mobs, items, MIN_LEVEL, ItemHolders | Leer solo el header del script + constructor |
| 4. Mecánicas core | Identificar callbacks usados (`onEvent`, `onTalk`, `onKill`). Consultar API verificada en docs core existentes | Reutilizar documentación — no releer `Quest.java` a menos que haya duda en la API |
| 5. Diálogos | Leer todos los HTML/HTM del directorio | Extraer PLAYER CLUES (P) y LORE (Q) |
| 6. SOURCE↔RUNTIME | Comparar imports, licencia, packages, lógica | Registrar CONFLICT con tabla (K) |
| 7. CLIENT | Buscar en `Lineage2-TCT-273-client/` | UNKNOWN si `*.dat` cifrado |
| 8. Trazabilidad | Registrar archivo + línea + procedencia para cada hallazgo | Usar formato estándar (O) |
| 9. Síntesis | Tabla de conclusión + estado | Ver sección final del template (W) |

### Token efficiency: reglas

1. **No leer toda la KB indiscriminadamente.** Usar `MASTER_INDEX.md` como router.
2. **Reutilizar docs core existentes.** Si `QUEST_REWARDS.md` documenta `giveItemRandomly`, no releer `Quest.java`.
3. **Leer Java selectivamente.** Solo callbacks relevantes si script > 200 líneas.
4. **Cachear patrones.** Una vez identificado un patrón, reutilizar documentación existente.
5. **Ir al código.** Nunca sustituir evidencia por suposición.

---

## ENTIDADES Y PROCEDENCIA

| Entidad | Ruta (relativa a workspace root) | Rol | Estado de referencia actual |
|---------|----------------------------------|-----|---------------------------|
| **SERVER_SOURCE** | `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive/` | Autoridad para implementación/arquitectura | Commit `e2518ab10872b28cd4c6860e102b493656ba8728` |
| **SERVER_RUNTIME** | `L2J_Mobius_CT_2.6_HighFive/` | Estado actualmente desplegado | Build **26/05/2024** |
| **CLIENT** | `Lineage2-TCT-273-client/` | Información exclusiva del cliente | TCT-273 (cifrado pendiente) |
| **KB** | `AI_KNOWLEDGE_BASE/` | Memoria estructurada del conocimiento | v2.0 (2026-08-25) |

> **IMPORTANTE**: SOURCE baseline ≠ RUNTIME build. Estos estados de referencia pueden actualizarse en el futuro; el framework siempre compara contra el estado **actual** al momento de investigar.

---

## TAXONOMÍA DE ESTADOS (KB v2.0)

Obligatoria para cualquier afirmación factual importante (no para secciones metodológicas, enlaces o ejemplos):

| Estado | Definición | Ejemplo |
|--------|-----------|---------|
| **VERIFIED** | Confirmado contra código/archivo | `super(39)` en SOURCE y RUNTIME |
| **OBSERVED** | Visto en ejecución RUNTIME | (reservado para observaciones en vivo) |
| **UNVERIFIED** | Afirmado sin comprobación directa | Conteo "3 .htm + 11 .html" de una instrucción (vs. 4 .htm + 10 .html verificado) |
| **ASSUMPTION** | Inferencia razonable sin prueba directa | Suposición de que un mob dropea X |
| **UNKNOWN** | No investigado | Textos del cliente (cifrado en `*.dat`) |
| **DEPRECATED** | Obsoleto | Ninguno en casos actuales |
| **CONFLICT** | Discrepancia entre fuentes | Packages `entity.actor.*` (SOURCE) vs `model.actor.*` (RUNTIME) |
> **Regla**: Un mecánico específico documentado en una quest **NO** se convierte en regla universal. Solo en **patrón reutilizable bajo sección M** si la quest lo demuestra.

---

## PLANTILLA A-W

Cada investigación de quest sigue esta estructura. **No todas las secciones deben estar presentes** — marcar ABSENT / NOT_APPLICABLE según corresponda.

> **Nota**: Cada afirmación factual importante indica su **Estado**, **Procedencia** y **Evidencia** (cuando corresponda). Las secciones metodológicas, enlaces y ejemplos no requieren estos campos.

---

### A. Identificación

| Campo | Valor | Procedencia | Estado |
|-------|-------|-------------|--------|
| Quest ID | | SOURCE `super(id)` / RUNTIME `super(id)` | VERIFIED |
| Nombre (código) | | SOURCE + RUNTIME (carpeta y clase) | VERIFIED |
| Nombre mostrado | | server-side (en código fuente) | VERIFIED |
| Nombre en CLIENTE | | CLIENT `questname-e.dat` | UNKNOWN (cifrado) |
| Nivel mínimo | | SOURCE/RUNTIME `MIN_LEVEL` | VERIFIED |
| Tipo de quest | | Análisis de callbacks (onKill, onTalk, onEvent) | VERIFIED |

### B. Scripts / ubicación

| Artefacto | SOURCE | RUNTIME | Estado |
|-----------|--------|---------|--------|
| `.java` | `UPSTREAM/.../dist/game/data/scripts/quests/Q000XX_*/` | `L2J_Mobius_CT_2.6_HighFive/game/data/scripts/quests/Q000XX_*/` | VERIFIED |
| HTML (N) | presente | presente (mismo conteo) | VERIFIED |
| Otros assets | | | ABSENT / NOT_APPLICABLE |

- Clase: `Q000XX_Name extends Quest`.
- Constructor: `super(id)`, `addStartNpc(…)`, `addTalkId( …)`, `registerQuestItems(…)`.

### C. NPCs

| Nombre | ID | Tipo | Título | SOURCE | RUNTIME |
|--------|----|------|-------|--------|---------|
| | | | | script + `stats/npcs/*.xml` | idem |

- Constantes del script: VERIFIED.
- Nombres/títulos en `stats/npcs/*.xml` (RUNTIME): VERIFIED.
- Correlación cliente (`NpcName-e.dat`): UNKNOWN (cifrado).
- **NPCs narrativos** (mencionados en HTML pero no en `addStartNpc`/`addTalkId`): distinguir como LORE, no como interactivos.

### D. Mobs

| Nombre/ID | ID | Tipo | Fuente | Estado |
|-----------|----|------|--------|--------|
| | | | `addKillId` | VERIFIED |

- Registro: `addKillId(…)` — PRESENT si hay kills, ABSENT si no.
- Correlación cliente (`mobname-e.dat` o NpcName-e.dat): UNKNOWN (cifrado).

### E. Items

| Item | ID | Tipo | Nombre (server) | Constante | Cantidad objetivo |
|------|----|------|-----------------|-----------|------|
| | | | | | |

- Items usan `int` (Q00001) o `ItemHolder` (Q00039). Ambos son válidos.
- Quest items registrados via `registerQuestItems(…)`.
- Correlación cliente (`itemname-e.dat`): UNKNOWN (cifrado).

### F. Recompensas

| Tipo | Valor | Procedencia | Estado |
|------|-------|-------------|--------|
| Item | | SOURCE/RUNTIME | VERIFIED |
| EXP | | SOURCE/RUNTIME | VERIFIED |
| SP | | SOURCE/RUNTIME | VERIFIED |
| Adena | | SOURCE/RUNTIME | VERIFIED |
| Otro | | | ABSENT / NOT_APPLICABLE |

- Exit quest: `exitQuest(false, true)`.
- Sonidos: PRESENT / ABSENT.

### G. Estados / condiciones

| Cond | Trigger | HTML |
|------|---------|------|
| CREATED | — | `npcId-01.html` |
| 1 | `startQuest()` / `onEvent` | |
| 2 | `setCond(2, true)` | |
| ... | | |
| COMPLETED | `exitQuest(false, true)` | |

### H. Flujo de diálogo

| Diálogo | NPC | Cond | Estado previo | Acción | Estado siguiente | HTML destino |
|---------|-----|------|----------|--------|---------|--------------|
| | | | | | | |

- Bypass pattern: `bypass -h Quest QuestName npcId-page.html` → `onEvent`.
- Evento solo HTML: `htmltext = event;`.
- Evento con acción: `startQuest()`, `giveItems(…)`, `setCond(…)`, etc.

```

### I. Mecánicas core

| Mecánica | Uso en esta quest | API core | Referencia doc | Estado |
|----------|-------------------|---------|----------------|--------|
| `giveItemRandomly` | | Quest.java L4936 | QUEST_REWARDS.md §2.2 | VERIFIED |
| `getRandomPartyMemberState` | | Quest.java | Q00039 §3.1 | VERIFIED |
| `giveItems` | | Quest.java L4782 | QUEST_REWARDS.md §1 | VERIFIED |
| `takeItems` | | Quest.java L5011 | QUEST_REWARDS.md §1.4 | VERIFIED |
| `addExpAndSp` | | Quest.java L5147 | QUEST_REWARDS.md §2 | VERIFIED |
| `hasItem` | | Quest.java | Q00039 §3.2 | VERIFIED |
| `hasQuestItems` | | Quest.java | Q00001 uso | VERIFIED |
| `hasAllItems` | | Quest.java | Q00039 uso | VERIFIED |

### J. Archivos HTML

| Archivo | NPC | Cond | Clue | Lore |
|---------|-----|------|------|------|
| | | | EXPLICIT/INFERENCE/STATE | GAMEPLAY/LORE |

- Naming convention: `npcId-pagNum.html` / `.htm`.
- `.htm` usado en bypass de aceptación/inicio; `.html` para respuestas de NPC.

### K. SOURCE vs RUNTIME

| Aspecto | SOURCE (UPSTREAM) | RUNTIME | Diferencia | Estado |
|---------|-------------------|---------|------------|--------|
| License header | | | | |
| Imports/packages | | | | |
| `@author` | | | | |
| Constantes | | | | |
| Lógica (callbacks) | | | | |
| HTML count | | | | |

> **No asumir que toda quest tiene CONFLICT.** Q00001 y Q00039 tuvieron conflictos de naturaleza distinta:
> - Q00001: **diferencia de lógica** (NewbieGuide en SOURCE, no en RUNTIME).
> - Q00039: **diferencias estéticas** (packages, licencia, author) pero **lógica idéntica**.
>
> Siempre registrar la **naturaleza** del conflicto (lógica vs formato).

### L. CLIENT

| Entidad | ID | Server (claro) | Cliente (archivo) | Estado |
|---------|----|----------------|-------------------|--------|
| Quest | | | `questname-e.dat` | UNKNOWN (cifrado) |
| NPC | | | `NpcName-e.dat` | UNKNOWN (cifrado) |
| Item | | | `itemname-e.dat` | UNKNOWN (cifrado) |
| NpcString | | | `NpcString-e.dat` | UNKNOWN (cifrado) |
| Diálogos | | HTML server-side | `L2Text/` (server render) | Verificado en server HTML |

> **No asumir** que todos los textos visibles están en el cliente. Los HTML del datapack server-side pueden contener diálogos y pistas. Solo marcar CLIENT como UNKNOWN si el archivo cliente está cifrado y no se puede verificar.

### M. Patrones reutilizables

| Patrón | Descripción | Generalizable | Aplicado en |
|--------|-------------|---------------|-------------|
| | | | |

### N. Taxonomía / status

| Dimensión | Estado | Evidencia |
|-----------|--------|-----------|
| Server logic | VERIFIED / CONFLICT | SOURCE + RUNTIME comparados |
| HTML diálogos | VERIFIED | Archivos leídos |
| CLIENT textos | UNKNOWN | Cifrado |
| SOURCE↔RUNTIME | VERIFIED / CONFLICT | Comparados |

### O. Trazabilidad

Formato estándar para cada hallazgo:

```
Fuente: <ruta/archivo>
Clase: <clase>
Método: <método>
Líneas: <inicio>–<fin>
Procedencia: SERVER_SOURCE | SERVER_RUNTIME | CLIENT | HTML
Estado: VERIFIED | ... | CONFLICT
```

Ejemplo:
```
Fuente: L2J_Mobius_CT_2.6_HighFive/game/data/scripts/quests/Q00039_RedEyedInvaders/Q00039_RedEyedInvaders.java
Clase: Q00039_RedEyedInvaders
Método: onKill
Líneas: 189-254
Procedencia: SERVER_RUNTIME
Estado: VERIFIED
```


### P. PLAYER CLUES

Metodología para reconstruir: QUEST STATE + NPC DIALOGUE + EXPLICIT CLUE + PLAYER INFERENCE → NEXT OBJECTIVE.

| Fuente | Tipo | Texto/Descripción | NPC | Cond/Fase | Objetivo implícito | Verificación cruzada |
|--------|------|-------------------|-----|-----------|-------------------|---------------------|
| HTML / State / Gameplay | EXPLICIT / INFERENCE / STATE / CODE-ONLY | | | | | |

**Tipos de clue:**
- **EXPLICIT**: El NPC lo dice directamente. Cita literal del HTML.
- **INFERENCE**: El jugador deduce algo que el NPC sugiere indirectamente (razonamiento del jugador, no del investigador).
- **STATE**: Información visible en UI (progreso de items, cond actual).
- **CODE-ONLY**: Lo que solo sabemos mirando código (NO es un clue del jugador; va en trazabilidad).

### Q. LORE

Separar estrictamente:

| Tipo | Definición | Ejemplo |
|------|-----------|---------|
| **Gameplay Fact** | Mecánica verificable en código | "Dropea 100 collares (7178)" |
| **Player Clue** | Información accesible al jugador (del HTML) | "Maten a los Lizardmen" |
| **Lore/Narrative** | Historia/contexto del mundo del diálogo | "Conspiración entre Lizardmen y arañas" |
| **Inference** | Interpretación del investigador | "Magister Rohmer investiga la magia" (solo se menciona su nombre) |

> **Regla de oro**: Una interpretación narrativa NO debe presentarse como hecho.


### R. Quest Assistant data

Datos que un futuro Quest Assistant solicitaría (NO implementar — solo definir):

| Dato | Fuente | Comentario |
|------|--------|------------|
| Quest state | QuestState.getState() | CREATED / STARTED / COMPLETED |
| Player context | QuestState.getCond() | Condición numérica actual |
| Clues | HTML + onTalk | Diálogos que indican objetivo |
| Next objective | Estado + cond + HTML | Qué hacer ahora |
| NPC | addStartNpc / addTalkId | Start NPC, Talk NPCs |
| Mobs | addKillId | Mobs a matar |
| Items | registerQuestItems | Items de colección (ID, count, limit) |
| Locations | Texto HTML | Lugares mencionados en diálogos |
| Drop logic | onKill / giveItemRandomly | chance, limit, playerChance, branching |
| Reward logic | giveItems / addExpAndSp / giveAdena | Items, EXP, SP, Adena |
| Client info | CLIENT `*.dat` | Nombres localizados (si accesibles) |
| Confidence/status | Taxonomía (N) | VERIFIED / UNKNOWN / CONFLICT |

### S. Token efficiency

| Práctica | Aplicación |
|----------|------------|
| MASTER_INDEX first | Router inicial, no releer KB entera |
| Reutilizar docs core | No releer `Quest.java` si `QUEST_REWARDS.md` documenta la API |
| Leer Java selectivamente | Solo callbacks relevantes si script > 200 líneas |
| Cachear patrones | Una vez identificado, reutilizar documentación existente |
| Ir al código | Nunca sustituir evidencia por suposición |

### T. Ejemplo

Ver **EJEMPLO HIPOTÉTICO** al final de este documento (sección dedicada). El ejemplo demuestra cómo aplicar la plantilla A-W con datos ficticios; **no** es una investigación real y no debe usarse como evidencia del proyecto.

### U. Verificación

Checklist antes de publicar una investigación:

- [ ] Estado + Procedencia + Evidencia en cada afirmación factual importante (no en secciones metodológicas/enlaces/ejemplos)
- [ ] SOURCE ↔ RUNTIME comparados; CONFLICT registrados con su naturaleza (lógica vs formato)
- [ ] CLIENT investigado (UNKNOWN si cifrado)
- [ ] Trazabilidad registrada (archivo + clase + método + líneas) para hallazgos clave
- [ ] PLAYER CLUES distinguen EXPLICIT / INFERENCE / STATE / CODE-ONLY
- [ ] LORE separado de Gameplay Fact e Inference
- [ ] Patrones específicos no generalizados como universales
- [ ] Componentes marcados PRESENT / ABSENT / NOT_APPLICABLE / UNKNOWN / UNVERIFIED según corresponda
- [ ] Enlaces internos validados
- [ ] Consistente con [../AI_INSTRUCTIONS/VERIFICATION_RULES.md](../AI_INSTRUCTIONS/VERIFICATION_RULES.md)

### V. Enlaces

| Tema | Documento |
|------|-----------|
| Arquitectura core de quests | [QUEST_ARCHITECTURE.md](QUEST_ARCHITECTURE.md) |
| Ciclo de vida / carga / registro | [QUEST_LIFECYCLE.md](QUEST_LIFECYCLE.md) |
| Eventos y callbacks | [QUEST_EVENTS.md](QUEST_EVENTS.md) |
| Estados, cond, variables, persistencia | [QUEST_STATES_VARIABLES.md](QUEST_STATES_VARIABLES.md) |
| Recompensas | [QUEST_REWARDS.md](QUEST_REWARDS.md) |
| Diálogo Player↔NPC↔HTML | [QUEST_PLAYER_NPC_DIALOG.md](QUEST_PLAYER_NPC_DIALOG.md) |
| Timers | [QUEST_TIMERS.md](QUEST_TIMERS.md) |
| Caso Q00001 (piloto CLIENT↔SERVER) | [../CLIENT_RESEARCH/QUEST_PILOT_Q00001.md](../CLIENT_RESEARCH/QUEST_PILOT_Q00001.md) |
| Caso Q00039 (análisis completo) | [Q00039_REDEYEDINVADERS_ANALYSIS.md](Q00039_REDEYEDINVADERS_ANALYSIS.md) |
| Baseline upstream | [../VERSIONING/UPSTREAM_BASELINE.md](../VERSIONING/UPSTREAM_BASELINE.md) |
| Reglas de verificación | [../AI_INSTRUCTIONS/VERIFICATION_RULES.md](../AI_INSTRUCTIONS/VERIFICATION_RULES.md) |
| Mapeo autoridad CLIENT↔SERVER | [../CLIENT_RESEARCH/CLIENT_SERVER_MAPPING.md](../CLIENT_RESEARCH/CLIENT_SERVER_MAPPING.md) |

### W. Fuente / estado final

Cierre de cada investigación + resumen de confianza (integrado aquí, sin ampliar la plantilla):

| Campo | Valor |
|-------|-------|
| Quest investigada | (especificar ID + nombre) |
| Baseline SOURCE usado | commit upstream vigente al momento (ver [../VERSIONING/UPSTREAM_BASELINE.md](../VERSIONING/UPSTREAM_BASELINE.md)) |
| Baseline RUNTIME usado | build vigente al momento |
| CLIENT | estado del descifrado al momento |
| Resumen de confianza | p.ej. "server VERIFIED · cliente UNKNOWN · N conflictos registrados" |
| Última actualización | fecha |
| Próximos pasos pendientes | (especificar) |


---

## EJEMPLO HIPOTÉTICO — APLICACIÓN DEL FRAMEWORK

> **HIPOTÉTICO · DATOS FICTICIOS · NO ES UNA INVESTIGACIÓN REAL · NO UTILIZAR COMO EVIDENCIA DEL PROYECTO**
>
> Los datos de esta sección son inventados exclusivamente para demostrar cómo aplicar la plantilla A-W. Ninguna etiqueta `DEMO` debe interpretarse como `VERIFIED`. Esta quest **no existe** en el codebase.

### Quest demo: Q00900_EchoesOfTheSwamp (ficticia)

**A. Identificación**

| Campo | Valor | Procedencia | Estado |
|---|---|---|---|
| Quest ID | 900 | SOURCE/RUNTIME `super(900)` | DEMO |
| Nombre (código) | `Q00900_EchoesOfTheSwamp` | SOURCE + RUNTIME | DEMO |
| Nombre mostrado | "Echoes of the Swamp" | server-side | DEMO |
| Nombre en CLIENTE | — | CLIENT | UNKNOWN (cifrado) |
| Nivel mínimo | 25 | `MIN_LEVEL = 25` | DEMO |
| Tipo | Kill + colección simple | callbacks | DEMO |

**B–F (resumen demo)**: script en `quests/Q00900_EchoesOfTheSwamp/`; NPC start MarshWarden (30777), turn-in AlchemistDria (30778); mob FenMarshSpider (21520) vía `addKillId`; items `SWAMP_VENOM = new ItemHolder(8123, 50)`; recompensa: item ficticio ×1 + EXP/SP.

**G–I (resumen demo)**: cond 0→1 (`startQuest` desde bypass) → cond 2 cuando `giveItemRandomly(..., limit=50)` retorna true → entrega con `hasAllItems` → `exitQuest(false, true)`. Mecánica core usada: `giveItemRandomly` [API real documentada en QUEST_REWARDS.md].

**J (demo)**: 5 HTML: `30777-01.htm` (inicio), `30777-02.html`, `30778-01.html` (progreso), `30778-02.html` (entrega), `30778-03.html` (falta venom).

**K (demo)**: SOURCE↔RUNTIME comparados — sin diferencias de lógica; licencia y packages coinciden → sin CONFLICT. *(En el mundo real podría haber conflictos o no: siempre comparar, nunca presuponer el resultado.)*

**L (demo)**: nombres en `itemname-e.dat` / `NpcName-e.dat` → UNKNOWN (cifrado).

**P (demo PLAYER CLUES)**

| Fuente | Tipo | Texto | Objetivo implícito |
|--------|------|-------|--------------------|
| `30777-01.htm` | EXPLICIT | "The marsh spiders' venom has been poisoning the wells." | Matar FenMarshSpiders |
| State | STATE | 37/50 venoms | Faltan 13 |

**Q (demo LORE)**

| Afirmación | Tipo | Estado |
|-----------|------|--------|
| "Venom poisons the wells" (texto del HTML) | Player Clue | DEMO |
| "Las arañas fueron corrompidas por un antiguo ritual" (interpretación del investigador a partir del nombre del mob) | Inference | ASSUMPTION (demo) |
| Dropea veneno hasta límite 50 | Gameplay Fact (patrón real `giveItemRandomly`) | DEMO |

**N/W (demo)**: resumen de confianza → server logic DEMO-VERIFIED contra patrón real · cliente UNKNOWN · 0 conflictos registrados.

> Fin del ejemplo ficticio. Todo lo anterior usa IDs, nombres e items **inventados**; las únicas piezas reales son los patrones de API citados (`giveItemRandomly`, `hasAllItems`, `exitQuest`), que están verificados en la KB core.

---

## VERIFICACIÓN CONCEPTUAL CONTRA LOS CASOS MODELO

El framework puede representar ambos casos sin pérdida de información:

- **Q00001** (simple): D (mobs) = ABSENT · I (mecánicas) mínimas · K = CONFLICT de **lógica** (integración NewbieGuide solo en SOURCE). Todas las secciones aplican.
- **Q00039** (compleja): D y M completas · K = CONFLICT de **formato** (packages/licencia/@author) con lógica idéntica · P/Q extraen clues y lore distribuidos en 14 HTML.

La plantilla distingue correctamente qué es generalizable (metodología, registro, trazabilidad) de lo que no lo es (mecánicas particulares). No se requiere re-investigación de Q00039 para validar esto.

---

## RIESGOS Y MITIGACIONES

| Riesgo | Descripción | Mitigación |
|--------|------------|------------|
| Over-generalización | Convertir mecánicas específicas (NewbieGuide, party weighting) en reglas universales | Sección "Patrones específicos" explícita + columna "¿Generalizable?" en M |
| Duplicación de docs | Re-investigar lo ya documentado | §S token efficiency + MASTER_INDEX como router |
| Presumir igualdad SOURCE≈RUNTIME | Dar por hecho que coinciden | §K obliga a comparar siempre, pero **no presupone el resultado** ni el tipo de conflicto |
| Inventar contenido cliente | Rellenar huecos de `*.dat` cifrados | Regla UNKNOWN estricta (§L) |
| Mezclar lore y gameplay | Presentar interpretación narrativa como hecho | Taxonomía §Q + regla de oro |
| Plantilla pesada | Aplicar A-W completo a quests triviales | Marcar ABSENT/NOT_APPLICABLE; secciones metodológicas no requieren metadatos |
| Version drift | Baselines obsoletos con el tiempo | §W documenta referencia vigente por investigación; rotación vía VERSIONING |

---

**Status**: VERIFIED — metodología modelada contra Q00001 + Q00039  
**Verified**: 2026-08-25  
**Fuente**: docs core QUESTS/* existentes, Q00039_REDEYEDINVADERS_ANALYSIS.md, QUEST_PILOT_Q00001.md, VERIFICATION_RULES.md, UPSTREAM_BASELINE.md  
**Nota**: este documento define metodología; las afirmaciones sobre quests concretas citan sus documentos fuente, donde reside la evidencia original.
> Los valores de baseline son **estados de referencia actuales**, no permanentes: el upstream puede avanzar y el runtime puede cambiar. Cada investigación documenta contra qué referencia trabajó.