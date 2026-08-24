# ENTITY TYPES INDEX

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Source of Truth**: `java/org/l2jmobius/gameserver/entity/` (relativa a la raíz del servidor)  
**Verified**: 2026-08-23  
**Status**: VERIFIED (reconstruido en Fase 2A contra el código)

---

## CORRECCIÓN RESPECTO A FASE 1

La jerarquía publicada en Fase 1 NO coincide con el código real en varios puntos clave. Esto fue verificado leyendo las declaraciones `extends`/`implements` de cada clase.

1. **`Monster` NO es subclase directa de `Npc`.** El orden real es `Npc -> Attackable -> Monster`.
2. **`Attackable` es subclase de `Npc`** (no al revés).
3. **`Door`, `StaticObject`, `Decoy`, `Vehicle` extienden `Creature`** (no `WorldObject`).
4. **`Fence` sí extiende `WorldObject`** directamente.
5. **`Castle` no vive en un subpaquete de `entity`**; las residencias están en `entity/residences/` (`AbstractResidence`, `ClanHall`, `AuctionableHall`, etc.).
6. La jerarquía real de contenedores está en `entity/itemcontainer/` (`ItemContainer` no extiende `WorldObject`).

---

## JERARQUÍA DE ENTIDADES REAL (VERIFICADA)

Basado en líneas `extends`/`implements` reales de cada archivo:

```
WorldObject (abstract) extends ListenersContainer implements IPositionable
├── Creature (abstract) extends WorldObject
│   ├── Playable (abstract) extends Creature          // Javadoc: Player | Summon
│   │   ├── Player extends Playable
│   │   └── Summon (abstract) extends Playable
│   │       ├── Pet extends Summon
│   │       └── Servitor extends Summon implements Runnable
│   ├── Npc extends Creature
│   │   ├── Attackable extends Npc
│   │   │   ├── Monster extends Attackable
│   │   │   │   ├── RaidBoss extends Monster
│   │   │   │   ├── GrandBoss extends Monster          // NOTA: NO extiende RaidBoss
│   │   │   │   ├── EventMonster extends Monster
│   │   │   │   ├── RiftInvader extends Monster
│   │   │   │   ├── FeedableBeast extends Monster (TamedBeast extends FeedableBeast)
│   │   │   │   ├── Chest extends Monster
│   │   │   │   ├── Block extends Monster
│   │   │   │   └── FestivalMonster extends Monster
│   │   │   ├── Guard extends Attackable (instance/Guard.java y derivados)
│   │   │   └── Defender extends Attackable (FortCommander extends Defender)
│   │   ├── Folk extends Npc
│   │   │   ├── Merchant extends Folk  (Teleporter extends Merchant)
│   │   │   ├── Trainer extends Folk
│   │   │   ├── Warehouse extends Folk
│   │   │   └── UCTower extends Folk
│   │   ├── Tower (abstract) extends Npc  // ControlTower, FlameTower (instance)
│   │   ├── Trap extends Npc
│   │   ├── Artefact extends Npc
│   │   ├── TerrainObject extends Npc
│   │   ├── FlyTerrainObject extends Npc
│   │   └── EffectPoint extends Npc
│   ├── Decoy extends Creature
│   ├── Door extends Creature
│   ├── StaticObject extends Creature
│   └── Vehicle (abstract) extends Creature
│       ├── Boat extends Vehicle
│       └── AirShip extends Vehicle (ControllableAirShip extends AirShip)
├── Item extends WorldObject        // entity/item/instance/Item.java
└── Fence extends WorldObject
```

### NOTA SOBRE LOS TIPOS (InstanceType)

El enum `InstanceType` (`entity/actor/enums/creature/InstanceType.java`) define OTRA jerarquía (jerarquía de tipos para despacho), cuyo árbol no siempre coincide con el `extends` de las clases:
- En `InstanceType`: `GrandBoss(RaidBoss)`, `Trap(Npc)`, `Artefact(Folk)`, `ControlTower(Npc)`, `FlameTower(Npc)`, `SiegeFlag(Npc)`.
- En las clases: `GrandBoss extends Monster`, `Artefact extends Npc`, `ControlTower/FlameTower extends Tower` (Tower extends Npc).

Ver [ENTITY_SYSTEM.md](../SYSTEMS/ENTITY_SYSTEM.md) para el detalle completo del enum.
## CLASES PRINCIPALES

### WorldObject
- **Package**: `org.l2jmobius.gameserver.entity`
- **Path**: `entity/WorldObject.java`
- **Extends**: `ListenersContainer`
- **Implements**: `IPositionable`
- **Responsibility**: clase base de TODOS los objetos interactivos del mundo (id, posición, región, spawn, visibilidad básica).

### Creature
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Creature.java`
- **Extends**: `WorldObject`
- **Implements**: (ninguno declarado en la cabecera)
- **Responsibility**: entidades vivas con stats/status, AI, skills, efectos, combate.

### Playable
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Playable.java`
- **Extends**: `Creature` (abstract)
- **Responsibility**: jugadores y summons (Player | Summon según su Javadoc).

### Player
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Player.java`
- **Extends**: `Playable`
- **Implements**: (ninguno declarado en la cabecera)
- **Responsibility**: avatar de un usuario humano; inventario, clan, party, summon, red, persistencia.

### Npc
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Npc.java`
- **Extends**: `Creature`
- **Responsibility**: personaje no jugador. Base de todos los NPC incl. hostiles.

