# QUEST PARTY CREDIT

**Proyecto**: L2J Mobius CT 2.6 HighFive
**Capa**: Quests — mecánicas de crédito de kill en party
**Source of Truth**: `mechanics/script/Quest.java` (SOURCE) — helpers L1979–2257 · config `PlayerConfig.java` L200/L490
**Verified**: 2026-08-26
**Status**: VERIFIED (server-side, desde código) · casos de uso verificados en Q00003 y Q00039

> Propósito: referencia reutilizable para clasificar **quién recibe el progreso/ítem** cuando un mob objetivo muere, sin re-leer el core. La KB orienta; el código manda.

---

## 1. Overloads verificados en `Quest.java`

| Firma | Línea | Semántica |
|---|---|---|
| `public Player getRandomPartyMember(Player player, int cond)` | L1979 | Delega en `(player, "cond", String.valueOf(cond))`. Match **exacto** de la variable `cond`. |
| `public Player getRandomPartyMember(Player player, String var, String value)` | L1995–2057 | Núcleo genérico: match exacto `var == value` + filtros distancia/instancia + selección aleatoria uniforme. |
| `public Player getRandomPartyMember(Player player, Npc npc)` | L2135 | Miembro aleatorio cercano al NPC (solo filtro distancia). |
| `public Player getRandomPartyMemberState(Player player, byte state)` | L2067 | Miembro aleatorio cuyo estado de quest == `state`. |
| `public QuestState getRandomPartyMemberState(Player player, int condition, int playerChance, Npc target)` | L2190 | Versión ponderada: el killer entra `playerChance` veces en la lista de tickets; el resto 1 vez. Condición exacta (`-1` = cualquier quest iniciada). |

Helpers relacionados: `checkPartyMemberConditions(qs, condition, npc)` L2235 · `checkDistanceToTarget(...)` L2200.

## 2. Comportamiento detallado del núcleo (var/value) — L1995–2057

1. `player == null` → `null`.
2. **Sin party** → se evalúa **al propio jugador**: su `QuestState` debe tener `var` con ese valor → devuelve `player` o `null`.
3. **Con party** → construye lista de candidatos:
   - miembro tiene la quest con `var == value` (**equalsIgnoreCase**, match exacto);
   - `partyMember.isInsideRadius3D(target, ALT_PARTY_RANGE)` — target = target del killer o el propio killer;
   - **mismo instanceId** que el killer.
4. Si no hay candidatos → `null`; si hay → `candidates.get(Rnd.get(size))` → **uniforme**.

## 3. Configuración: `ALT_PARTY_RANGE`

| Aspecto | Valor | Fuente |
|---|---|---|
| Campo | `PlayerConfig.ALT_PARTY_RANGE` (L200) | `config/PlayerConfig.java` |
| Lectura | `config.getInt("AltPartyRange", 1500)` (L490) → **default 1500** | ídem |
| SOURCE | `dist/game/config/Player.ini` → `AltPartyRange = 1500` | VERIFIED |
| RUNTIME | `game/config/Character.ini` → `AltPartyRange = 1500` | VERIFIED |
| Uso | filtro de candidatos en L2043 (`isInsideRadius3D`) | VERIFIED |

## 4. Taxonomía de clasificación

| Clase | Definición | Helper típico | Ejemplo verificado |
|---|---|---|---|
| **INDIVIDUAL** | El progreso va siempre al jugador que ejecuta la acción; no se consultan miembros de party. | `getQuestState(killer/player)` directo (+ `isCond`) | Q00005 Miner's Favor (sin party mechanics), Q00291 |
| **PARTY_CREDIT_SHARED_RANDOM** (≈ PARTY_CREDIT_SHARED) | Un miembro **aleatorio uniforme** entre los elegibles (incluido el killer) recibe el ítem/progreso. | `getRandomPartyMember(player, cond)` / `(player,var,value)` / `(player,Npc)` | **Q00003 Will the Seal be Broken?** — `getRandomPartyMember(player, 1)` |
| **KILLER_WEIGHTED** | Selección aleatoria pero con **tickets extra para el killer** (`playerChance > 1`). El killer tiene más probabilidad, no exclusividad. | `getRandomPartyMemberState(killer, cond, chance>1, npc)` | **Q00039 RedEyedInvaders** — `getRandomPartyMemberState(killer, cond, 3, npc)` |
| OTHER / UNKNOWN | Cualquier patrón no cubierto (p.ej. repartir a todos, usar `OnKillNotifyTask` custom, etc.) | — | No documentado aún |

> Regla: no generalizar. Clasificar **por código concreto** de cada quest. Si usa helper de party → SHARED/WEIGHTED según overload; si no lo usa → INDIVIDUAL.

## 5. Preguntas frecuentes respondidas por el código

- ¿Puede recibir el crédito alguien que **no hizo el kill**? **Sí** (SHARED y KILLER_WEIGHTED).
- ¿El killer está incluido como candidato? **Sí**, siempre (en WEIGHTED con peso extra si `playerChance>1`).
- ¿El filtro de condición es mínimo o exacto? **Exacto** (`equalsIgnoreCase` sobre la variable); solo `-1` tiene significado especial en `getRandomPartyMemberState` ("quest iniciada", cualquier cond).
- ¿Importa la instancia? **Sí** (`getInstanceId()` igual en el núcleo var/value; en `_State` vía checks estándar).
- ¿Qué pasa si nadie cumple? `null` → el script normalmente no entrega nada.
- ¿Repite item si ya se tiene? Depende del script, no del helper (p.ej. Q00003 comprueba `!hasQuestItems` antes de dar).

## 6. Casos verificados

### Q00003 — `PARTY_CREDIT_SHARED_RANDOM`
- `member = getRandomPartyMember(player, 1)` (Q00003 .java L96/S · L92/R).
- Solo: propio jugador si `cond=="1"`. Party: uniforme entre miembros cond1 en rango 1500 + misma instancia.
- Item entregado a `member` vía helper local `giveItem(member,...)`.

### Q00039 — `KILLER_WEIGHTED`
- `getRandomPartyMemberState(killer, cond, 3, npc)`: killer con 3 tickets vs 1 por compañero (detalle en [Q00039_REDEYEDINVADERS_ANALYSIS.md](Q00039_REDEYEDINVADERS_ANALYSIS.md) §3.1).

### Q00005 — `INDIVIDUAL`
- Sin helpers de party; todo `giveItems/takeItems` sobre el jugador de la interacción ([Q00005_PVE_VERTICAL_SLICE.md](Q00005_PVE_VERTICAL_SLICE.md) §14).

---

## Trazabilidad
- `Quest.java`: L1979, L1995–2057, L2067, L2135, L2190–2235, L490→`PlayerConfig`.
- Config: `Player.ini` (SOURCE) / `Character.ini` (RUNTIME) = 1500.

## Ver también
- [QUEST_STATES_VARIABLES.md](QUEST_STATES_VARIABLES.md) — cond/variables
- [QUEST_REWARDS.md](QUEST_REWARDS.md) — giveItems/giveItemRandomly/hasQuestItems
- [../WORLD/SPAWN_QUERY_GUIDE.md](../WORLD/SPAWN_QUERY_GUIDE.md) — localizar spawns de los mobs implicados

---
**Status**: VERIFIED (código) · Verified: 2026-08-26
