# DEATH FLOW

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: COMBAT — flujo de muerte  
**Source of Truth**: `entity/actor/Creature.java`, `entity/actor/Player.java`, `entity/actor/Attackable.java`, `entity/actor/Npc.java`, `entity/actor/instance/Monster.java`, `entity/actor/status/CreatureStatus.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (flujo general)

---

## 1. Quién detecta la muerte

- La muerte la dispara **`CreatureStatus.reduceHp`** (o `PlayerStatus.reduceHp` con CP) cuando el HP baja a `< 0.5` y la criatura es `isMortal()`:

```java
// CreatureStatus.java ~L180
if ((_creature.getCurrentHp() < 0.5) && _creature.isMortal()) {
    _creature.abortAttack();
    _creature.abortCast();
    _creature.doDie(attacker);
}
```

- `Createure.doDie(Creature killer)` es el método central.

---

## 2. Muerte genérica — `Creature.doDie(killer)`

**Path**: `entity/actor/Creature.java` (L2625)

1. **Guarda**: `synchronized(this)`; si `_isDead` ya → `return false`.
2. `setCurrentHp(0)`; `setDead(true)`.
3. **Eventos**:
   - `ON_CREATURE_DEATH` → `new OnCreatureDeath(killer, this)`.
   - `ON_CREATURE_KILLED` → puede `TerminateReturn` (cancelar la muerte).
4. **Rewards**: `mainDamageDealer = isMonster() ? asMonster().getMainDamageDealer() : null;`
   - `calculateRewards(mainDamageDealer != null ? mainDamageDealer : killer)` (solo Attackable).
5. `setTarget(null)`; `stopMove(null)`; `_status.stopHpMpRegeneration()`.
6. **Efectos**:
   - Si `isAttackable()`:
     - stop efectos (según `Spawn.isRespawnEnabled`).
     - **clan help aggro**: si killer playable + `template.getClans()` → a vecinos rango clanHelpRange → `notifyActionAggression(killer, 1)` + `OnAttackableFactionCall`.
   - else `stopAllEffectsExceptThoseThatLastThroughDeath()`.
7. `broadcastStatusUpdate()`; `getAI().notifyActionDeath()`.
8. `ZoneManager.getRegion(this).onDeath(this)`.
9. `getAttackByList().clear()`; abortar channelization.
10. Si `GrandBoss` → anuncios de derrota (`BossAnnouncementsConfig`).

---

## 3. Muerte de `Player` — `Player.doDie(killer)` override

**Path**: `entity/actor/Player.java` (L5242)

Después de `super.doDie`:
- `stopFeed` (si monta), `stopFakeDeath`.
- **KrateiArena**: respawn task + puntos al killer.
- **Evento `ON_PLAYER_PVP_KILL`** (`OnPlayerPvPKill`).
- **PvP/PK item rewards** (`PvpRewardItemConfig`).
- **Anuncio PvP/PK** (`PvpAnnounceConfig`).
- **Karma** para fake-player killer (`FakePlayersConfig`, +150).
- **Cursed weapon / combat flag drop**.
- `onDieDropItem(killer)`:
  - evita si event / war / PvP zone;
  - **karma drop** (`KARMA_RATE_DROP*`) o **NPC drop** (`PLAYER_RATE_DROP*`) según `RatesConfig`;
  - `Rnd < dropPercent` → suelta items del inventario.
- `calculateDeathExpPenalty(killer, atWar)`:
  - `ExperienceLossData.getPercentLost(level)`; mods `REDUCE_EXP_LOST_BY_RAID/MOB/PVP`;
  - `RATE_KARMA_EXP_LOST`; cap 10% del nivel; festival/war ÷4; AdventBlessing sin pérdida.
  - `setExpBeforeDeath(getExp())` → normalmente se restaura en resurrect.
- Limpieza cubics, channelization; DimensionalRift dead list.

**Resurrección** (solo conexión):
- `Player.reviveRequest(reviver, pet, power)` → `ConfirmDlg` → `reviveAnswer(answer)` → `doRevive(power)`.
- `Player.doRevive()` (L11125): `DecayTaskManager.cancel(this)`, `updateEffectIcons`, restaurar summon en peace, `startFeed` si monta, manejo instance rift.
- Base `Creature.doRevive()` (L2846): `setDead(false)`, restaurar CP/HP/MP según `PlayerConfig.RESPAWN_RESTORE_*`, `broadcastPacket(new Revive(this))`, `ZoneManager.onRevive`.

---

## 4. Muerte/despawn de `Npc` y `Monster`

### `Npc.deleteMe()` — (L1412)
```
onDecay()                       // decae el cuerpo
abort channelization
ZoneManager.getRegion(this).removeFromZones(this)
super.deleteMe()
```

### `Monster.deleteMe()` — override
- Con minions: `_master.getMinionList().onMasterDie(true)` (si es minion, `onMinionDie`).

### Respawn del NPC
- `Creature.doDie` → para Attackable, si `Spawn._doRespawn`:
  - `RespawnTaskManager.add(npc, time)`.
- `RespawnTaskManager.run()` → `spawn.respawnNpc(npc)` (reinicializa).
- `DecayTaskManager` agenda la desaparición del cuerpo (y `cancel` en revive).

---

## 5. TRAZABILIDAD

| Paso | Path |
|------|------|
| `CreatureStatus.reduceHp` detecta muerte | `entity/actor/status/CreatureStatus.java` |
| `Creature.doDie` | `entity/actor/Creature.java` |
| `Creature.setDead/isDead`, `isMortal` | `entity/actor/Creature.java` |
| `Player.doDie` override | `entity/actor/Player.java` |
| `onDieDropItem` / `calculateDeathExpPenalty` | `entity/actor/Player.java` |
| `Attackable.calculateRewards` | `entity/actor/Attackable.java` |
| `Npc.deleteMe` / `Monster.deleteMe` | `entity/actor/Npc.java` / `instance/Monster.java` |
| `RespawnTaskManager` | `taskmanagers/RespawnTaskManager.java` |
| `DecayTaskManager` | `taskmanagers/DecayTaskManager.java` |
| Eventos | `mechanics/events/holders/actor/creature/OnCreatureDeath.java`, `OnPlayerPvPKill.java` |

---

## 6. UNKNOWN / REQUIRES CODE VERIFICATION

- Posición exacta de respawn de Player tras muerte (dónde se teletransporta) → `REQUIRES CODE VERIFICATION`.
- Schema DB de `expBeforeDeath` y persistencia del drop → `REQUIRES` (fase DATABASE).
- Flujo completo de `reviveRequest`/`ConfirmDlg` (diálogo red) → `REQUIRES` si se detalla (no profundizado aquí).

---
**Status**: VERIFIED (flujo de muerte)  
**Verified**: 2026-08-23