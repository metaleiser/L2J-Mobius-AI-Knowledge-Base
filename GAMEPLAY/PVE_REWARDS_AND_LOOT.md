# PvE REWARDS & LOOT

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: GAMEPLAY — Sistema de recompensas, drops y loot  
**Source of Truth**: `entity/actor/Attackable.java`, `entity/actor/templates/NpcTemplate.java`  
**Evidence Date**: 2026-08-27 (Sprint 0.7 — Checkpoint 2)  
**Status**: VERIFIED (server-side SOURCE)

---

## 1. REWARD FLOW AFTER MONSTER DEATH

```
Monster death (Creature.doDie)
  → Attackable.calculateRewards(lastAttacker)
    → 1. Calculate damaging parties (group by Party)
    → 2. Determine mostDamageParty vs solo mainDamageDealer
    → 3. doItemDrop(leader/mainDamageDealer)       ← DROPS
    → 4. EventDropManager.doEventDrop()            ← EVENT DROPS
    → 5. calculateExpAndSp per attacker/party       ← EXP/SP
    → 6. Distribute EXP/SP with rates + vitality    ← REWARDS
```

[FACT] Source: `entity/actor/Attackable.java L440-691`

---

## 2. ITEM DROP (doItemDrop)

### Flow [FACT]

```
doItemDrop(NpcTemplate, mainDamageDealer):
  → if mainDamageDealer is null → return
  → player = mainDamageDealer.asPlayer()
  → if player is null → handle fake player drops → return
  → CursedWeaponsManager.checkDrop(this, player)
  → if isSpoiled():
      → _sweepItems = npcTemplate.calculateDrops(DropType.SPOIL, this, player)
  → deathItems = npcTemplate.calculateDrops(DropType.DROP, this, player)
  → For each drop item:
      → Check auto-loot conditions:
          AUTO_LOOT (general)
          AUTO_LOOT_RAIDS (if raid)
          AUTO_LOOT_HERBS (if herb)
          AUTO_LOOT_ITEM_IDS (if in list)
      → If auto-loot: player.doAutoLoot(this, drop) → direct to inventory
      → If not: dropItem(player, drop) → item on ground
      → If isRaid && !isRaidMinion && count > 0:
          → broadcast C1_DIED_AND_DROPPED_S3_S2
```

**Source**: `entity/actor/Attackable.java L1095-1210`

### Drop Types

| Tipo | Método | Propósito |
|---|---|---|
| `DropType.DROP` | `calculateDrops(DROP, ...)` | Drops normales del mob |
| `DropType.SPOIL` | `calculateDrops(SPOIL, ...)` | Items de spoil |
| `DropType.CHAMPION` | (cuando `_champion`) | Champion extra drops |
| `EventDropManager` | `doEventDrop()` | Drops de eventos especiales |

[FACT]

---

## 3. AUTO-LOOT CONFIGURATION [FACT]

| Config | Default | Efecto |
|---|---|---|
| `PlayerConfig.AUTO_LOOT` | false | Auto-loot general de items |
| `PlayerConfig.AUTO_LOOT_RAIDS` | false | Auto-loot para raids |
| `PlayerConfig.AUTO_LOOT_HERBS` | false | Auto-loot y auto-uso de hierbas |
| `PlayerConfig.AUTO_LOOT_ITEM_IDS` | (lista vacía) | Auto-loot para items específicos |

---

## 4. SPOIL SYSTEM

- `Attackable._sweepItems` almacena items de spoil cuando `isSpoiled()` [FACT]
- Integrado con party loot: `RANDOM_INCLUDING_SPOIL` y `BY_TURN_INCLUDING_SPOIL` en `PartyDistributionType` [FACT]
- **Source**: `entity/actor/Attackable.java L1171-1173`

---

## 5. RELEVANT CONFIGURATION [FACT]

| Config | Default | Path |
|---|---|---|
| `RateDropItems` | 1 | `config/RatesConfig.java` |
| `RateDropSpoil` | 1 | `config/RatesConfig.java` |
| `RateDropAdena` | 1 | `config/RatesConfig.java` |
| `ChampionRewardsExpSp` | (varios) | `config/ChampionMonstersConfig.java` |

---

## 6. KNOWN UNKNOWNS

- **DropData XML format**: No verificado en detalle (cómo se define drops en NpcTemplate XML)
- **Drop groups/categories**: Categorización de drops no investigada
- **Spoil skill exacto**: Qué skill aplica el estado "spoiled" — no verificado
- **RUNTIME rates**: Los valores de `Character.ini` en RUNTIME pueden diferir de SOURCE

## Cross-links

- [LEVELING_AND_PROGRESSION.md](LEVELING_AND_PROGRESSION.md) — EXP/SP acquisition
- [PARTY_PVE.md](PARTY_PVE.md) — Loot distribution en party
- [RAID_BOSS_SYSTEM.md](RAID_BOSS_SYSTEM.md) — Raid drops
- [../COMBAT/DEATH_FLOW.md](../COMBAT/DEATH_FLOW.md) — Muerte y rewards
- [../QUESTS/QUEST_REWARDS.md](../QUESTS/QUEST_REWARDS.md) — Quest rewards
- [../SYSTEMS/ITEM_SYSTEM.md](../SYSTEMS/ITEM_SYSTEM.md) — Items

---

**Status**: VERIFIED (server-side SOURCE). Mecánica de drops, spoil y auto-loot documentada.  
**Evidence Date**: 2026-08-27