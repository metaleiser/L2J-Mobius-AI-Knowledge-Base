# ANÁLISIS DE QUEST: Q00039_RedEyedInvaders

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Quest ID**: 39  
**Nombre**: Red-eyed Invaders  
**Nivel mínimo**: 20  
**Capa**: Datapack Quest — Análisis Transversal CLIENT ↔ SERVER ↔ GAMEPLAY  

**Sources of Truth** (raíces de entidad — no únicamente `dist/`):
- **SERVER_SOURCE**: `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive/` — script en `dist/game/data/scripts/quests/Q00039_RedEyedInvaders/`
- **SERVER_RUNTIME**: `L2J_Mobius_CT_2.6_HighFive/` — script en `game/data/scripts/quests/Q00039_RedEyedInvaders/`
- **CLIENT**: `Lineage2-TCT-273-client/` (investigación de datos / descifrado pendiente)

**Procedencia de datos**: cada afirmación indica explícitamente su origen — `SOURCE` (código UPSTREAM), `RUNTIME` (servidor desplegado), `CLIENT` (cliente H5), `HTML` (diálogos), `GAMEPLAY` (mecánica jugable), `LORE` (narrativa).

**Verified**: 2026-08-25  
**Status**: VERIFIED  

---

## 1. RESUMEN EJECUTIVO

Estudio transversal completo de la quest **Q00039_RedEyedInvaders**, utilizada como segundo caso de estudio (tras Q00001 y Q00152) para modelar mecánicas de:
- **Drops en party con ponderación de killer** (`getRandomPartyMemberState` con exact condition match) [VERIFIED · SOURCE / RUNTIME]
- **Drops con branching aleatorio** (el mismo mob elige entre dos rutas de drop vía `getRandomBoolean()`) [VERIFIED · SOURCE / RUNTIME]
- **Triggers de condición por completitud de colecciones** (`giveItemRandomly` con limit + `hasItem` con verificación de cantidad completa) [VERIFIED · SOURCE / RUNTIME]
- **Pistas en diálogos distribuidos en múltiples fases** (14 archivos HTML/HTM con pistas explícitas) [VERIFIED · HTML]

---

## 2. ESTRUCTURA Y CONSTANTES (GAMEPLAY FACT)

### 2.1 Datos generales [VERIFIED · SOURCE / RUNTIME]
- **Quest ID**: 39 (`super(39)`)
- **Nivel mínimo**: 20 (`MIN_LEVEL = 20`)
- **Start NPC**: Guard Babenco (30334)
- **Turn-in NPC**: Captain Bathis (30332)

### 2.2 Constantes de código [VERIFIED · SOURCE / RUNTIME]

```java
// NPCs
private static final int CAPTAIN_BATHIA = 30332;
private static final int GUARD_BABENCO  = 30334;

// Mobs
private static final int MALE_LIZARDMAN       = 20919;
private static final int MALE_LIZARDMAN_SCOUT = 20920;
private static final int MALE_LIZARDMAN_GUARD = 20921;
private static final int GIANT_ARANE          = 20925;

// Items de quest (ItemHolder: ID, count)
private static final ItemHolder LIZ_NECKLACE_A = new ItemHolder(7178, 100); // Red Bone Totem Necklace
private static final ItemHolder LIZ_NECKLACE_B = new ItemHolder(7179, 100); // Black Bone Totem Necklace
private static final ItemHolder LIZ_PERFUME    = new ItemHolder(7180, 30);  // Mysterious Incense Pouch
private static final ItemHolder LIZ_GEM        = new ItemHolder(7181, 30);  // Five-colored Beads

// Recompensas (ItemHolder)
private static final ItemHolder GREEN_HIGH_LURE   = new ItemHolder(6521, 60); // Green High-Quality Fishing Lure
private static final ItemHolder BABYDUCK_ROD     = new ItemHolder(6529, 1);  // Babydoll Rod
private static final ItemHolder FISHING_SHOT_NONE = new ItemHolder(6535, 500);// Fishing Shot (No Grade)
```

### 2.3 NPCs del entorno [VERIFIED · SOURCE / RUNTIME / HTML]
- **NPCs interactivos reales**: 2 — Guard Babenco (30334, start NPC) y Captain Bathis (30332, turn-in NPC) [VERIFIED · SOURCE / RUNTIME]
- **Referencias narrativas**: 1 — "Magister Rohmer", mencionado en **dos** archivos HTML (`30332-08.html` y `30332-09.html`) como contexto de historia; **no** es NPC interactivo de esta quest (no aparece en `addStartNpc`/`addTalkId`) [VERIFIED · HTML / LORE / SOURCE]

---

## 3. MECÁNICAS DEL CORE VERIFICADAS (GAMEPLAY FACT)

Verificadas contra `Quest.java` del SERVER_SOURCE (`java/org/l2jmobius/gameserver/mechanics/script/Quest.java`).

