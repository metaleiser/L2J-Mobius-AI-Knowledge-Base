# ROADMAP

**Última actualización**: 2026-08-26 (KB v2.4)

Hoja de ruta del proyecto/KB. Cronológico orientativo. Los ítems marcados con fecha futura son **direcciones**, no compromisos.

---

## Actual (2026-08)

- [x] **KB v2.0** — contexto de proyecto, separación SOURCE/RUNTIME/CLIENT/KB, taxonomía de estados, master index actualizado.
- [x] **KB v2.1** — reconciliación de metadatos y conteos (74 técnicos / 5 project / 9 sistema = 88 .md); documentación e indexación de Fase 3, Q00039, Fase 3D y Q00005.
- [x] **Slices Q00005/Q00003 + consolidación quest** — vertical slices verificados (delivery y combate), `QUEST_PARTY_CREDIT.md`, `WORLD/SPAWN_QUERY_GUIDE.md`, `QUEST_VERTICAL_SLICE_TEMPLATE.md`. *(commit `63c9b40` usó "KB v0.9" como etiqueta de trabajo; versión canónica interna v2.2).*
- [x] **KB v2.2** — consolidación y hardening: `QUEST_ENGINE_REFERENCE.md` (referencia central del motor), [GAPS.md](../GAPS.md) (mapa de cobertura), patrones reutilizables en QUEST_REWARDS, conteo quests corregido (532 .java / 510 carpetas), cross-links quest-party vs drop normal, taxonomía ampliada. 81 técnicos / 5 project / 9 sistema = **95 .md**.
- [x] **KB v2.3** — **SKILL / BUFF SEMANTIC FOUNDATION**: `SKILL_SEMANTIC_REFERENCE.md` (referencia central semántica para Scheme Validator), `ABNORMAL_TYPE_REFERENCE.md` (catálogo de 337 AbnormalType), extensión de EFFECT_SYSTEM (EffectType/EffectFlag, stacking, buff routing), SKILL_TARGETING (AffectScope/AffectObject) y SKILL_DATA_MODEL (categorías y beneficial/harmful 4-capas). GAPS.md e índices actualizados. 83 técnicos / 5 project / 9 sistema = **97 .md**.
- [x] **KB v2.4** — **SCHEME ENGINE / VALIDATOR (diseño, documentation-only)**: creada carpeta `BUFFS/` con `SCHEME_BUFFER_ANALYSIS.md` (SchemeBuffer RUNTIME: DB, handlers, storage CSV, slots reales), `COMMUNITY_BOARD_SCHEME_ANALYSIS.md` y `SCHEME_SYSTEM_COMPARISON.md`; creados `SKILLS/SCHEME_VALIDATOR_DESIGN.md` (clasificaciones + decision tree + evidence matrix) y `SKILLS/SCHEME_VALIDATION_LIFECYCLE.md`. Cross-links en SKILL_SEMANTIC_REFERENCE; bootstrap/manifest/GAPS sincronizados. 88 técnicos / 5 project / 11 sistema = **104 .md** (AUDIT-006). Implementación del validador diferida.
- [x] **Fase 3 (inicio)** — investigación CLIENT H5: creado `CLIENT_RESEARCH/` (estructura, cifrado, caso piloto Q00001, mapeo autoridad). Detectado cifrado de texto del cliente y diferencia NEWBIEGUIDE SOURCE↔RUNTIME (CONFLICT).
- [x] **Fase 3D** — creado `QUESTS/QUEST_RESEARCH_FRAMEWORK.md` (marco A-W de análisis transversal de quests).
- [x] **Q00039 RedEyedInvaders** — análisis transversal SOURCE↔RUNTIME↔CLIENT↔GAMEPLAY.
- [x] **Q00005 Miner's Favor** — análisis SOURCE↔RUNTIME con NEWBIE GUIDE CONFLICT; en `QUESTS/` + referenciado en MASTER_INDEX y QUEST_ARCHITECTURE.
- [ ] **Descifrado de texto del cliente** (bloqueo de Fase 3): determinar una vía segura para leer `questname-e.dat`, `NpcName-e.dat`, `itemname-e.dat`, etc.
- [ ] Verificación puntual de documentos técnicos pendientes (PARTIAL/UNKNOWN/RCV) contra el source.
- [ ] Reconciliación visible de contradicciones de conteos (managers, packets, configs) marcándolas CONFLICT/DEPRECATED.

## Próximos micro-sprints (tras KB v2.2) — hacia Community Board

> Objetivo final de esta línea: que la KB sirva como **especificación verificable** para construir sistemas de gameplay mediante IA. Nada de esto está implementado todavía; es planificación.

