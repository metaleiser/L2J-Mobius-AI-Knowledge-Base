# ROADMAP

**Última actualización**: 2026-08-25 (KB v2.1)

Hoja de ruta del proyecto/KB. Cronológico orientativo. Los ítems marcados con fecha futura son **direcciones**, no compromisos.

---

## Actual (2026-08)

- [x] **KB v2.0** — contexto de proyecto, separación SOURCE/RUNTIME/CLIENT/KB, taxonomía de estados, master index actualizado.
- [x] **KB v2.1** — reconciliación de metadatos y conteos (74 técnicos / 5 project / 9 sistema = 88 .md); documentación e indexación de Fase 3, Q00039, Fase 3D y Q00005.
- [x] **Fase 3 (inicio)** — investigación CLIENT H5: creado `CLIENT_RESEARCH/` (estructura, cifrado, caso piloto Q00001, mapeo autoridad). Detectado cifrado de texto del cliente y diferencia NEWBIEGUIDE SOURCE↔RUNTIME (CONFLICT).
- [x] **Fase 3D** — creado `QUESTS/QUEST_RESEARCH_FRAMEWORK.md` (marco A-W de análisis transversal de quests).
- [x] **Q00039 RedEyedInvaders** — análisis transversal SOURCE↔RUNTIME↔CLIENT↔GAMEPLAY.
- [x] **Q00005 Miner's Favor** — análisis SOURCE↔RUNTIME con NEWBIE GUIDE CONFLICT; en `QUESTS/` + referenciado en MASTER_INDEX y QUEST_ARCHITECTURE.
- [ ] **Descifrado de texto del cliente** (bloqueo de Fase 3): determinar una vía segura para leer `questname-e.dat`, `NpcName-e.dat`, `itemname-e.dat`, etc.
- [ ] Verificación puntual de documentos técnicos pendientes (PARTIAL/UNKNOWN/RCV) contra el source.
- [ ] Reconciliación visible de contradicciones de conteos (managers, packets, configs) marcándolas CONFLICT/DEPRECATED.

### Documentación completada (post-v2.0)
- `CLIENT_RESEARCH/`, `Q00039_REDEYEDINVADERS_ANALYSIS.md`, `QUEST_RESEARCH_FRAMEWORK.md`, `Q00005_MINERSFAVOR_ANALYSIS.md`.
- **Cliente**: cifrado detectado y registrado; **no descifrado validado**.

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
