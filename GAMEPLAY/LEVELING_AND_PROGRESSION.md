# LEVELING & PROGRESSION SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: GAMEPLAY — Leveling, experiencia y progresión de personaje  
**Source of Truth**: `data/xml/ExperienceData.java`, `data/xml/ExperienceLossData.java`, `entity/actor/stat/PlayerStat.java`, `entity/actor/Player.java`  
**Evidence Date**: 2026-08-27 (Sprint 0.7 — Checkpoint 2)  
**Status**: VERIFIED (server-side desde SOURCE)

---

## 1. EXPERIENCE TABLE

### Clase central: `ExperienceData`

| Propiedad | Valor |
|---|---|
| **Class** | `ExperienceData` |
| **Package** | `org.l2jmobius.gameserver.data.xml` |
| **Path (SOURCE)** | `data/xml/ExperienceData.java` |
| **Tipo** | Singleton (`getInstance()`) |
| **XML source** | `data/stats/players/experience.xml` |
| **Atributos XML** | `maxLevel` (default 85), `maxPetLevel` (default 85) |

**Métodos**: `getExpForLevel(int level)`, `getMaxLevel()`, `getMaxPetLevel()`

**Config**: `PlayerConfig.PLAYER_MAXIMUM_LEVEL` = `config.getByte("MaximumPlayerLevel", (byte) 85)` (++ → 86 interno)  
**Source**: `config/PlayerConfig.java L374-375`

### XP Table (valores de SOURCE `experience.xml`) [FACT]

```
maxLevel=85, maxPetLevel=85
<experience level="N" tolevel="XP_ACUMULADA" />
```

| Level | XP Acumulada | XP para subir (aprox) |
|---|---|---|
| 1 | 0 | 0 |
| 2 | 68 | 68 |
| 5 | 2,884 | ~1,700 |
| 10 | 48,229 | ~17,000 |
| 20 | 835,862 | ~160,000 |
| 30 | 4,555,796 | ~523,000 |
| 40 | 15,422,929 | ~1,578,000 |
## 2. EXPERIENCE LOSS (Death Penalty)

### Clase central: `ExperienceLossData`

| Propiedad | Valor |
|---|---|
| **Class** | `ExperienceLossData` |
| **Path (SOURCE)** | `data/xml/ExperienceLossData.java` |
| **XML source** | `data/stats/players/experienceLoss.xml` |
| **Default** | `1.0` (100%) para cualquier nivel no explicitado |

### Tabla de pérdida XP por muerte (`experienceLoss.xml`) [FACT]

| Level Range | % XP perdido |
|---|---|
| 1 | 10.0% |
| 10 | 8.875% |
| 20 | 7.625% |
| 30 | 6.375% |
| 40 | 5.125% |
| 50-75 | 4.0% |
| 76 | 2.5% |
| 77 | 2.0% |
| 78 | 1.5% |
| 79-85 | 1.0% |

> La pérdida es un **porcentaje de la XP total acumulada** del jugador, no de la XP del nivel actual. [FACT]

### Death XP Penalty Flow [FACT]

```
Player.calculateDeathExpPenalty(killer, atWar)
  → percentLost = ExperienceLossData.getInstance().getPercentLost(getLevel())
  → Modifiers:
      REDUCE_EXP_LOST_BY_RAID  / REDUCE_EXP_LOST_BY_MOB / REDUCE_EXP_LOST_BY_PVP
  → RATE_KARMA_EXP_LOST (si tiene karma)
  → Cap: 10% del nivel actual
  → Festival / War: /4
  → AdventBlessing: sin pérdida (0%)
  → setExpBeforeDeath(getExp())  // preserva XP para restaurar en resurrect
```

**Source**: `entity/actor/Player.java ~L5859`
## 3. SP / SKILL POINTS

- **SP máximo configurable**: `PlayerConfig.MAX_SP` = `config.getLong("MaxSp", 50000000000L)` [FACT]
- **Adquisición**: Misma ruta que EXP en `Attackable.calculateRewards()` [FACT]
- **Quest SP**: `Quest.addExpAndSp(player, exp, sp)` [FACT]

## 4. EXP/SP ACQUISITION FROM COMBAT [FACT]

```
Monster death → Attackable.calculateRewards()
  → Solo:
      calculateExpAndSp(attacker.getLevel(), damage, totalDamage)
      → Level difference penalty
      → ChampionMonstersConfig.CHAMPION_REWARDS_EXP_SP multiplier
      → penalty = servitor exp multiplier or 1.0
      → Overhit bonus
      → calcStat(Stat.EXPSP_RATE, exp, null, null)  // base rate
      → PremiumSystemConfig.PREMIUM_RATE_XP/SP
      → addExpAndSp(addExp, addSp, useVitalityRate())
      → updateVitalityPoints()
      → PcCafePointsManager.givePcCafePoint()
  → Party:
      → collect rewardedMembers in ALT_PARTY_RANGE (1500)
      → partyDmg / totalDamage → partyMul
      → calculateExpAndSp(partyLvl, partyDmg, totalDamage)
      → exp *= partyMul
      → Party.distributeXpAndSp(exp, sp, rewardedMembers, partyLvl, partyDmg, this)
```

