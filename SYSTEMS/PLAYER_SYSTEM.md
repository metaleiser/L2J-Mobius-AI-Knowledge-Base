# PLAYER SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Fase**: Mapa de dependencias del jugador  
**Source of Truth**: `entity/actor/Player.java`, `entity/actor/Playable.java`, `entity/actor/Creature.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (mapa de dependencias; subsistemas se profundizan en fases posteriores)

---

## 1. CLASE

| Campo | Valor |
|-------|-------|
| **Class** | `Player` |
| **Package** | `org.l2jmobius.gameserver.entity.actor` |
| **Path** | `entity/actor/Player.java` |
| **Extends** | `Playable` |
| **Implements** | (ninguno declarado en cabecera) |

Cadena de herencia real:
```
Player -> Playable -> Creature -> WorldObject
```

Javadoc de `Playable`: "This class represents all Playable characters... Player | Summon".

---

## 2. MAPA DE DEPENDENCIAS

| Dependencia | Clase/sistema | Cómo se referencia | Detalle |
|---|---|---|---|
| **Playable / Creature** | `Playable` | parent | hereda AI, stats/status, skills, efectos |
| **World** | `entity/World.java` | registro/visibilidad/broadcast | se da de alta online; movilidad entre regiones |
| **Inventory** | `PlayerInventory` | campo `_inventory` | contenedor de items del jugador |
| **Freight/Warehouse** | `PlayerFreight`, `PlayerWarehouse` | campos (freight) | almacenes de pct |
| **Summon** | `Summon` (+ Pet/Servitor) | método `getSummon()` | mascotas / invocación |
| **Clan** | `entity/clan/Clan.java` | método `getClan()` | membresía/privilegios |
| **Party** | `entity/groups/Party` | método `getParty()` | grupo temporal |
| **Skills** | `Creature._skills: Map<Integer,Skill>` | herencia | habilidades aprendidas/uso |
| **Quests** | `mechanics/script/QuestState` (`Playable` importa) | estado de quest | progreso de quest por jugador |
| **Network** | `GameClient` | campo `_client` | canal de paquetes |
| **Database** | `CharInfoTable`, `ItemContainer.restore` | carga/persistencia | datos de personaje e items |

---

## 3. CAMPOS IMPORTANTES (verificados; parcial)

- **Sesión/red**: `_client`, `_ip`, `_accountName`, `_isOnline`, `_offlinePlay`, `_enteredWorld`, `_onlineTime`, `_lastAccess`, `_uptime`.
- **Inventario/almacenes**: `_inventory` (`PlayerInventory`), `_freight` (`PlayerFreight`).
- **Social**: `_clan`, `_party`, `_friendList`, `_contactList` (`ContactList`), `_married`, `_partnerId`, `_coupleId`.
- **Clase**: `_subClasses` (`Map<Integer,SubClassHolder>`), `_baseClass`, `_activeClass`, `_classIndex`, `_learningClass`.
- **Vehicle**: `_vehicle` (`Vehicle`), `_mountType`, `_mountNpcId`, `_mountLevel`, `_canFeed`.
- **Transform/cubic**: `_transformation` (`Transform`), `_transformSkills`, `_cubics` (`Map<Integer,Cubic>`).
- **Siege/Oly/Duel**: `_siegeState`, `_siegeSide`, `_isInSiege`, `_inOlympiadMode`, `_olympiadGameId`, `_isInDuel`, `_duelState`, `_duelId`.
- **Enchant**: `_isEnchanting`, `_activeEnchantItemId`, `_activeEnchantSupportItemId`, `_activeEnchantAttrItemId`.
- **PvP**: `_pvpKills`, `_pkKills`, `_pvpFlag`, `_fame`, `_curWeightPenalty`, `_cursedWeaponEquippedId`.
- **Otros**: `_tpbookmarks`, `_premiumItems`, `_bookmarkSlot`, `_fish`, `_lure`, `_autoPlaySettings`, `_negotiated...`.

---

## 4. MÉTODOS IMPORTANTES (verificados)

- `getInventory(): PlayerInventory`
- `getClan(): Clan`
- `getParty(): Party`
- `getSummon(): Summon`
- `getClient(): GameClient`
- `getActiveWeaponInstance() / getSecondaryWeaponInstance(): Item`
- `doRevive()` / `doRevive(double power)`

---

## 5. PARTICIPACIÓN EN EL MUNDO

- Al entrar: se añade a `World._allPlayers` (`World.addObject`); al salir: `World.removeObject`.
- Los jugadores son el “observador” central de visibilidad (`isVisibleFor`/`sendInfo` desde `WorldObject`).
- Nota: el flujo exacto de `Player.enterWorld()` / `World.init()` no se detalló en esta fase (marcar `REQUIRES CODE VERIFICATION` si se requiere).

---

## 6. UNKNOWN / REQUIRES CODE VERIFICATION

- `UNKNOWN`: comportamiento exacto de skills del jugador (fase SKILL).
- `UNKNOWN`: lógica completa de quests por jugador (fase QUEST).
- `REQUIRES CODE VERIFICATION`: orden interno de `spawnMe` en `Player` al entrar.

---
## Ver también

- [COMBAT/ATTACK_FLOW.md](../COMBAT/ATTACK_FLOW.md) — cómo inicia un ataque el jugador
- [AI/TARGET_SYSTEM.md](../AI/TARGET_SYSTEM.md) — target en Creature/AbstractAI
- [AI/AI_ARCHITECTURE.md](../AI/AI_ARCHITECTURE.md) — PlayerAI dentro del árbol AI

---
**Status**: VERIFIED (mapa de dependencias)  
**Verified**: 2026-08-23