# NPC SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Fase**: Personajes no jugador  
**Source of Truth**: `entity/actor/Npc.java`, `entity/actor/Attackable.java`, `entity/actor/templates/NpcTemplate.java`, `entity/actor/instance/*`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (herencia/templates/spawn/interacción; AI se profundiza en fase propia)

---

## 1. CLASE BASE `Npc`

- **Class**: `Npc`
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Npc.java`
- **Extends**: `Creature`
- **Implements**: —

Responsibility: base común de todos los NPC; template-driven via `NpcTemplate`, con diálogos HTML (`HtmCache`), estados de ocupación, y equipamiento visual (manos).

Constantes verificadas: `INTERACTION_DISTANCE = 250`, `RANDOM_ITEM_DROP_LIMIT = 70`.

Campos verificados: `_spawn` (`Spawn`), `_isBusy`, `_busyMessage`, `_isDecayed`, `_castleIndex`, `_fortIndex`, `_isInTown`, `_isAutoAttackable`, `_isTalkable`, `_isQuestMonster`, `_isFakePlayer`, `_isRandomAnimationEnabled`, `_isRandomWalkingEnabled`, `_isWalker`, `_currentLHandId`, `_currentRHandId`, `_currentEnchant`, `_currentCollisionHeight/Radius`, `_soulshotamount`, `_spiritshotamount`, `_displayEffect`, `_shotsMask`, `_killingBlowWeaponId`, `_scriptValue`, `_summonedNpcs`, `_questTimers`, `_timerHolders`.

---

## CLASE `Attackable`

- **Class**: `Attackable`
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Attackable.java`
- **Extends**: `Npc`

Responsibility: NPC que puede recibir ataques. Contiene lógica de aggro list y estados de combate; usado como base de monstruos y guardias.

Métodos verificados por uso: `setTarget()`, `stopMove()`, `stopAllEffects()`, `clearAggroList()`, `getAttackByList()`.

---

## TEMPLATES (heredan de `CreatureTemplate`)

| Clase | Package | Path | Extends |
|-------|---------|------|---------|
| `NpcTemplate` | org.l2jmobius.gameserver.entity.actor.templates | entity/actor/templates/NpcTemplate.java | CreatureTemplate |
| `PlayerTemplate` | ídem | entity/actor/templates/PlayerTemplate.java | CreatureTemplate |
| `CreatureTemplate` | ídem | entity/actor/templates/CreatureTemplate.java | (base) |
| `CubicTemplate` / `DoorTemplate` | ídem | templates/ | — |

El template es la “plantilla” que define comportamiento estático (aggro, talkable, questMonster, fakePlayer, drops) para N instancias.

---

## SPAWN

- `entity/spawns/Spawn.java` — `public class Spawn extends Location`, maneja spawn/respawn de grupos de NPC.
- `entity/spawns/SpawnGroup`, `SpawnGroupEntry`, `SpawnSelection`.
- `entity/spawns/AutoSpawnHandler` — auto-respawn.
- `data/xml/SpawnData.java` + `data/xml/NpcData.java` — cargan los datos desde XML.
- `taskmanagers/RespawnTaskManager` — reprograma respawns.

---

## INTERACCIÓN

- `INTERACTION_DISTANCE = 250` (constante de Npc).
- `onAction(Player player, boolean interact)`: toda NPC es invocada al hacer click; los handlers de acción (`handler/ActionClickHandler`) despachan por `InstanceType`.
- Diálogos: `cache/HtmCache` (NPC HTML) y prefax del jugador.

---

## AI (sólo mapa de clases, sin profundizar)

Paquete `gameserver/ai/`:
`AbstractAI`, `CreatureAI`, `PlayableAI`, `PlayerAI`, `SummonAI`, `AttackableAI`, `AirShipAI`, `BoatAI`, `DoorAI`, `SiegeGuardAI`, `FortSiegeGuardAI`, `SpecialSiegeGuardAI`, `DistrustAI`, `Action`, `Intention`, `NextAction`.

(Detalle del bucle de AI en fase AI_SYSTEM.)

---

## MANIPULACIÓN REAL DE SUBTIPOS (`entity/actor/instance`)

Hay **72** clases concretas en `entity/actor/instance/` (verificado). Algunas relaciones documentadas en Fase 2A:

| Clase | extends |
|-------|---------|
| Monster | Attackable |
| RaidBoss / GrandBoss / EventMonster / RiftInvader / FestivalMonster | Monster |
| FeedableBeast -> TamedBeast | Monster -> (TamedBeast extends FeedableBeast) |
| Folk | Npc |
| Merchant / Trainer / Warehouse / UCTower | Folk |
| Teleporter | Merchant |
| Trap / Artefact / TerrainObject / FlyTerrainObject / EffectPoint | Npc |
| ControlTower / FlameTower | Tower |

Y otras clases concretas (lista parcial): `Adventurer`, `Auctioneer`, `BroadcastingTower`, `ClanHallDoorman/Manager`, `Doorman`, `DungeonGatekeeper`, `Guard`, `QuestGuard`, `FestivalGuide`, `FestivalMonster`, `Fisherman`, `FortCommander`, `FortDoorman`, `FortLogistics`, `FortManager`, `FriendlyMob`, `KrateiCubeManager`, `KrateisMatchManager`, `OlympiadManager`, `PetManager`, `RaceManager`, `SchemeBuffer`, `SignsPriest` (+ `Dawn/DuskPriest`), `StaticObject`, `TamedBeast`, `TerritoryWard`, `Village...`, `Warehouse`, etc.

---

## TRAZABILIDAD

La KB de FASE 1 clasificaba `Monster` como subtipo directo de `Npc`. Corregido: `Monster -> Attackable -> Npc`.

---
**Status**: VERIFIED (estructura/herencia/spawn)  
**Verified**: 2026-08-23