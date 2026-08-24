# MONSTER SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Fase**: Monstruos hostiles  
**Source of Truth**: `entity/actor/instance/Monster.java` y subclases  
**Verified**: 2026-08-23  
**Status**: VERIFIED (en la relación extends y conexiones; Combat/AI se profundiza en fases propias)

---

## 1. CLASE `Monster`

- **Class**: `Monster`
- **Package**: `org.l2jmobius.gameserver.entity.actor.instance`
- **Path**: `entity/actor/instance/Monster.java`
- **Extends**: `Attackable`
- **Implements**: —

Constructor:
```java
public Monster(NpcTemplate template) {
    super(template);
    setInstanceType(InstanceType.Monster);
    setAutoAttackable(true);
}
```

Responsibility: monstruo hostil. Gestión de minions (líder con `MinionList`), aggro, comportamiento de spawn, cura de raids.

Campos verificados:
- `_enableMinions` (boolean)
- `_master` (`Monster`) — líder
- `_minionList` (`MinionList`) — minions asociados

Métodos verificados:
- `isAutoAttackable(Creature attacker)` — override
- `isAggressive()` — `getTemplate().isAggressive() && !isAffected(PASSIVE)`
- `onSpawn()`, `onTeleported()`, `deleteMe()`
- `getLeader()` / `setLeader(Monster)`
- `hasMinions()` / `getMinionList()` / `enableMinions(boolean)`
- `isMonster()`, `asMonster()`, `isWalker()`, `giveRaidCurse()`, `doCast(...)`

---

## 2. SUBTIPOS CONFIRMADOS EN FASE 2A

Los siguientes fueron **verificados leyendo su `extends`**:

| Clase | Extends |
|-------|---------|
| `RaidBoss` | Monster |
| `GrandBoss` | Monster (**NO** RaidBoss) |
| `EventMonster` | Monster |
| `RiftInvader` | Monster |
| `FeedableBeast` | Monster |
| `TamedBeast` | FeedableBeast |
| `Chest` | Monster |
| `Block` | Monster |
| `FestivalMonster` | Monster |

> En el enum `InstanceType`, sin embargo, `GrandBoss(RaidBoss)` se define como hijo de RaidBoss. Esto es una diferencia jerárquica **enum vs extends** que debe tenerse presente.

---

## 3. CONEXIONES CON OTROS SISTEMAS

| Sistema | Vía real | Nota |
|---------|----------|------|
| **Spawn** | `NpcTemplate`, `Spawn`, `SpawnData`, `RespawnTaskManager` | se crean con template; respawn automático |
| **AI** | `AttackableAI`, `CreatureAI` | bucle de intención (Idle→Attack→…) |
| **Combat** | `Creature` (stats/effects), `Skill.doCast` | Monster bloquea skills positivas sobre jugadores en `doCast` |
| **Drop** | `ItemManager`, `ItemsOnGroundManager`, `RANDOM_ITEM_DROP_LIMIT` | drop post-muerte |
| **Agro** | `Attackable` (aggro list) | una lista de jugadores que atacaron |
| **Muerte** | `deleteMe()`, efectos/caída | quita minimons del líder |
| **Respawn** | `RespawnTaskManager` + `Spawn` | vuelta al mundo |

---

## 4. NOTA AGRO / FAKE PLAYERS

`Monster.isAutoAttackable` es override:
- No ataca a otros `Monster` salvo que el atacante sea fake player.
- Si `NpcConfig.GUARD_ATTACK_AGGRO_MOB` y aggro, un `Guard` recibe autoataque.
- Solo ataca a `Player`/`Attackable`/`Trap`/`EffectPoint`.

---

## TRAZABILIDAD

- FASE 1 ponía `GrandBoss` bajo `RaidBoss` (clase). Real: `GrandBoss extends Monster`.
- FASE 1 ponía `Monster` bajo `Npc` directo. Real: `Monster -> Attackable -> Npc`.

---
## Ver también

- [AI/AI_ARCHITECTURE.md](../AI/AI_ARCHITECTURE.md) — árbol y ciclo del AI
- [AI/AGGRO_SYSTEM.md](../AI/AGGRO_SYSTEM.md) — aggro/threat real (`_aggroList`, `AggroInfo`)
- [COMBAT/DEATH_FLOW.md](../COMBAT/DEATH_FLOW.md) — muerte, rewards, drops, respawn

---
**Status**: VERIFIED (relaciones); contenido de damage/aggro completo en fase COMBAT  
**Verified**: 2026-08-23