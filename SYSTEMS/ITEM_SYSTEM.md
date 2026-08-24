# ITEM SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Fase**: Items y plantillas  
**Source of Truth**: `entity/item/ItemTemplate.java`, `entity/item/instance/Item.java`, `entity/item/Weapon.java`, `entity/item/Armor.java`, `entity/item/EtcItem.java`, `entity/item/Henna.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (estructura Template/Instance; comportamiento avanzado en fase posterior)

---

## 1. CONCEPTO: TEMPLATE vs INSTANCE

Distinción central **verificada** en el código real:

- **`ItemTemplate`** (abstracta) — define datos estáticos de un item (id, nombre, icono, peso, stackable, tipo, precio, restricciones). Una plantilla representa UN modelo de item (ej. "Espada grande").
- **`Item`** (instancia) — es un objeto concreto en el mundo o en un contenedor (`_count`, `_enchantLevel`, `_loc`, `_owner`, ...). `Item extends WorldObject`.

Patrón: un `Item` se crea a partir de su `ItemTemplate` y se clona/instancia por cada unidad.

---

## 2. `Item` (instancia)

| Campo | Valor |
|-------|-------|
| **Class** | `Item` |
| **Package** | `org.l2jmobius.gameserver.entity.item.instance` |
| **Path** | `entity/item/instance/Item.java` |
| **Extends** | `WorldObject` |

Campos verificados (parcial):
- `_owner` (Creature), `_dropperObjectId`, `_itemId`, `_itemTemplate`.
- `_count`, `_initCount`, `_time`, `_decrease`.
- `_loc` (`ItemLocation`), `_locData`, `_enchantLevel`, `_wear`, `_augmentation`, `_mana`.
- `_type1`, `_type2`, `_dropTime`, `_published`, `_protected`, `_existsInDb`, `_storedInDb`.
- `_elementals`, `_itemLootSchedule`, `_dropProtection`, `_shotsMask`, `_enchantOptions`.

Métodos verificados: `getItemByItemId` (ItemContainer), `updateDatabase`, `destroyItem` (ItemManager/Item), `restoreFromDb` (estático).

---

## 3. SUBTEMPLATES

| Clase | Path | Extends |
|-------|------|---------|
| `Weapon` | entity/item/Weapon.java | ItemTemplate |
| `Armor` | entity/item/Armor.java | ItemTemplate |
| `EtcItem` | entity/item/EtcItem.java | ItemTemplate |
| `Henna` | entity/item/Henna.java | *(sin extends declarado)* |

> NOTA: `Henna` no declaró `extends` en su cabecera en la inspección. Se marca `REQUIRES CODE VERIFICATION` para confirmar si hereda de `ItemTemplate` o `EtcItem`.

---

## 4. ENUMERADORES Y SOPORTE (`entity/item/`)

- **Enums**: `ItemType`, `ArmorType`, `WeaponType`, `EtcItemType`, `MaterialType`, `CrystalType`, `BodyPart`, `ShotType`, `ActionType`, `ItemLocation`, `ItemProcessType`, `ItemGrade`, `ItemSkillType`, `ElementalItemType`.
- **Holders**: `ItemHolder`, `ItemInfo`, `ItemChanceHolder`, `ItemEnchantHolder`, `PremiumItem`, `UniqueItemHolder`, `PetItemHolder`, `RestorationItemHolder`, `ExtractableProduct`, `ExtractableProductItem`.
- **Enchant**: `AbstractEnchantItem`, `EnchantScroll`, `EnchantScrollGroup`, `EnchantItemGroup`, `EnchantRateItem`, `EnchantSupportItem`, `EnchantResultType`.
- **Recipe**: `RecipeList`, `RecipeItemInfo`, `ManufactureItem`.
- **Acción**: `ItemAuction` (subpaquete `auction/`).

---

## 4. GESTIÓN DE INSTANCIA

- `managers/ItemManager` — **no** es singleton; métodos estáticos `createItem(process, itemId, count, actor, reference)` y `destroyItem(...)`. Crea `Item` con `IdManager.getInstance().getNextId()` y notifica `OnItemCreate` vía `EventDispatcher`.
- `managers/ItemsOnGroundManager` — items en el suelo.
- Registro en el mundo: `Item extends WorldObject` → `World.addObject/removeObject`.

---

## 5. TEMPLATE vs INSTANCE — IMPORTANTE

NO se asumió comportamiento típico de L2J. La separación `ItemTemplate` (definición) vs `Item` (instancia) es la distinción real observable en el código, y el catálogo de la KB de FASE 1 no distinguía esto (trataba `Item` y subtipos como equivalentes).

---

## 6. UNKNOWN

- Exactas condiciones de `validateCapacityByItemId`/`drop-mechanics` → fase economia/combate.
- Herencia de `Henna` → REQUIRES CODE VERIFICATION.

---
**Fuente**: `entity/item/**/*.java` + `managers/ItemManager.java`  
**Status**: VERIFIED (sección templates/instancias); secciones marcadas pendientes  
**Verified**: 2026-08-23