### Attackable
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Attackable.java`
- **Extends**: `Npc`
- **Responsibility**: NPC atacable; gestión de aggro list y combate básico (monstruos, guardias).

### Monster
- **Package**: `org.l2jmobius.gameserver.entity.actor.instance`
- **Path**: `entity/actor/instance/Monster.java`
- **Extends**: `Attackable`
- **Responsibility**: monstruo hostil; sistema de minions (master/minion), aggro, spawn.

### Summon
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Summon.java`
- **Extends**: `Playable` (abstract)
- **Responsibility**: entidad controlada por el jugador (pet/servitor).

### Pet
- **Package**: `org.l2jmobius.gameserver.entity.actor.instance`
- **Path**: `entity/actor/instance/Pet.java`
- **Extends**: `Summon`
- **Responsibility**: mascota; requiere item de control; alimentación, nivel.

### Servitor
- **Package**: `org.l2jmobius.gameserver.entity.actor.instance`
- **Path**: `entity/actor/instance/Servitor.java`
- **Extends**: `Summon`
- **Implements**: `Runnable`
- **Responsibility**: invocación por skill; vida limitada, consumo de item.

### Tower
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Tower.java`
- **Extends**: `Npc` (abstract)
- **Responsibility**: torres de asedio; superclase de `ControlTower` y `FlameTower`.

### Vehicle
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Vehicle.java`
- **Extends**: `Creature` (abstract)
- **Responsibility**: vehículos con pasajeros; superclase de `Boat` y `AirShip`.

### Item
- **Package**: `org.l2jmobius.gameserver.entity.item.instance`
- **Path**: `entity/item/instance/Item.java`
- **Extends**: `WorldObject`
- **Responsibility**: instancia de item (objeto en mundo o en contenedor).

### Fence
- **Package**: `org.l2jmobius.gameserver.entity.actor.instance`
- **Path**: `entity/actor/instance/Fence.java`
- **Extends**: `WorldObject`
- **Responsibility**: valla decorativa/obstáculo (sin Creature).

---

## OTRAS ENTIDADES IMPORTANTES VERIFICADAS

| Clase | Path (entity/) | Extends |
|-------|----------------|---------|
| Decoy | actor/instance/Decoy.java | Creature |
| Door | actor/instance/Door.java | Creature |
| StaticObject | actor/instance/StaticObject.java | Creature |
| Trap | actor/instance/Trap.java | Npc |
| Artefact | actor/instance/Artefact.java | Npc |
| TerrainObject | actor/instance/TerrainObject.java | Npc |
| FlyTerrainObject | actor/instance/FlyTerrainObject.java | Npc |
| EffectPoint | actor/instance/EffectPoint.java | Npc |
| ControlTower | actor/instance/ControlTower.java | Tower |
| FlameTower | actor/instance/FlameTower.java | Tower |
| Boat | actor/instance/Boat.java | Vehicle |
| AirShip | actor/instance/AirShip.java | Vehicle |
| ControllableAirShip | actor/instance/ControllableAirShip.java | AirShip |

---

## CONTENEDORES (no son WorldObject)

```
ItemContainer (abstract)          // entity/itemcontainer/ItemContainer.java
├── Inventory (abstract) -> PlayerInventory, PetInventory
├── Warehouse (abstract) -> PlayerWarehouse, ClanWarehouse
├── PlayerFreight
├── Mail
└── PlayerRefund
```

Ver [INVENTORY_SYSTEM.md](../SYSTEMS/INVENTORY_SYSTEM.md).

---

## SPAWNING Y REGISTRO EN EL MUNDO

- `entity/spawns/Spawn.java` — `public class Spawn extends Location` (spawn, no entidad).
- `entity/spawns/` — `AutoSpawnHandler`, `SpawnGroup`, `SpawnGroupEntry`, `SpawnSelection`.
- `data/xml/SpawnData.java` — carga de spawns desde XML.
- `taskmanagers/RespawnTaskManager.java` — respawn.
- Registro/visibilidad: `World.addObject/removeObject/addVisibleObject/removeVisibleObject/switchRegion`.

Ver [WORLD_SYSTEM.md](../SYSTEMS/WORLD_SYSTEM.md) y SPAWN_SYSTEM.md (planificado).

---

## ZONAS (resumen)

`entity/zone/ZoneType.java` (abstract) y `entity/zone/type/*` (20+ tipos). Las zonas NO son `WorldObject`; son áreas con reglas. Se documentan en una fase posterior (ZONE_SYSTEM).

---

## ESTADÍSTICAS

- Archivos `.java` bajo `entity/`: **356** (verificado).
- Subclases concretas de actor en `entity/actor/instance/`: **72** (verificado).
- Managers reales: **53** (ver [MANAGERS_INDEX.md](MANAGERS_INDEX.md)).
- `World` **no** es un manager singleton (`entity/World.java`, utilidad estática).

---

## ENLACES

- [ENTITY_SYSTEM.md](../SYSTEMS/ENTITY_SYSTEM.md)
- [WORLD_SYSTEM.md](../SYSTEMS/WORLD_SYSTEM.md)
- [PLAYER_SYSTEM.md](../SYSTEMS/PLAYER_SYSTEM.md)
- [NPC_SYSTEM.md](../SYSTEMS/NPC_SYSTEM.md)
- [MONSTER_SYSTEM.md](../SYSTEMS/MONSTER_SYSTEM.md)
- [SUMMON_SYSTEM.md](../SYSTEMS/SUMMON_SYSTEM.md)
- [ITEM_SYSTEM.md](../SYSTEMS/ITEM_SYSTEM.md)
- [INVENTORY_SYSTEM.md](../SYSTEMS/INVENTORY_SYSTEM.md)

---

**Source of Truth**: declaraciones de clase en el código fuente real  
**Status**: VERIFIED  
**Verified**: 2026-08-23