### 3.1 `getRandomPartyMemberState(player, condition, playerChance, target)` [VERIFIED · SOURCE]
- **condition = exact match**: el filtro es `qs.isCond(condition)` (vía `checkPartyMemberConditions`); solo el valor `-1` tiene significado especial ("quest iniciada"). No es "condición mínima" ni rango.
- **playerChance = weighted ticket multiplier**: el killer se añade `playerChance` veces a la lista de candidatos; cada otro miembro de party que cumple la condición se añade 1 vez. La selección final es uniforme sobre los tickets → el killer tiene `playerChance`× más probabilidad por ticket, no probabilidad garantizada.
- En esta quest se usa con `playerChance = 3` en todas las llamadas [VERIFIED · SOURCE / RUNTIME].
- Hay chequeo de distancia al target (`ALT_PARTY_RANGE`) tras la selección [VERIFIED · SOURCE].

### 3.2 `hasItem(player, ItemHolder)` [VERIFIED · SOURCE]
- Con `checkCount = true` (default): verifica `getQuestItemsCount(player, item.getId()) >= item.getCount()` — es decir, **cantidad ≥ ItemHolder.getCount()**, no mera presencia del item.
- En Q00039, los `hasItem` del `onKill` usan los `ItemHolder` completos (p.ej. `LIZ_NECKLACE_B` con count 100), por lo que exigen la colección **completa** del item complementario.

### 3.3 `giveItemRandomly(player, npc, itemId, amount, limit, chance, playSound)` [VERIFIED · SOURCE]
- **Respeta `limit`**: si `currentCount >= limit` retorna `true` inmediatamente (sin dar item); si la entrega superaría el límite, recorta `amountToGive = limit - currentCount`.
- **Valor de retorno**: `true` si (a) `limit > 0` y el límite fue alcanzado (incl. ya alcanzado previamente), o (b) `limit <= 0` y se entregaron items; `false` en todos los demás casos (drop fallido, sin capacidad, etc.).
- Aplica `RatesConfig.QUEST_ITEM_DROP_AMOUNT_MULTIPLIER` a las cantidades y bonus de champion si aplica [VERIFIED · SOURCE].

### 3.4 `setCond(3)` — primera colección [VERIFIED · SOURCE / RUNTIME]
- **NO es "tener al menos 1 de cada"**. La lógica verificada es:
  ```java
  giveItemRandomly(..., LIZ_NECKLACE_A.getId(), 1, 100, 0.5, true) && hasItem(player, LIZ_NECKLACE_B)
  ```
  `giveItemRandomly` retorna `true` cuando el jugador **alcanza el límite (100)** del item dropeado, y `hasItem(LIZ_NECKLACE_B)` exige **≥100** del item complementario. Por tanto `setCond(3)` se dispara cuando el jugador **completa la segunda colección** (alcanza 100 del item que acaba de dropear) **y ya tiene completa (≥100) la otra** — patrón límite + item complementario completo.
- Mobs de la primera colección: Male Lizardman (20919, collar A), Male Lizardman Scout (20920, rama A), Male Lizardman Guard (20921, rama B) [VERIFIED · SOURCE / RUNTIME].

### 3.5 `setCond(5)` — segunda colección [VERIFIED · SOURCE / RUNTIME]
- **Mismo patrón** para la segunda colección: `giveItemRandomly` con límite 30 (Incense 7180 o Beads 7181) retorna `true` al completar esa colección, y `hasItem` exige ≥30 del complementario.
- Mobs de la segunda colección: Male Lizardman Scout (20920, rama perfume), Male Lizardman Guard (20921, rama perfume), Giant Arane (20925, beads) [VERIFIED · SOURCE / RUNTIME].
- **Branching aleatorio**: Scout y Guard eligen ruta de drop vía `getRandomBoolean()` — el mismo mob puede dropear item de la primera o de la segunda colección según el resultado [VERIFIED · SOURCE / RUNTIME].

---

## 4. ITEMS DE QUEST (GAMEPLAY FACT) [VERIFIED · SOURCE / RUNTIME]

| ID | Nombre | Constante | Cantidad objetivo |
|----|--------|-----------|-------------------|
| 7178 | Red Bone Totem Necklace | `LIZ_NECKLACE_A` | 100 |
| 7179 | Black Bone Totem Necklace | `LIZ_NECKLACE_B` | 100 |
| 7180 | Mysterious Incense Pouch | `LIZ_PERFUME` | 30 |
| 7181 | Five-colored Beads | `LIZ_GEM` | 30 |

Recompensas: 6521 Green High-Quality Fishing Lure ×60, 6529 Babydoll Rod ×1, 6535 Fishing Shot (No Grade) ×500, más 62366 exp / 2783 sp [VERIFIED · SOURCE / RUNTIME].

---

## 5. FLUJO DE CONDICIONES (GAMEPLAY FACT) [VERIFIED · SOURCE / RUNTIME]

| Cond | Trigger | HTML asociado |
|------|---------|---------------|
| 1 | `startQuest()` desde `30334-03.htm` (Guard Babenco) | 30334-04.html |
| 2 | Hablar con Bathis en cond 1 (`30332-02.html`) | 30332-03.html |
| 3 | Completar colección collares (ver §3.4) | 30332-04.html |
| 4 | Entregar ambos collares completos en `30332-05.html` (`hasAllItems` + `takeAllItems`) | 30332-07.html |
| 5 | Completar colección incense/beads (ver §3.5) | 30332-08.html |
| fin | Entregar en `30332-09.html` → recompensas + `exitQuest(false, true)` | 30332-09.html |

