# AI_KNOWLEDGE_BASE

**Project**: L2J Mobius CT 2.6 HighFive  
**Created**: 2026-08-23  
**Status**: KB **v2.5** (2026-08-27) — Sprint 0.6B (Compressions, SOURCE_ENTRY, Routing)  
**Source of Truth**: según la entidad — código fuente (`SERVER_SOURCE`/UPSTREAM) para arquitectura; servidor desplegado (`SERVER_RUNTIME`) para estado observable; cliente (`CLIENT`) para lo solo-cliente. Ver [00_PROJECT/PROJECT_CONTEXT.md](00_PROJECT/PROJECT_CONTEXT.md).

---

## LAS 4 ENTIDADES (resumen)

| Entidad | Ruta real | Rol |
|---|---|---|
| **SERVER_SOURCE** | `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive` | código fuente (Git GitLab, baseline `e2518ab`) |
| **SERVER_RUNTIME** | `L2J_Mobius_CT_2.6_HighFive` | servidor desplegado/ejecutable (build 26/05/2024, sin Git) |
| **CLIENT** | `Lineage2-TCT-273-client` | cliente H5 |
| **KNOWLEDGE_BASE** | `AI_KNOWLEDGE_BASE` | esta documentación |

> **SOURCE baseline ≠ RUNTIME build**: no son idénticos ni representan el mismo commit.
> Detalles: [00_PROJECT/PROJECT_CONTEXT.md](00_PROJECT/PROJECT_CONTEXT.md) · [VERSIONING/UPSTREAM_BASELINE.md](VERSIONING/UPSTREAM_BASELINE.md)

---

## PURPOSE

This Knowledge Base documents the **L2J Mobius CT 2.6 HighFive** server architecture and systems for AI consumption.

**Goal**: Enable AI agents to understand this complex project without re-analyzing the entire codebase in each query.

---

## CRITICAL RULE: SOURCE OF TRUTH

**The actual source code is always the source of truth.**

This Knowledge Base documents the code structure, but does NOT replace it.

If contradiction exists between this KB and the source code → **code wins**.

---

## WHAT THIS KB CONTAINS

✓ Architecture documentation  
✓ System descriptions  
✓ Component relationships  
✓ Data flow explanations  
✓ Entry points and initialization  
✓ Manager responsibilities  
✓ Cross-linked indexes  

## WHAT THIS KB DOES NOT CONTAIN

✗ Full source code copies  
✗ Complete method signatures  
✗ Invented functionality  
✗ Speculative behavior  

---

## SPECIAL MARKERS

This Knowledge Base uses the evidence taxonomy defined in `AI_INSTRUCTIONS/VERIFICATION_RULES.md`:

- `VERIFIED`
- `SOURCE_REQUIRED`
- `PARTIAL`
- `UNKNOWN`
- `ASSUMPTION`
- `CONFLICT`
- `DEPRECATED`
- `UNKNOWN_CLIENT`

Older markers such as `REQUIRES CODE VERIFICATION` are deprecated. Map them to `UNVERIFIED` or `SOURCE_REQUIRED`.

## HOW AN AI SHOULD USE THIS KB

**For AI agents, the mandatory entry point is `AI_INSTRUCTIONS/AI_BOOTSTRAP.md`.**

Do not start with this README. Use the routing table in `AI_INSTRUCTIONS/AI_MANIFEST.md` to select task-specific documents.

General rules:

1. Start at `AI_INSTRUCTIONS/AI_BOOTSTRAP.md`.
2. Read `GAPS.md` to check coverage.
3. Read `VERSIONING/KB_VERSION.md` for baseline identity.
4. Load only the KB documents relevant to the task.
5. Inspect `SERVER_SOURCE` only when the KB marks information as `SOURCE_REQUIRED`, `PARTIAL`, `UNKNOWN`, or stale.
6. Verify claims against source code before implementing.
7. Update the KB when new reusable knowledge is discovered.
8. The KB is never more current than the source code.

For exact evidence rules and authority hierarchy, see `AI_INSTRUCTIONS/VERIFICATION_RULES.md`.


---

## ORGANIZATION

```
AI_KNOWLEDGE_BASE/
├── README.md (this file)
├── Core Documentation
│   ├── PROJECT_OVERVIEW.md
│   ├── PROJECT_STRUCTURE.md
│   ├── ARCHITECTURE_OVERVIEW.md
│   └── BUILD_AND_DEPLOYMENT.md
├── Server Documentation
│   ├── GAMESERVER_ARCHITECTURE.md
│   ├── LOGINSERVER_ARCHITECTURE.md
│   └── COMMONS_ARCHITECTURE.md
├── AI/ (Inteligencia artificial)
├── COMBAT/ (Sistema de combate)
├── SKILLS/ (Sistema de skills)
├── QUESTS/ (Motor de quests)
├── PACKETS/ (Paquetes cliente↔servidor)
├── SYSTEMS/ (Main game systems)
├── CLIENT_RESEARCH/ (Investigación del cliente H5) ← Fase 3
├── NETWORK/ (Communication)
├── DATABASE/ (Persistence)
├── CONFIGURATION/ (Runtime config)
├── THREADING/ (Concurrency)
├── SCRIPTING/ (Script engine)
└── INDEXES/ (Navigation)
```

