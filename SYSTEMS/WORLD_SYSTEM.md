# WORLD SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Fase**: Modelo del mundo  
**Source of Truth**: `entity/World.java`, `entity/WorldRegion.java`, `entity/WorldObject.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (construido en Fase 2A)

---

## 1. ¿QUÉ ES `World`?

`World` es una **clase de utilidad estática**, NO un manager singleton.

- **Path**: `java/org/l2jmobius/gameserver/entity/World.java`
- **Package**: `org.l2jmobius.gameserver.entity`
- Declaración: `public class World` (constructor `private World()` para evitar instancias).
- **NO** define `public static World getInstance()`. Todos sus métodos son `public static`.

Esto corrige la afirmación de FASE 1 que lo listaba como manager singleton de 52.

---

## 2. ESTADO EN MEMORIA (ConcurrentHashMap)

| Campo | Tipo | Propósito |
|-------|------|-----------|
| `_allPlayers` | `Map<Integer,Player>` | jugadores online (id→Player) |
| `_allGoodPlayers` | `Map<Integer,Player>` | jugadores "Good" (si FactionSystem) |
| `_allEvilPlayers` | `Map<Integer,Player>` | jugadores "Evil" (si FactionSystem) |
| `_allObjects` | `Map<Integer,WorldObject>` | todos los objetos visibles (id→object) |
| `_petsInstance` | `Map<Integer,Pet>` | mascotas por `ownerId` |
| `_worldRegions` | `WorldRegion[][][]` | grid de regiones 3D |

---

## 3. GRID DE REGIONES (`WorldRegion`)

`WorldRegion` vive en `entity/WorldRegion.java` (clase concreta, no singleton).

- Representa una celda de región en coordenadas de mundo.
- `_visibleObjects` (Set), `_doors` (Set), `_fences` (Set).
- `_surroundingRegions[]`: vecinas para consultas de visibilidad.
- `_active` / activación de vecinos (si `GRIDS_ALWAYS_ON` es false, se activa/desactiva según jugadores).

**Cálculo de regiones** (en `World.java`):
- `TILE_SIZE = 32768`
- `SHIFT_BY = 11`
- Límites X: tiles 11–26 (con cero en `TILE_ZERO_COORD_X=20`); Y: 10–26.
- Z: desde `-16000` a `16000`, `Z_REGION_SIZE = 2000`.
- `init()` instancia todas las `WorldRegion` y llama a `setSurroundingRegions()`.

---

## 4. REGISTRO Y ELIMINACIÓN DE OBJETOS

### `World.addObject(WorldObject object)`
- `_allObjects.putIfAbsent(object.getObjectId(), object)`.
- Si es `Player`: `_allPlayers.putIfAbsent(...)`, y si está en teleport termina. Detecta `existingPlayer` duplicado → desconecta ambos (uso de `Disconnection`, `LeaveWorld`).
- Si `FactionSystemConfig` activo, actualiza `_allGoodPlayers`/`_allEvilPlayers`.

### `World.removeObject(WorldObject object)`
- `_allObjects.remove(object.getObjectId())`.
- Si es `Player` y no está teleportando, `_allPlayers.remove(...)` y maneja facción.

### `World.addVisibleObject(object, WorldRegion newRegion)`
- Si `!newRegion.isActive()` retorna sin actuar.
- Recorre objetos visibles del objeto y envía `sendInfo(player)` a los que le aplique. (Vista parcial; el método se documenta en detalle si se decide.)

### `World.removeVisibleObject(WorldObject object, WorldRegion oldRegion)`
- `oldRegion.removeVisibleObject(object)` y recorre las vecinas para notificar desaparición.

### `World.switchRegion(WorldObject object, WorldRegion newRegion)`
- Si `oldRegion == newRegion`, no hace nada.
- Recorre regiones viejas no limítrofes con la nueva y actualiza visibilidad (desaparición/aparición).

### `World.disposeOutOfBoundsObject(WorldObject object)`
- Se invoca cuando `getRegion()` obtiene `ArrayIndexOutOfBoundsException` (objeto fuera del grid).

---
## 5. VISIBILIDAD Y BÚSQUEDA

Todos los métodos de búsqueda están sobre `World` usando regiones cercanas:

- `getVisibleObjects()` / `getVisibleObjectsCount()` — vista global (`_allObjects`).
- `getVisibleObjects(object, Class<T>)` / `InRange(object, clazz, range)` — objetos visibles a otro objeto dentro de sus regiones vecinas.
- Variantes con `Predicate<T>`: filtrado.
- `getFirstVisibleObject...(...)`, `getNearestVisibleObject(...)`, `getRandomVisibleObject(...)` y variantes `forEach`/`forFirst`/`forNearest`/`forRandom` con `Consumer` o `Predicate+Consumer`.
- `findObject(int objectId)` — lookup exacto por id.
- `getRegion(WorldObject)` / `getRegion(int,int,int)` — región actual (lanza manejo vía `disposeOutOfBoundsObject` si queda fuera del grid).

### Override de visibilidad en `WorldObject`

- `isVisibleFor(Player player)` y `sendInfo(Player player)`: permiten a cada entidad decidir si se muestra y qué datos envía. `World.addVisibleObject` llama a `sendInfo` para los objetos que deben tornarse visibles.

---

## 6. JUGADORES Y PETS

- `getPlayers()`, `getPlayer(String name)`, `getPlayer(int objectId)` — consultas de jugadores.
- `getPet(int ownerId)`, `addPet(int ownerId, Pet pet)`, `removePet(int ownerId)` — registro de silicon animals por id de dueño (`_petsInstance`).
- `getAllGoodPlayers()` / `getAllEvilPlayers()` — si `FactionSystemConfig` soporte habilitado.

---

## 7. BROADCAST

- `broadcastToVisiblePlayers(WorldObject object, ServerPacket packet)` — avisa a jugadores que ven `object`.
- `broadcastToSelfAndVisiblePlayers(...)` — incluye al propio jugador (si es playable).
- Variantes `...InRange(object, packet, range)`.
- `broadcastToAllOnlinePlayers(ServerPacket packet)` y `broadcastToAllOnlinePlayers(String text[, isCritical])` — mensajes globales.
- `broadcastToAllOnlinePlayersOnScreen(String text)` — mensaje en pantalla.

---

## 8. SPAWN / DESPAWN (flujo real)

El flujo fundamentado en código es:

1. **Creación**: un `Npc`/`Monster` con `NpcTemplate` (y un `Spawn` de `entity/spawns/Spawn.java`).
2. **Registro**: `World.addObject(object)` → `_allObjects`.
3. **Región**: el objeto entra en `WorldRegion` vía `World.addVisibleObject(object, region)` y queda en `_visibleObjects`.
4. **Visibilidad**: los jugadores de regiones vecinas que cumplen `isVisibleFor` reciben `sendInfo`.
5. **Interacción**: acciones (ataque, conversación) usan handlers y los getVisibleObjects* / broadcast.
6. **Eliminación**: `World.removeObject` + `World.removeVisibleObject(object, oldRegion)`; `decayMe()` (~`_isSpawned=false`); respawn via `RespawnTaskManager`.

> No se documenta el cuerpo completo de `spawnButton`/`activate` para no duplicar código; se acepta `REQUIRES CODE VERIFICATION` para el orden exacto de `onSpawn` en cada subclase concreta.

---

## 9. RELACIÓN CON INSTANCIAS

- Existe `entity/instancezone/` (`Instance`, `InstanceWorld`, `InstanceReenterType`, `InstanceRemoveBuffType`).
- `World` principal es un singleton de proceso; las instancias usan `InstanceWorld` como mundo separado.
- `InstanceManager` (managers/) coordina la creación de instancias.
- Detalle completo: fase de INSTANCE_SYSTEM (planificado).

---
## 10. NOTA DE TRAZABILIDAD

Inconsistencias detectadas en FASE 1 y corregidas:
- `World` NO es un manager singleton → se documenta como utilidad estática.
- Método real de registro: `addObject(object)` (y `addVisibleObject` con región), no "void entity)".

---
**Fuente**: `World.java`, `WorldRegion.java`, `WorldObject.java`  
**Status**: VERIFIED  
**Verified**: 2026-08-23