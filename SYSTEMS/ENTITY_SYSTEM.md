# ENTITY SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Sistema**: Modelo de entidades en el Game Server  
**Source of Truth**: `java/org/l2jmobius/gameserver/entity/` (relativa a la raíz del servidor)  
**Verified**: 2026-08-23  
**Status**: VERIFIED (Fase 2A)

> NOTA: Este documento reconstruye la jerarquía leyendo las declaraciones `extends`/`implements` reales. NO es una copia de la KB de Fase 1 ni del conocimiento genérico de L2J.

---

## 1. VISIÓN GENERAL

Todas las entidades interactivas del mundo viven en `entity/` y descienden de `WorldObject`. El contenedor de items (`ItemContainer`) es una excepción notable: **no** extiende `WorldObject`; es un contenedor de instancias de `Item`.

Paquete base: `org.l2jmobius.gameserver.entity`

---

## 2. ÁRBOL DE HERENCIA REAL (VERIFICADO)

```
WorldObject (abstract) extends ListenersContainer implements IPositionable
├── Creature (abstract)
│   ├── Playable (abstract)                       // Javadoc: Player | Summon
│   │   ├── Player
│   │   └── Summon (abstract)
│   │       ├── Pet
│   │       └── Servitor implements Runnable
│   ├── Npc
│   │   ├── Attackable
│   │   │   ├── Monster
│   │   │   │   ├── RaidBoss
│   │   │   │   ├── GrandBoss
│   │   │   │   ├── EventMonster
│   │   │   │   ├── RiftInvader
│   │   │   │   ├── FeedableBeast ──► TamedBeast
│   │   │   │   ├── Chest
│   │   │   │   ├── Block
│   │   │   │   └── FestivalMonster
│   │   │   ├── Guard        (QuestGuard extends Guard)
│   │   │   └── Defender     (FortCommander extends Defender)
│   │   ├── Folk
│   │   │   ├── Merchant (Teleporter extends Merchant)
│   │   │   ├── Trainer
│   │   │   ├── Warehouse
│   │   │   └── UCTower
│   │   ├── Tower (abstract) → ControlTower, FlameTower
│   │   ├── Trap
│   │   ├── Artefact
│   │   ├── TerrainObject
│   │   ├── FlyTerrainObject
│   │   └── EffectPoint
│   ├── Decoy
│   ├── Door
│   ├── StaticObject
│   └── Vehicle (abstract)
│       ├── Boat
│       └── AirShip → ControllableAirShip
├── Item                            // entity/item/instance/Item.java
└── Fence                           // entity/actor/instance/Fence.java
```

### DIFERENCIAS clave respecto a la KB de Fase 1

| Afirmación Fase 1 | Real (código) |
|---|---|
| `Creature -> Attackable -> Npc` | `Creature -> Npc -> Attackable` |
| `Monster -> Npc` (directo) | `Monster -> Attackable -> Npc` |
| `Door/Fence/StaticObject/Vehicle -> WorldObject` | `Door/StaticObject/Vehicle -> Creature`; solo `Fence -> WorldObject` |
| `GrandBoss -> RaidBoss` | `GrandBoss extends Monster` (NO RaidBoss) |
| `Castle` en `entity` con manager | `entity/residences/*` se documenta aparte |

---

## 3. InstanceType (enum de tipos)

**Path**: `entity/actor/enums/creature/InstanceType.java`  
**Package**: `org.l2jmobius.gameserver.entity.actor.enums.creature`

Definición: `public enum InstanceType`.

- Cada entidad registra su `InstanceType` al construirse (`setInstanceType(...)`) en `WorldObject`.
- Mantiene enlaces padre y calcula máscaras de bits de 128 bits (`_typeL`, `_typeH`, `_maskL`, `_maskH`) para despacho veloz.
- Métodos públicos: `getParent()`, `isType(it)`, `isTypes(it...)`.
- Árbol del enum (ejemplos) — no siempre coincide con `extends`:
  - `WorldObject→Item`, `WorldObject→Creature`, `Creature→Npc`, `Creature→Playable`, `Playable→Player|Summon`, `Summon→Pet|Servitor`, `Npc→Folk→Merchant|Warehouse|Trainer`, `Npc→Attackable→Monster→RaidBoss→GrandBoss`, etc.
  - `Artefact(Folk)`, `Trap(Npc)`, `ControlTower/FlameTower(Npc)`, `SiegeFlag(Npc)` (en el enum), mientras que en las clases `Artefact extends Npc` y `Tower extends Npc`.

### `Entity` enum (entity/actor/enums/creature/Race.java) — pendiente de detalle

Todas las entidades tienen `_entityType`/`_race` (Race) y `_team` (Team), verificados en `Creature`.

