# ROADMAP

**Última actualización**: 2026-08-25 (KB v2.0)

Hoja de ruta del proyecto/KB. Cronológico orientativo. Los ítems marcados con fecha futura son **direcciones**, no compromisos.

---

## Actual (2026-08)

- [x] **KB v2.0** — contexto de proyecto, separación SOURCE/RUNTIME/CLIENT/KB, taxonomía de estados, master index actualizado.
- [ ] Verificación puntual de documentos técnicos pendientes (PARTIAL/UNKNOWN/RCV) contra el source.
- [ ] Reconciliación visible de contradicciones de conteos (managers, packets, configs) marcándolas CONFLICT/DEPRECATED.

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