**Source**: `entity/actor/Attackable.java L530-683`
## 6. DYNAMIC EXP RATES

### Clase: `DynamicExpRateData`

| Propiedad | Valor |
|---|---|
| **Path** | `data/xml/DynamicExpRateData.java` |
| **Propósito** | Tasas de EXP/SP que varían por nivel del jugador |
| **Estructura** | `float[] _expRates` y `float[] _spRates` |
| **Default** | `_enabled = false` (deshabilitado) [FACT] |

## 7. SUBCLASS PROGRESSION

### Clase: `SubClassHolder`

| Propiedad | Valor |
|---|---|
| **Path** | `entity/actor/holders/player/SubClassHolder.java` |
| **Base level** | `PlayerConfig.BASE_SUBCLASS_LEVEL` = 40 [FACT] |
| **Max subclasses** | `PlayerConfig.MAX_SUBCLASS` = 3 (default) |
| **Noble unlock** | Requiere completar ciertas quests (no investigado) |

## 8. RELEVANT CONFIGURATION [FACT]

| Config | Default | Path | Propósito |
|---|---|---|---|
| `MaximumPlayerLevel` | 85 | `config/PlayerConfig.java L374` | Nivel máximo |
| `BaseSubclassLevel` | 40 | `config/PlayerConfig.java L377` | Nivel mínimo subclass |
| `MaxSubclass` | 3 | `config/PlayerConfig.java L376` | Cantidad subclasses |
| `MaxSp` | 50,000,000,000 | `config/PlayerConfig.java L373` | SP máximo |
| `RateXp` | 1 | `config/RatesConfig.java` | Multiplicador EXP global |
| `RateSp` | 1 | `config/RatesConfig.java` | Multiplicador SP global |
| `RateQuestRewardXp` | 1 | `config/RatesConfig.java` | Multiplicador EXP quest |
| `RateQuestRewardSp` | 1 | `config/RatesConfig.java` | Multiplicador SP quest |
| `RaidbossUseVitality` | true | `config/PlayerConfig.java` | Vitality en raids |
| `AltPartyRange` | 1500 | `config/PlayerConfig.java L490` | Rango party EXP share |

## 9. KNOWN UNKNOWNS

- **Vitality exact values**: Cuántos puntos se ganan/pierden por mob kill — requiere verificar `getVitalityPoints()` en subclases
- **Dynamic rates XML**: No verificado si existen datos XML
- **NpcTemplate rewardExp/rewardSp**: Valores exactos por NPC — requiere leer XMLs de stats/npcs/
- **RUNTIME vs SOURCE**: Los XML de experience existen en ambos (COMMON) pero no se verificó igualdad línea a línea

## Cross-links

- [PARTY_PVE.md](PARTY_PVE.md) — Distribución de EXP en party
- [PVE_REWARDS_AND_LOOT.md](PVE_REWARDS_AND_LOOT.md) — Rewards y drops
- [PVE_CONTENT_MODEL.md](PVE_CONTENT_MODEL.md) — Mapa de contenido PvE por nivel
- [../COMBAT/DEATH_FLOW.md](../COMBAT/DEATH_FLOW.md) — Muerte y penalidades
- [../QUESTS/QUEST_REWARDS.md](../QUESTS/QUEST_REWARDS.md) — Recompensas de quest
- [../SYSTEMS/PLAYER_SYSTEM.md](../SYSTEMS/PLAYER_SYSTEM.md) — Sistema del jugador

---

**Status**: VERIFIED (server-side SOURCE) · **Evidence Date**: 2026-08-27

## 5. VITALITY SYSTEM

- **Activado por**: `useVitalityRate()` (cada criatura decide) [FACT]
- **RaidBoss**: `PlayerConfig.RAIDBOSS_USE_VITALITY` (default true)
- **GrandBoss**: Siempre false
- **Mobs normales**: true (default)
| 50 | 40,154,162 | ~3,348,000 |
| 60 | 126,509,653 | ~18,000,000 |
| 70 | 429,634,523 | ~42,000,000 |
| 76 | 931,275,828 | ~127,000,000 |
| 80 | 3,075,966,164 | ~1,230,000,000 |
| 85 | 13,180,481,103 | ~3,700,000,000 |

> La tabla continúa hasta level 87 en SOURCE, pero `maxLevel` es 85. Los niveles 86-87 están definidos pero no alcanzables sin modificar config. [FACT]