--- 
## 4. CLASES DEL NÚCLEO

Formato por clase: `Class`, `Package`, `Path`, `Extends`, `Implements`, `Responsibility`, campos y métodos importantes, sistemas relacionados.

---

### WorldObject — raíz de todas las entidades

- **Class**: `WorldObject`
- **Package**: `org.l2jmobius.gameserver.entity`
- **Path**: `entity/WorldObject.java`
- **Extends**: `ListenersContainer`
- **Implements**: `IPositionable`
- **Responsibility**: base de todos los objetos interactivos. Dirección por `objectId`, posición, región de mundo, estado de spawn e invisibilidad.

Campos importantes:
- `_name`, `_objectId` (int, único), `_worldRegion` (`WorldRegion`), `_location` (`Location`), `_instanceType` (`InstanceType`), `_isSpawned` (bool), `_isInvisible` (bool), `_scripts` (Map).

Métodos importantes:
- `spawnMe()` / `spawnMe(x,y,z)` → `setSpawned(true)` → llama `onSpawn()`.
- `decayMe()` → `_isSpawned = false`.
- `onSpawn()` (abstract hook).
- `sendInfo(Player player)` (abstract): envía info de la entidad a un jugador.
- `isVisibleFor(Player player)`, `setXYZ` (via ipositionable), `getInstanceType()/isInstanceTypes(...)`.

Sistemas relacionados: World (registro), handlers de acción (`ActionClickHandler`, etc.), `IdManager` (objectId).

---

### Creature — criatura viva