Si faltan items en la entrega: `30332-06.html` (cond 3) / `30332-10.html` (cond 5) [VERIFIED · SOURCE / RUNTIME].

---

## 6. ARCHIVOS HTML (PLAYER CLUE / LORE) [VERIFIED · RUNTIME / SOURCE]

**Total: 14 archivos** en la carpeta de la quest (idéntico en SOURCE y RUNTIME):
- **10 `.html`**: `30332-01` … `30332-10` (Captain Bathis)
- **4 `.htm`**: `30334-01`, `30334-02`, `30334-03`, `30334-04` (Guard Babenco)

> **CONFLICT**: una instrucción de consolidación indicaba "3 .htm + 11 .html". El conteo real verificado en filesystem (SOURCE y RUNTIME) es **4 .htm + 10 .html**. Se documenta el valor verificado; la cifra de la instrucción queda como UNVERIFIED/incorrecta.

Pistas distribuidas en múltiples fases: Babenco introduce la amenaza (`30334-01.htm`, PLAYER CLUE), Bathis guía cada etapa, y la narrativa de la conspiración Maille Lizardmen ↔ arañas gigantes con Magister Rohmer se revela en `30332-08.html` / `30332-09.html` (LORE/NARRATIVE).

---

## 7. CONFLICTOS SOURCE ↔ RUNTIME [VERIFIED]

Diferencias verificadas entre el script de SOURCE (UPSTREAM `dist/`) y RUNTIME (`game/`). **No se interpretan automáticamente como bugs**; se registran como CONFLICT:

| Aspecto | SOURCE (UPSTREAM) | RUNTIME |
|---------|-------------------|---------|
| Licencia cabecera | MIT ("Copyright (c) 2013 L2jMobius") | GPL v3+ |
| Imports | `entity.actor.*`, `entity.item.holders.ItemHolder`, `mechanics.script.Quest` | `model.actor.*`, `model.holders.ItemHolder`, `model.quest.Quest` |
| `@author` | `Janiko` | `janiko` |

La lógica de la quest (constantes, eventos, onKill, condiciones) es **idéntica** en ambos [VERIFIED]. La divergencia de paquetes sugiere que RUNTIME proviene de una generación/refactorización distinta del codebase; requiere conciliación con [../VERSIONING/UPSTREAM_BASELINE.md](../VERSIONING/UPSTREAM_BASELINE.md) antes de cualquier sincronización [UNKNOWN → pendiente].

---

## 8. DIMENSIÓN CLIENT [UNKNOWN]

No se ha verificado aún la correlación con el CLIENT (`Lineage2-TCT-273-client/`): nombres localizados de items 7178–7181, NPCs 30332/30334 y mobs 20919–20921/20925 en `L2text/`, y diálogos equivalentes. Pendiente de descifrado (ver [../CLIENT_RESEARCH/CLIENT_ENCRYPTION.md](../CLIENT_RESEARCH/CLIENT_ENCRYPTION.md)) [UNKNOWN].

---

## 9. PATTERNS USEFUL FOR FUTURE QUEST ASSISTANT

Patrones extraíbles de esta quest para un futuro asistente (NO implementado aquí):

1. **Colección doble con complementario cruzado**: dos pares de items donde el trigger de avance exige completar uno y tener completo el otro → generalizable como "collection pair with cross-check".
2. **Branching de drop por mob**: un mismo mob con `getRandomBoolean()` reparte dos rutas de drop → patrón "mob multi-route drop".
3. **Party drop ponderado**: `getRandomPartyMemberState(killer, cond, 3, npc)` → patrón "killer-weighted party drop" con exact condition match.
4. **Trigger por límite alcanzado**: usar el retorno de `giveItemRandomly` (true = límite alcanzado) como disparador de `setCond` → patrón "advance on collection complete".
5. **HTML como pistas progresivas**: diálogos que revelan lore y guían la siguiente colección → patrón "progressive narrative hints".
6. **NPC narrativo no interactivo**: distinguir referencias LORE (Rohmer) de NPCs registrados (`addTalkId`) → patrón "narrative-only NPC detection".

---

## 10. TAXONOMÍA

- **VERIFIED**: confirmado contra código/archivo indicado.
- **OBSERVED**: visto en ejecución del RUNTIME (no usado aún en este documento).
- **UNVERIFIED**: afirmado sin confirmación (p.ej. conteo "3 .htm + 11 .html" de la instrucción).
- **ASSUMPTION**: inferencia razonable sin prueba directa.
- **UNKNOWN**: no investigado (dimensión CLIENT).
- **DEPRECATED**: obsoleto (ninguno en este documento).
- **CONFLICT**: discrepancia entre fuentes (§6 conteo HTML, §7 SOURCE↔RUNTIME).

---
**Fuente**: `quests/Q00039_RedEyedInvaders/**` (SOURCE y RUNTIME), `mechanics/script/Quest.java`  
**Status**: VERIFIED  
**Verified**: 2026-08-25