| Micro-sprint | Tema | Contenido previsto | Estado |
|---|---|---|---|
| ~~2.5~~ | ~~SKILL / BUFF SEMANTIC FOUNDATION~~ | ~~Base semántica de skills/buffs extraída de SOURCE+XML: tipos de efecto, targets, duraciones, stacking, flags beneficial/non-beneficial.~~ | ✅ Completado (KB v2.3) |
| **2.6** | **SCHEME ENGINE / VALIDATOR** | Motor y validador de schemes sobre la base 2.5. | ✅ **DISEÑO COMPLETADO (documentación-only, KB v2.4)** |
| 2.7 | COMMUNITY BOARD FOUNDATION | Fundamentos del Community Board (bypass, páginas, servicios). | Planificado |
| 2.8 | GAMEPLAY / PLAYTEST LOOP | Bucle de pruebas jugables contra la KB (expected vs actual). | Planificado |

### Requisitos futuros del Scheme Validator (documentar, NO implementar aún)

El validador deberá distinguir, con evidencia de SOURCE/XML, para cada skill/buff:

- party buffs vs self buffs
- augmentation
- weapon-related effects
- summon-only buffs
- clan-related effects
- incompatible buffs
- role/class restrictions
- beneficial vs non-beneficial
- target type
- PvE/PvP applicability
- duración
- stacking
- conflicto entre buffs
- applicability al jugador y al party
- orden del scheme

Clasificaciones objetivo del validador: `VALID · INVALID · SELF_ONLY · PARTY_VALID · SUMMON_ONLY · AUGMENTATION · WEAPON_EFFECT · INCOMPATIBLE · CLASS_RESTRICTED · TARGET_RESTRICTED · REQUIRES_REVIEW`.

### Documentación completada (post-v2.0)
- `CLIENT_RESEARCH/`, `Q00039_REDEYEDINVADERS_ANALYSIS.md`, `QUEST_RESEARCH_FRAMEWORK.md`, `Q00005_MINERSFAVOR_ANALYSIS.md`, slices Q00003/Q00005, `QUEST_PARTY_CREDIT.md`, `WORLD/SPAWN_QUERY_GUIDE.md`, `QUEST_VERTICAL_SLICE_TEMPLATE.md`, `QUEST_ENGINE_REFERENCE.md`, `GAPS.md`.
- **Cliente**: cifrado detectado y registrado; **no descifrado validado**.
- **Micro-Sprint 2.6 (KB v2.4)**: `BUFFS/SCHEME_BUFFER_ANALYSIS.md`, `BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md`, `BUFFS/SCHEME_SYSTEM_COMPARISON.md`, `SKILLS/SCHEME_VALIDATOR_DESIGN.md`, `SKILLS/SCHEME_VALIDATION_LIFECYCLE.md`. Diseño del Scheme Validator documentado (verification-first); implementación diferida.

## Corto plazo (siguientes fases)

- [ ] Documentar sistemas de juego pendientes: CLAN, SIEGE, INSTANCE, ZONE, EVENT, SPAWN (ver MASTER_INDEX ⧗).
- [ ] Catálogos pendientes: `PACKET_INDEX.md`, `CONFIG_REFERENCE.md`, `HANDLER_INDEX.md`, `FILE_TREE.md`.
- [ ] Auditoría periódica contra el source (protocolo en `AI_INSTRUCTIONS/AUDIT_PROTOCOL.md`).

## Medio plazo

- [ ] **Investigación del cliente** (`Lineage2-TCT-273-client`): QUESTS, LORE, DIALOGUES, HTML, NPC, ITEMS, TEXTURES, ICONS, SOUNDS, MAPS, relaciones CLIENT↔SERVER.

## Futuro (direcciones)

- [ ] **Quest Knowledge** multi-perspectiva: SERVER + CLIENT + RUNTIME OBSERVATION + LORE → conocimiento completo de quests.
- [ ] **Quest Assistant** (idea): ayuda a jugadores perdidos en una quest (identificar quest/estado/pistas/próximo paso).
- [ ] **AI features / Bots** (idea): análisis y comparación de sistemas de bots con H5.
- [ ] **Crónicas superiores** (investigación): descubrir características que puedan mejorar la experiencia H5 — **sin portar automáticamente**.

## Principio de inversión

```
descubrir → analizar → comparar con H5 → estudiar dependencias → evaluar → proponer → decidir → implementar (solo si se aprueba)
```

Nunca se **porta** automáticamente una característica de otra crónica ni se implementa una idea solo por estar aquí.
