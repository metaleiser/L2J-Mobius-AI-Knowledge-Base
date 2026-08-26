# AI_KNOWLEDGE_BASE

**Project**: L2J Mobius CT 2.6 HighFive  
**Created**: 2026-08-23  
**Status**: KB **v2.1** (2026-08-25) — SOURCE↔SERVER auditado, 4 entidades documentadas    
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

### `UNKNOWN`
Information that could not be determined from code analysis.

Example:
```
Performance Characteristics: UNKNOWN
(The specific timing requirements are not documented in code comments)
```

### `REQUIRES CODE VERIFICATION`
Information needs to be verified in the actual source code before implementation.

Example:
```
Exact table schema: REQUIRES CODE VERIFICATION
(Check actual database schema files)
```

---

## HOW AN AI SHOULD USE THIS KB

### Rule 1: Start with INDEXES
Always begin at `INDEXES/MASTER_INDEX.md`

### Rule 2: Navigate by Theme
Identify which system your question relates to, then navigate to that document.

Example:
- Question about Player stats? → `SYSTEMS/PLAYER_SYSTEM.md`
- Question about skill casting? → `SKILLS/CAST_FLOW.md`
- Question about network packets? → `PACKETS/PACKET_ARCHITECTURE.md`

### Rule 3: Verify Against Code
Before making ANY change or decision:
1. Consult relevant KB document
2. Locate the actual class in source
3. Read the real source code
4. Make decision based on source code, not KB

### Rule 4: Update KB When Code Changes
If you implement changes to the server code, update the relevant KB documents.

### Rule 5: KB is Never More Current Than Code
The Knowledge Base reflects the state of the codebase at creation time.

Future changes to code are the source of truth immediately.

### Rule 6: Use Cross-References
When a document references another:
- Follow the link to get full context
- Do not assume summary is complete
- Read both documents for understanding

### Rule 7: No Speculation
If something is not documented:
- Mark it as `UNKNOWN`
- Check source code directly
- Ask for code verification if needed
- Do not guess behavior

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
- **Fase 3** (2026-08-25): investigación del cliente H5 iniciada — creado `CLIENT_RESEARCH/` (estructura del cliente, cifrado de texto, caso piloto Q00001, mapeo autoridad CLIENT↔SERVER). Detectado que el texto del cliente está cifrado (bloqueo) y una diferencia SOURCE↔RUNTIME en Q00001 (CONFLICT).
- Previamente: Fase 2F (Database) completada + Fases 2A–2E documentadas (SYSTEMS/, AI/, COMBAT/, SKILLS/, QUESTS/, PACKETS/). Correcciones de auditoría FASE 2: 2026-08-24.

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
