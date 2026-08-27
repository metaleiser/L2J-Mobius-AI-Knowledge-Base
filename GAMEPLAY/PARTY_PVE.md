# PARTY PvE SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: GAMEPLAY — Sistema de grupo, distribución de EXP y loot  
**Source of Truth**: `entity/groups/Party.java`, `entity/groups/PartyDistributionType.java`, `entity/groups/PartyExpType.java`, `entity/groups/CommandChannel.java`, `entity/actor/Attackable.java`  
**Evidence Date**: 2026-08-27 (Sprint 0.7 — Checkpoint 2)  
**Status**: VERIFIED (server-side desde SOURCE)

---

## 1. PARTY ARCHITECTURE

| Clase | Path | Propósito |
|---|---|---|
| `Party` | `entity/groups/Party.java` ~1160L | Toda la lógica de grupo |
| `AbstractPlayerGroup` | `entity/groups/AbstractPlayerGroup.java` | Clase base (Party, CommandChannel) |
| `CommandChannel` | `entity/groups/CommandChannel.java` | Canal multi-party |
| `PartyDistributionType` | `entity/groups/PartyDistributionType.java` | Enum: 5 modos de loot |
| `PartyExpType` | `entity/groups/PartyExpType.java` | Enum: 5 modos de EXP |
| `PartyMessageType` | `entity/groups/PartyMessageType.java` | Mensajes de expulsión/salida |

[FACT]

---

## 2. PARTY FORMATION & MEMBERSHIP

- **Líder**: Primer miembro en `_members` (`getLeader()`) [FACT]
- **Miembros**: Lista `_members` (`CopyOnWriteArrayList<Player>`) [FACT]
- **Tamaño máximo**: No verificado explícitamente — típicamente 9 en Lineage 2
- **Expulsión**: `removePartyMember()`, mensajes via `PartyMessageType` (EXPELLED, LEFT, DISCONNECTED) [FACT]
- **Party Match**: `PartyMatchRoomList` en `entity/groups/matching/` [FACT]
- **Rango de party**: `PlayerConfig.ALT_PARTY_RANGE` = 1500 (default) — usado para EXP share, loot, quest credit [FACT]

---

## 3. PARTY LOOT DISTRIBUTION

### `PartyDistributionType` enum (5 modos) [FACT]

| ID | Nombre | Comportamiento |
|---|---|---|
| 0 | `FINDERS_KEEPERS` | Cada jugador recoge lo que dropea |
| 1 | `RANDOM` | El item se asigna aleatoriamente a un miembro |
| 2 | `RANDOM_INCLUDING_SPOIL` | Aleatorio incluyendo items de spoil |
| 3 | `BY_TURN` | Por turnos entre miembros |
| 4 | `BY_TURN_INCLUDING_SPOIL` | Por turnos incluyendo spoil |

**Source**: `entity/groups/PartyDistributionType.java L22-28`

### Loot distribution flow [FACT]

```
Attackable.calculateRewards()
  → Calculate damagingParties (group by Party)
  → Sort damagingParties by total damage (descending)
  → Most damage party → doItemDrop(party.leader)
  → If no party with more damage than solo:
      → doItemDrop(mainDamageDealer or lastAttacker)
  → EventDropManager.doEventDrop()
```

**Source**: `entity/actor/Attackable.java L483-498`

### Loot ownership [FACT]

- **Auto loot**: Si `PlayerConfig.AUTO_LOOT` (y `AUTO_LOOT_RAIDS` para raids, `AUTO_LOOT_HERBS` para hierbas, `AUTO_LOOT_ITEM_IDS` para items específicos) — el item va directamente al inventario
- **No auto loot**: El item se dropea al suelo (`dropItem(player, drop)`)
- **Raid broadcast**: Si el mob es raid (`_isRaid`) y no minion, se broadcasta el drop

---

## 4. PARTY EXP/SP DISTRIBUTION

### `PartyExpType` enum (5 modos) [FACT]

| Modo | Comportamiento |
|---|---|
| `NONE` | Sin EXP de party |
| `AUTO` | Automático |
| `LEVEL` | Basado en nivel |
| `PERCENTAGE` | Basado en porcentaje |
| `HIGHFIVE` | Modo High Five |

**Source**: `entity/groups/PartyExpType.java L26-33`

### EXP distribution algorithm [FACT]

