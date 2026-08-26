# QUEST VERTICAL SLICE TEMPLATE

**Proyecto**: L2J Mobius CT 2.6 HighFive
**Propósito**: checklist reutilizable para auditar y documentar una quest como vertical slice trazable (Quest → NPC → Mob/Evento → Spawn → Item → Progress → Reward → Player State).
**Origen**: destilado de los slices verificados [Q00005_PVE_VERTICAL_SLICE.md](Q00005_PVE_VERTICAL_SLICE.md) (delivery, sin combate) y [Q00003_PVE_VERTICAL_SLICE.md](Q00003_PVE_VERTICAL_SLICE.md) (combate + party credit).
**Jerarquía**: `SOURCE > RUNTIME > XML > CLIENT > KB`. La KB orienta; el código manda.

---

## Reglas transversales

- Nunca convertir inferencia en VERIFIED.
- Distinguir siempre: SERVER_VERIFIED · PARTIALLY_VERIFIED · UNKNOWN_CLIENT · GAMEPLAY_UNVERIFIED · GAP.
- No confundir item ID con cantidad; quest-item vs reward-item; drop normal vs adquisición de quest-item.
- Lore ≠ zona formal (p.ej. "Strip Mine", "School of Dark Arts" = LORE/CONTEXT).
- Clasificar party-credit por código concreto ([QUEST_PARTY_CREDIT.md](QUEST_PARTY_CREDIT.md)).

## Secciones del slice

| # | Sección | Qué verificar | Dónde mirar (fuente preferida) |
|---|---|---|---|
| 1 | Identity | Quest ID (`super(n)`), nombre, clase, autor, MIN_LEVEL, tipo, repeatable | `.java` S+R (constructor/docstring/onTalk) |
| 2 | Authority | Raíces usadas + baseline | rutas SOURCE/RUNTIME/XML/CLIENT |
| 3 | Prerequisites | Raza/clase/nivel/items previos/quests encadenadas | `.java` (onTalk CREATED) + HTML intro |
| 4 | NPC Chain | start/talk/completion NPCs reales (`addStartNpc`/`addTalkId`), rol de cada uno, lore-only NPCs | `.java` constructor + onTalk |
| 5 | Dialogue | Inventario HTML completo, bypasses, eventos no-op, texto clave | `quests/<Quest>/*.htm(l)` |
| 6 | State Machine | CREATED→STARTED→COMPLETED (+cond), qué método cambia cada estado | `.java` + [QUEST_STATES_VARIABLES.md](QUEST_STATES_VARIABLES.md) |
| 7 | Conditions | nivel/prerequisitos/aceptación/progreso/fin/abandono/reinicio; sin inventar tiers | `.java` |
| 8 | Objectives | Pasos reales, orden (¿obligatorio?), cantidades | `.java` + HTML |
| 9 | Kill/Event Flow | addKillId/onAttack/etc., flujo onKill completo | `.java` + [QUEST_EVENTS.md](QUEST_EVENTS.md) |
| 10 | Quest Items | IDs registrados, cantidades, consumo, limpieza al salir | `.java` registerQuestItems + [QUEST_REWARDS.md](QUEST_REWARDS.md) |
| 11 | Mob Mapping | mob ID → quest item exacto (uno a uno), niveles/nombres | `.java` switch onKill + `stats/npcs/*.xml` |
| 12 | Party/Credit | INDIVIDUAL / SHARED_RANDOM / KILLER_WEIGHTED; radio, instancia, killer incluido | `.java` + [QUEST_PARTY_CREDIT.md](QUEST_PARTY_CREDIT.md) |
| 13 | World/Region | región formal (locId/town/castle), sub-regiones; nombres lore = LORE/CONTEXT | `mapregion/*.xml`, `zones/*.xml` |
| 14 | Spawn Verification | conteo por ID en RUNTIME y SOURCE, coords/heading/respawn, celdas `xx_yy` | [../WORLD/SPAWN_QUERY_GUIDE.md](../WORLD/SPAWN_QUERY_GUIDE.md) |
| 15 | Rewards | item(s) con ID+nombre+cantidad, EXP/SP, Adena, condicionales | `.java` (giveItems/addExpAndSp/giveAdena) + `stats/items/*.xml` |
| 16 | Server Flow | diferencias SOURCE↔RUNTIME (lógica vs cosmético), CONFLICTOS explícitos | diff de ambos `.java` |
| 17 | Client Mapping | qué cruza por ID; todo lo textual = UNKNOWN_CLIENT | [../CLIENT_RESEARCH/CLIENT_ENCRYPTION.md](../CLIENT_RESEARCH/CLIENT_ENCRYPTION.md) |
| 18 | Gameplay Walkthrough | pasos jugables con datos reales | derivado de 1–15 |
| 19 | Solo/Party Behavior | clasificación final + implicaciones prácticas | §12 |
| 20 | Failure/Edge Cases | nivel insuficiente, NPC equivocado, faltan items, completada, party edge cases — solo demostrados | `.java` ramas + HTML |
| 21 | Evidence Matrix | claim → evidencia → archivo/línea → status → confianza | acumulado |
| 22 | Known Gaps | UNKNOWN_CLIENT, gameplay no ejecutado, zonas sin formalizar, otros usos de los mobs | — |
| 23 | Verdict | `VERTICAL_SLICE_VERIFIED` / `_PARTIALLY_VERIFIED` / `_FAILED` + por qué | síntesis |
| 24 | First Playable Test | pasos numerados + Expected vs Actual matrix (⬜ pendientes) | derivado |

## Qué NO debe asumirse jamás

1. Que el nombre de la quest implica su mecánica (leer código).
2. Que un número es cantidad cuando puede ser item ID (y viceversa).
3. Que "aparece en la quest" = "entrega/recibe items" (verificar helpers exactos).
4. Que un spawn existe sin verlo en XML (y contar ambos layouts).
5. Que hay mecánica de party sin helper de party (o viceversa).
6. Que un nombre narrativo es zona/región/grupo formal.
7. Que el cliente muestra X sin descifrar (UNKNOWN_CLIENT).
8. Que el gameplay funciona sin ejecutarlo (PARTIALLY_VERIFIED hasta test real).

---
**Status**: VERIFIED (plantilla basada en 2 slices completados) · Created: 2026-08-26