- **Class**: `Creature`
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Creature.java`
- **Extends**: `WorldObject`
- **Implements**: —
- **Responsibility**: entidad viva: stats/status, efectos, skills, AI, combate.

Campos importantes (verificados):
- `_stat` (`CreatureStat`), `_status` (`CreatureStatus`), `_template` (`CreatureTemplate`).
- `_skills: Map<Integer,Skill>`, `_reuseTimeStampsSkills`, `_disabledSkills`, `_triggerSkills`.
- `_effectList`, `_buffFinishTask`.
- `_ai` (`CreatureAI`), `_intention`, `_move` (`MoveData`), `_target` (`WorldObject`).
- `_team` (`Team`), `_karma`, `_knownRelations`, `_seenCreatures`.
- flags: `_isDead`, `_isImmobilized`, `_isParalyzed`, `_isRunning`, `_isTeleporting`, `_isInvul`, `_isMortal`, `_isFlying`, `_lethalable`.

Métodos importantes:
- `getAI()`, `getStat()`, `getStatus()`, `getTemplate()`.
- `setTarget()`, `stopMove()`, `stopAllEffects()`, `addAggro/clearAggro`, `isAttackable`, `isAliveDead...`.
- `sendInfo()` (abstract inherited), `onSpawn/decayMe` overridden.

Relacionado: AI system (`gameserver/ai/*`), skill system (`mechanics/skill`), effects.

---

### Playable — jugable (Player | Summon)

- **Class**: `Playable`
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Playable.java`
- **Extends**: `Creature` (abstract)
- **Responsibility**: capa común a `Player` y `Summon` (controlado por jugador). Usa `PlayableStat`, `PlayableStatus`, `QuestState`, `EventDispatcher`.

---

### Player — avatar del usuario

- **Class**: `Player`
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Player.java`
- **Extends**: `Playable`
- **Responsibility**: personaje de jugador humano; inventario, clan, party, summon, network, persistencia.

Campos importantes (verificados, parcial):
- Conexión/persistencia: `_client` (`GameClient`), `_accountName`, `_ip`, `_isOnline`, `_offlinePlay`, `_enteredWorld`, `_createDate`, `_uptime`.
- Inventario/oreja: `_inventory` (`PlayerInventory`), `_freight` (`PlayerFreight`), `_adena`.
- Grupos/social: `_clan`, `_party`, `_friendList`, `_contactList`, `_married`, `_coupleId`, `_partnerId`.
- Comunidades: `_subClasses`, `_classIndex`, `_baseClass`, `_activeClass`.
- Vehículos/monta: `_vehicle`, `_mountType`, `_mountNpcId`, `_mountLevel`.
- Cubics/transformation: `_cubics`, `_transformation`, `_transformSkills`.
- Siege/Olympiad/Duel: `_siegeState`, `_siegeSide`, `_inOlympiadMode`, `_olympiadGameId`, `_isInDuel`, `_duelState`, `_duelId`.
- Objetos: `_bookmarkSlot`, `_tpbookmarks`, `_premiumItems`, `_dwarven/book`.
- Otros: `_pvpKills`, `_pkKills`, `_pvpFlag`, `_fame`, `_curWeightPenalty`, `_cursedWeaponEquippedId`, `_fish`, `_lure`, `_autoPlay...`.

Métodos importantes (verificados por `Select-String`):
- `getInventory(): PlayerInventory`
- `getClan(): Clan`
- `getParty(): Party`
- `getSummon(): Summon`
- `getClient(): GameClient`
- `getActiveWeaponInstance()/getSecondaryWeaponInstance(): Item`
- `doRevive()` / `doRevive(double power)`

Sistemas relacionados: World, Clan (entity/clan), Party (entity/groups), QuestState (mechanics/script), skills (Creature), network (`GameClient`), database (CharInfoTable / ItemContainer.restore).

---
### Npc — personaje no jugador

- **Class**: `Npc`
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Npc.java`
- **Extends**: `Creature`
- **Responsibility**: base de todos los NPC (no hostiles y hostiles). Template-driven: usa `NpcTemplate`. Maneja diálogos HTML (`HtmCache`), estados de ocupación, manos/armas equipadas.

Campos/constantes importantes (verificados):
- `INTERACTION_DISTANCE = 250`, `RANDOM_ITEM_DROP_LIMIT = 70`.
- `_spawn` (`Spawn`), `_isBusy`, `_busyMessage`, `_isDecayed`.
- `_castleIndex`, `_fortIndex`, `_isInTown`.
- `_isAutoAttackable`, `_isTalkable`, `_isQuestMonster`, `_isFakePlayer`.
- `_isRandomAnimationEnabled`, `_isRandomWalkingEnabled`, `_isWalker`.
- `_currentLHandId`, `_currentRHandId`, `_currentEnchant`, `_currentCollisionHeight/Radius`.
- `_soulshotamount`, `_spiritshotamount`, `_displayEffect`, `_shotsMask`, `_killingBlowWeaponId`, `_scriptValue`.
- `_summonedNpcs: Map<Integer,Npc>`, `_questTimers`, `_timerHolders`.

Métodos importantes (verificados):
- `onSpawn()` (overridden).
- `isAutoAttackable(attacker)` (más específico en subclases).

Sistemas relacionados: `data/xml/NpcData`, `entity/actor/templates/NpcTemplate`, `entity/spawns/Spawn`, `cache/HtmCache`, `ai/*`.

---

### Attackable — NPC atacable

- **Class**: `Attackable`
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Attackable.java`
- **Extends**: `Npc`
- **Responsibility**: NPC que puede ser atacado (monstruos, guardias). Maneja aggro list y estados de combate. Retrofit para guardias de fortaleza/sitio.

Métodos importantes (verificados por uso directo desde `WorldRegion.switchAI`):
- `setTarget(null)`, `stopMove(null)`, `stopAllEffects()`, `clearAggroList()`, `getAttackByList().clear()`.
- `isAutoAttackable(Creature attacker)` (ver también Monster override).

---

### Monster — monstruo hostil

- **Class**: `Monster`
- **Package**: `org.l2jmobius.gameserver.entity.actor.instance`
- **Path**: `entity/actor/instance/Monster.java`
- **Extends**: `Attackable`
- **Responsibility**: monstruo hostil; sistema de minions (un master puede tener minions; un minion señala a su master) y comportamiento agresivo.

Constructor:
- `Monster(NpcTemplate template)` → `super(template)`, `setInstanceType(InstanceType.Monster)`, `setAutoAttackable(true)`.

Campos importantes:
- `_enableMinions`, `_master` (`Monster`), `_minionList` (`MinionList`).

Métodos importantes:
- `isAutoAttackable(Creature attacker)` — override: no ataca a otros monstruos salvo fake players; respeta `NpcConfig`.
- `isAggressive()` — `getTemplate().isAggressive() && !isAffected(PASSIVE)`.
- `onSpawn()`, `onTeleported()`, `deleteMe()`, `getLeader()/setLeader()`, `hasMinions()/getMinionList()`, `enableMinions()`, `isMonster()`, `asMonster()`, `isWalker()`, `giveRaidCurse()`, `doCast(...)`.
- `setAutoAttackable(true)` (heredada de `Npc`).

Sistemas relacionados: spawn (via NpcTemplate/spawns), AI (`AttackableAI`), combate (effects/skill), drop (ItemManager / ItemsOnGroundManager), aggro (Attackable), respawn (`RespawnTaskManager`).
### Summon — invocado/alimentado

- **Class**: `Summon`
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Summon.java`
- **Extends**: `Playable` (abstract)
- **Responsibility**: criatura controlada por el jugador (pet o servitor). Vive bajo un owner (`Player`).

Métodos importantes (verificados):
- `getOwner(): Player` (también se usa `_owner`).
- `unSummon(Player owner)`.
- Usa `SummonAI` (en `ai/SummonAI.java`), `ExperienceData`, `ItemData`.

### Pet — mascota

- **Class**: `Pet`
- **Package**: `org.l2jmobius.gameserver.entity.actor.instance`
- **Path**: `entity/actor/instance/Pet.java`
- **Extends**: `Summon`
- **Responsibility**: mascota controlada por item (`_controlObjectId`); se puede montar (`_mountable`), alimentar, y respetar nivel de mascota.

Campos de interés: `_controlObjectId`, `_respawned`, `_mountable`, `_feedTask`, `_data` (`PetData`), `_leveldata` (`PetLevelData`), `_expBeforeDeath`, `_curWeightPenalty`.

### Servitor — servidor invocado

- **Class**: `Servitor`
- **Package**: `org.l2jmobius.gameserver.entity.actor.instance`
- **Path**: `entity/actor/instance/Servitor.java`
- **Extends**: `Summon`
- **Implements**: `Runnable`
- **Responsibility**: invocado por skill (con `_referenceSkill`), con duración limitada (lifetime) y consumo de items.

Campos: `_expMultiplier`, `_itemConsume`, `_lifeTime`, `_lifeTimeRemaining`, `_consumeItemInterval-Remaining`, `_summonLifeTask`, `_referenceSkill`.

### Tower — torre de asedio

- **Class**: `Tower`
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Tower.java`
- **Extends**: `Npc` (abstract)
- **Responsibility**: torre atacable solo durante asedio. Subclases `ControlTower`, `FlameTower`.
- Detalle: `canBeAttacked()` aprovecha `getCastle().getSiege()`, `isAutoAttackable` solo para atacantes.

### Vehicle

- **Class**: `Vehicle`
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Vehicle.java`
- **Extends**: `Creature` (abstract)
- **Responsibility**: vehículo (barcos/airships) con pasajeros. Configura viaje `VehiclePathPoint[]`.
- Detalles verificados: `_passengers` (Set<Player>), `_dockId`, `updatePosition()` mueve pasajeros, `deleteMe()` expulsa pasajeros, `teleToLocation` teletransporte.

### Item

- **Class**: `Item`
- **Package**: `org.l2jmobius.gameserver.entity.item.instance`
- **Path**: `entity/item/instance/Item.java`
- **Extends**: `WorldObject`
- **Responsibility**: instancia de un item (sobre el suelo o dentro de un contenedor). Template-driven por `ItemTemplate`.
- Campos verificados: `_owner`, `_dropperObjectId`, `_count`, `_initCount`, `_time`, `_decrease`, `_itemId`, `_itemTemplate`, `_loc` (`ItemLocation`), `_locData`, `_enchantLevel`, `_wear`, `_augmentation`, `_mana`, `_type1/_type2`, `_dropTime`, `_published`, `_protected`, `_existsInDb`, `_storedInDb`, `_elementals`, `_itemLootSchedule`, `_dropProtection`, `_shotsMask`, `_enchantOptions`.
- `restoreFromDb(...)` estático (visto en `ItemContainer.restore`).

### Otras entidades (de `entity/actor/instance`)

- `Decoy` (Creature), `Door` (Creature), `StaticObject` (Creature), `Trap` (Npc).
- `Fence` (WorldObject — única no-Creature además de Item).
- `Artefact` (Npc), `TerrainObject` (Npc), `FlyTerrainObject` (Npc), `EffectPoint` (Npc).
- `Boat` / `AirShip` / `ControllableAirShip` (Vehicle).
- `ControlTower` / `FlameTower` (Tower).
- `Guard` (Attackable, `QuestGuard extends Guard`), `Defender` (Attackable, `FortCommander extends Defender`).
- `Folk` (Npc) y subtipos: `Merchant` (Teleporter), `Trainer`, `Warehouse`, `UCTower`, Village–y–otros.
- **72 clases** concretas de actor en `instance/` (verificado).

---

## 5. TRAZABILIDAD DE FASE 1 → 2A

- **Incorrecto en FASE 1**: `Monster` bajo `Npc` directo → corregido (`Monster -> Attackable -> Npc`).
- **Incorrecto en FASE 1**: `Door`/`Fence`/`StaticObject` como hijos de `WorldObject` → corregido.
- **Incorrecto en FASE 1**: `World` como manager singleton → corregido en [WORLD_SYSTEM.md](WORLD_SYSTEM.md).
- **Incorrecto en FASE 1**: `Castle` en paquete de entity con manager → residencias en `entity/residences/`.

---

**Fuente**: `entity/**/*.java` (declaraciones reales)  
**Status**: VERIFIED (con secciones que requieren revisión puntual marcadas con REQUIRES CODE VERIFICATION)  
**Verified**: 2026-08-23