---

## DOCUMENTATION STATUS

Estado sincronizado con [INDEXES/MASTER_INDEX.md](INDEXES/MASTER_INDEX.md):
- **KB v2.0** (2026-08-25): módulo `00_PROJECT/` (contexto, fuentes, decisiones, roadmap, ideas), separación de 4 entidades (SOURCE/RUNTIME/CLIENT/KB), build Ant documentado, taxonomía de estados ampliada. Ver [VERSIONING/CHANGELOG.md](VERSIONING/CHANGELOG.md).
- **KB v2.1** (2026-08-25): reconciliación de metadatos y conteos (74 técnicos/5 project/9 sistema = 88 .md), Fase 3, Q00039, Fase 3D `QUEST_RESEARCH_FRAMEWORK`, análisis Q00005 + integración de navegación. Conteos y STATUS validados (AUDIT-003).
- **KB v2.5** (2026-08-27): Sprint 0.6B — compressions, SOURCE_ENTRY conversions, routing optimization. Ver [VERSIONING/CHANGELOG.md](VERSIONING/CHANGELOG.md).
- **KB v2.3** (2026-08-26): base semántica de skills/buffs/effects para el Scheme Validator (`SKILL_SEMANTIC_REFERENCE.md`, `ABNORMAL_TYPE_REFERENCE.md`); catálogos verificados de `EffectType`/`EffectFlag`/`AffectScope`/`AffectObject`; reglas de stacking/reemplazo/slots en `EFFECT_SYSTEM.md`; categorías y beneficial/harmful 4-capas en `SKILL_DATA_MODEL.md`; `GAPS.md` e índices actualizados. **83 técnicos / 5 project / 9 sistema = 97 .md** (AUDIT-005).
- **KB v2.2** (2026-08-26): consolidación del engine de quests (`QUEST_ENGINE_REFERENCE`), mapa de cobertura [GAPS.md](GAPS.md), crédito de party ([QUEST_PARTY_CREDIT](QUESTS/QUEST_PARTY_CREDIT.md)), guía de spawns ([WORLD/SPAWN_QUERY_GUIDE](WORLD/SPAWN_QUERY_GUIDE.md)), plantilla de slices, slices Q00003/Q00005, patrones reutilizables en QUEST_REWARDS, conteo quests corregido (532 java / 510 carpetas), cross-links quest-party vs drop normal, taxonomía ampliada. **81 técnicos / 5 project / 9 sistema = 95 .md** (AUDIT-004).
- **Fase 3** (2026-08-25): investigación del cliente H5 iniciada — creado `CLIENT_RESEARCH/` (estructura del cliente, cifrado de texto, caso piloto Q00001, mapeo autoridad CLIENT↔SERVER). Detectado que el texto del cliente está cifrado (bloqueo) y una diferencia SOURCE↔RUNTIME en Q00001 (CONFLICT).
- Previamente: Fase 2F (Database) completada + Fases 2A–2E documentadas (SYSTEMS/, AI/, COMBAT/, SKILLS/, QUESTS/, PACKETS/). Correcciones de auditoría FASE 2: 2026-08-24.

> Nota: el commit `63c9b40` usó "KB v0.9" como nomenclatura de trabajo; la versión canónica interna es v2.5.

Prioridades pendientes (sin iniciar):
1. ✓ Core architecture
2. ✓ Entry points
3. ✓ Network layer
4. ✓ Database layer
5. ✓ Configuration system
6. ✓ Threading model
7. ⧗ Sistemas mayores restantes (CLAN, SIEGE, INSTANCE, ZONE, EVENT, SPAWN)
8. ⧗ Catálogo completo de managers (ver [INDEXES/MANAGERS_INDEX.md](INDEXES/MANAGERS_INDEX.md))
9. ⧗ PACKET_INDEX.md (referencia de packets)
10. ⧗ NPC type catalog
11. ⧗ CONFIG_REFERENCE.md · HANDLER_INDEX.md · FILE_TREE.md
12. ⧗ Investigación del cliente (QUESTS, LORE, DIALOGUES, HTML, NPC, ITEMS, TEXTURES, ICONS, SOUNDS, MAPS)

Ver hoja de ruta: [00_PROJECT/ROADMAP.md](00_PROJECT/ROADMAP.md).

---

## VERIFICATION

Each document includes:
```
Source of Truth: <file paths>
Verified: <date>
Status: VERIFIED / PARTIAL / UNKNOWN
```

Only consume information marked as VERIFIED.

---

## IMPORTANT DISCLAIMER

This Knowledge Base is provided as-is for documentation purposes.

It represents the state of the project as of 2026-08-23.

Code changes after this date supersede this documentation.

Always verify critical information against the actual source code before implementation.