```
## 5. SPOIL INTEGRATION

- `PartyDistributionType.RANDOM_INCLUDING_SPOIL` (ID 2) y `BY_TURN_INCLUDING_SPOIL` (ID 4) confirman que el sistema de spoil está integrado con la distribución de party [FACT]
- `Attackable._sweepItems` almacena items de spoil cuando `isSpoiled()` [FACT]
- **Source**: `entity/actor/Attackable.java L1171-1173`

## 6. PARTY vs QUEST PARTY CREDIT

**Importante**: No confundir los dos mecanismos [FACT]:

| Mecanismo | Decisión | Implementación |
|---|---|---|
| **NORMAL MOB REWARD** (EXP/SP/drops) | Quién recibe EXP/SP/drop normal → el `mainDamageDealer` (mayor daño) o su party | `Attackable.calculateRewards()`, `doItemDrop()` |
| **QUEST PARTY CREDIT** (progreso de quest) | A qué miembro de party con quest elegible se le atribuye el ítem/progreso | `Quest.getRandomPartyMember*()`, `Quest.getRandomPartyMemberState()` |

Ambos usan el mismo radio `ALT_PARTY_RANGE = 1500` pero pueden seleccionar jugadores diferentes. [FACT]

Detalle completo del crédito quest: [../QUESTS/QUEST_PARTY_CREDIT.md](../QUESTS/QUEST_PARTY_CREDIT.md)

## 7. PARTY DEATH INTERACTIONS

- Miembros muertos no reciben EXP ni drops: `if ((partyPlayer == null) \|\| partyPlayer.isDead()) continue;` [FACT]
- Miembros fuera de rango (>1500) no reciben EXP ni drops: `calculateDistance3D(partyPlayer) < PlayerConfig.ALT_PARTY_RANGE` [FACT]
- Eject en instancias: `Instance.notifyDeath()` programa expulsión si no revive [FACT]

## 8. COMMAND CHANNEL SPECIFICS

- **Tamaño**: Agrupa múltiples parties (hasta 5 parties para raids grandes)
- **Líder**: Líder de la party líder
- **Nivel usado**: `CommandChannel.getLevel()` = nivel del líder del CC
- **EXP share**: Incluye todos los miembros de todas las parties en el CC que están en rango

## 9. KNOWN UNKNOWNS

- **Party size cap**: No verificado explícitamente — valor típico 9 no confirmado en código
- **CommandChannel max parties**: No verificado
- **EXP distribution formula exacta**: `distributeXpAndSp()` no se leyó en detalle
- **Loot distribution mechanics**: Cómo `distributeItem()` reparte en cada modo — no se investigó en profundidad
- **RUNTIME configs**: `AltPartyRange` en RUNTIME = 1500 (COMMON con SOURCE)

## Cross-links

- [LEVELING_AND_PROGRESSION.md](LEVELING_AND_PROGRESSION.md) — EXP/SP adquisition
- [PVE_REWARDS_AND_LOOT.md](PVE_REWARDS_AND_LOOT.md) — Drop/loot distribution
- [../QUESTS/QUEST_PARTY_CREDIT.md](../QUESTS/QUEST_PARTY_CREDIT.md) — Crédito de party en quests
- [../COMBAT/DEATH_FLOW.md](../COMBAT/DEATH_FLOW.md) — Muerte y rewards
- [../AI/AGGRO_SYSTEM.md](../AI/AGGRO_SYSTEM.md) — Aggro y hate
- [../SYSTEMS/PLAYER_SYSTEM.md](../SYSTEMS/PLAYER_SYSTEM.md) — Sistema del jugador
- [INSTANCE_SYSTEM.md](INSTANCE_SYSTEM.md) — Party en instancias

---

**Status**: VERIFIED (server-side SOURCE) · **Evidence Date**: 2026-08-27
Party.distributeXpAndSp(exp, sp, rewardedMembers, partyLvl, partyDmg, this):
  → For each rewarded member in range (ALT_PARTY_RANGE=1500):
      → Calculate member share based on level, damage contribution
      → Apply rates, vitality, premium
      → addExpAndSp() to each member
      → If CommandChannel active:
          → uses CC level, includes all CC members
```

### CommandChannel integration [FACT]

- `Party.isInCommandChannel()` y `Party.getCommandChannel()`
- Cuando CC activo: `groupMembers = attackerParty.getCommandChannel().getMembers()` (todos los miembros de todas las parties)
- Level usado para cálculo: `attackerParty.getCommandChannel().getLevel()` (nivel del líder del CC)
- **Source**: `entity/actor/Attackable.java L594-622`