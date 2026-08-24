# ID Management

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Fase**: 2F - Database  
**Source of Truth**: `gameserver.managers.IdManager`, `DatabaseIdManager`, `IdManagerConfig` y consumidores de `getNextId()`/`releaseId()`  
**Verificado**: 2026-08-23  
**Status**: VERIFIED para el flujo descrito

## 1. Configuración

`IdManagerConfig.load()` lee `./config/IdManager.ini` mediante `ConfigReader`:

| Clave | Default real |
|---|---:|
| `DatabaseCleanUp` | `true` |
| `FirstObjectId` | `268435456` |
| `LastObjectId` | `2147483647` |
| `InitialCapacity` | `100000`, limitado por `LAST_OBJECT_ID - 1` |
| `ResizeThreshold` | `0.9` |
| `ResizeMultiplier` | `1.1` |

`ConfigLoader.init()` carga esta configuración antes de `DatabaseFactory.init()` y de la primera obtención de `IdManager.getInstance()` en `GameServer`.

## 2. Inicialización

`IdManager` usa el patrón holder singleton (`getInstance()`). Su constructor:

1. ejecuta `DatabaseIdManager.cleanDatabase()`;
2. ejecuta `cleanCharacterStatus()` y `cleanTimestamps()`;
3. crea un `BitSet` con capacidad inicial asignada por `PrimeCapacityAllocator.nextCapacity`;
4. calcula el total del rango como `LAST_OBJECT_ID - FIRST_OBJECT_ID`;
5. obtiene IDs usados desde `DatabaseIdManager.getUsedIds()`;
6. marca esos IDs en el `BitSet` y calcula `_nextFreeId` con `nextClearBit(0)`.

El `BitSet` representa posiciones relativas al primer ID: posición `0` corresponde a `FIRST_OBJECT_ID`.

## 3. Asignación y liberación

`getNextId()` adquiere un `ReentrantLock`, marca la siguiente posición libre, decrementa `_freeIdCount` y devuelve la posición más `FIRST_OBJECT_ID`. Calcula la utilización y puede ampliar la capacidad cuando alcanza `RESIZE_THRESHOLD`.

`releaseId(objectId)` usa el mismo lock y limpia la posición relativa si el ID no es menor que `FIRST_OBJECT_ID`. Los IDs liberados vuelven a estar disponibles. No se verificó una garantía de no reutilización permanente; de hecho, el método de liberación demuestra reutilización posible.

Consumidores confirmados incluyen `Player`, `Creature`, `Item`, `ClanTable`, `ItemManager`, `MailManager`, objetos de mundo, fences, barcos, eventos, peticiones y participantes de Olympiad.

## 4. Concurrencia y límites

La asignación, liberación y consulta de disponibles están protegidas por el `ReentrantLock`. `DatabaseIdManager` usa estructuras concurrentes y ejecutores en la recuperación de IDs usados, según el código revisado.

El comportamiento exacto cuando el rango se agota, cuando `BitSet.nextClearBit` devuelve una posición fuera de la capacidad o cuando la configuración tiene valores inválidos requiere pruebas de borde; se mantiene **UNKNOWN / REQUIRES CODE VERIFICATION**. El código declara una excepción para falta de IDs válidos, pero no se documenta aquí un resultado operativo más allá de esa ruta.

## 5. IDs usados y limpieza de BD

`DatabaseIdManager.getUsedIds()` consulta estas fuentes:

- `characters.charId`
- `items.object_id`
- `clan_data.clan_id`
- `itemsonground.object_id`
- `messages.messageId`

Con `DATABASE_CLEAN_UP=true`, `cleanDatabase()` elimina registros huérfanos y actualiza referencias de clanes/residencias según sus consultas. También pone `characters.online = 0` y elimina entradas expiradas de `character_instance_time` y `character_skills_save`.

Estas relaciones son integridad lógica ejecutada por Java/JDBC; no se verificaron foreign keys equivalentes en los SQL inspeccionados.

## 6. Trazabilidad y límites

- Rango y defaults: `IdManagerConfig.java`.
- Asignación/liberación: `IdManager.java`.
- Limpieza y extracción: `DatabaseIdManager.java`.
- Arranque: `GameServer.java`.
- Algoritmo de `PrimeCapacityAllocator`, impacto de overflow y comportamiento completo de IDs inválidos: **UNKNOWN / REQUIRES CODE VERIFICATION**.

## Ver también

- [DATABASE_ARCHITECTURE.md](DATABASE_ARCHITECTURE.md)
- [DATABASE_CONFIGURATION.md](DATABASE_CONFIGURATION.md)
- [DATABASE/SQL_SCHEMA.md](SQL_SCHEMA.md)
- [SYSTEMS/ENTITY_SYSTEM.md](../SYSTEMS/ENTITY_SYSTEM.md)
- [SYSTEMS/ITEM_SYSTEM.md](../SYSTEMS/ITEM_SYSTEM.md)
