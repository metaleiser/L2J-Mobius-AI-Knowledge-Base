# INVENTORY SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Fase**: Contenedores de items  
**Source of Truth (Path)**: `entity/itemcontainer/*.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (jerarquía observada; lógica de peso/capacidad en fase posterior)

---

## 1. CLASE BASE `ItemContainer`

- **Class**: `ItemContainer`
- **Package**: `org.l2jmobius.gameserver.entity.itemcontainer`
- **Path**: `entity/itemcontainer/ItemContainer.java`
- **Extends**: — (no extiende nada en la cabecera)
- **Implements**: —

> IMPORTANTE (corrección FASE 1): `ItemContainer` NO extiende `WorldObject`. Es un contenedor de `Item`.

### Declaración
```java
public abstract class ItemContainer {
    protected final Set<Item> _items = ConcurrentHashMap.newKeySet(1);
    protected abstract Creature getOwner();
    protected abstract ItemLocation getBaseLocation();
    ...
}
```

### Métodos verificados
- `getOwner(): Creature` (abstract), `getOwnerId()`
- `getSize()`, `getItems()`, `getItemByItemId(int itemId)`
- `addItem(...)`, `removeItem(Item item)` (protected)
- `deleteItem(...)`, `validate(boolean)`, `validateWeight`, `validateCapacity`
- `refreshWeight()`
- `deleteMe()` — recorre items, `updateDatabase(true)`, `stopAllTasks()`, `World.removeObject(item)`.
- `updateDatabase()` — guarda vía `item.updateDatabase(true)`.
- `restore()` — recarga desde tabla `items` con `owner_id=? AND loc=?`, invoca `Item.restoreFromDb`, `World.addObject(item)`. Si stackable existente: `addItem(ItemProcessType.RESTORE, ...)`

---

## 2. JERARQUÍA REAL DE CONTENEDORES

Con línea `extends` leída de cada archivo:

```
ItemContainer (abstract)
├── Inventory (abstract)       // entity/itemcontainer/Inventory.java
│   ├── PlayerInventory        // entity/itemcontainer/PlayerInventory.java
│   └── PetInventory           // entity/itemcontainer/PetInventory.java
├── Warehouse (abstract)       // entity/itemcontainer/Warehouse.java
│   ├── PlayerWarehouse        // entity/itemcontainer/PlayerWarehouse.java
│   └── ClanWarehouse          // entity/itemcontainer/ClanWarehouse.java
├── PlayerFreight              // entity/itemcontainer/PlayerFreight.java
├── Mail                       // entity/itemcontainer/Mail.java
└── PlayerRefund               // entity/itemcontainer/PlayerRefund.java
```

---

## 3. CLASES DETALLE

### `Inventory` (abstract)
- **Path**: `entity/itemcontainer/Inventory.java`
- Campos: `_paperdoll[]` (Item), `_paperdollListeners`, `_totalWeight`, `_wearedMask`, `_inventory`, `_changed`.

### `PlayerInventory`
- **Path**: `entity/itemcontainer/PlayerInventory.java`
- **Extends**: `Inventory`
- Campos: `_owner` (`Player`), `_adena`, `_ancientAdena`, `_questItemSize`, `_blockItems`, `_blockMode`.
- Es el inventario real del jugador: `Player._inventory`.

### `PetInventory`
- **Path**: `entity/itemcontainer/PetInventory.java`
- **Extends**: `Inventory`
- Mascota lleva items propios.

### `Warehouse` (abstract) — almacén
- **Path**: `entity/itemcontainer/Warehouse.java`
- **Extends**: `ItemContainer`
- Subtipos `PlayerWarehouse`, `ClanWarehouse`.

### Otros
- `PlayerFreight` (extends ItemContainer) — freito pesado del jugador.
- `Mail` (extends ItemContainer) — correo con adjuntos.
- `PlayerRefund` (extends ItemContainer) — reembolsos de items.

---

## 4. RELACIÓN CON PLAYER

`Player` crea su inventario en el constructor:
```
private final PlayerInventory _inventory = new PlayerInventory(this);
private final PlayerFreight _freight = new PlayerFreight(this);
```

Accesible via `Player.getInventory()` (verificado).

---

## 5. TRAZABILIDAD

- La KB de FASE 1 no documentaba `itemcontainer/`; se añade aquí la jerarquía real de contenedores.

---

## 6. UNKNOWN / REQUIRES CODE VERIFICATION

- Reglas exactas de validación de peso/capacidad por contenedor (difieren por subtipo).
- Lógica de `Mail` (adjuntos, envío) en fase posterior.

---
**Status**: VERIFIED (jerarquía/lista)  
**Verified**: 2026-08-23