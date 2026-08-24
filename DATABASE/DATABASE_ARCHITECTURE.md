# Database Architecture

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Fase**: 2F - Database  
**Source of Truth**: `commons.database.DatabaseFactory`, `commons.config.DatabaseConfig`, `dist/db_installer/sql/`, `gameserver.data.sql.*` y consultas JDBC de entidades  
**Verificado**: 2026-08-23  
**Status**: VERIFIED para el flujo descrito; catálogo semántico completo → REQUIRES CODE VERIFICATION

## 1. Capas

- `DatabaseConfig` carga `./config/Database.ini`.
- `DatabaseFactory` crea el pool HikariCP `L2JMobiusPool`.
- El instalador usa los scripts bajo `dist/db_installer/sql/login` y `game` en el despliegue real.
- La persistencia se realiza mediante JDBC directo desde entidades, data SQL y managers; no existe una única capa DAO que encapsule todas las tablas.
- Los loaders XML y scripts cargan datos de runtime que no son tablas SQL; se describen en [XML_DATA_LOADING.md](XML_DATA_LOADING.md).

## 2. Arranque verificado

En `GameServer` el orden relevante es:

1. `ConfigLoader.init()`.
2. `DatabaseFactory.init()`.
3. `ThreadPool.init()`.
4. `IdManager.getInstance()`.
5. scripting, mundo y managers iniciales.
6. loaders XML de datos generales.
7. loaders de skills.
8. loaders de items y sus datos asociados.

El resto del orden completo debe verificarse directamente en el constructor de `GameServer` cuando una dependencia concreta sea relevante. No se conserva el diagrama genérico anterior como orden garantizado.

## 3. Persistencia real confirmada

| Componente | Evidencia de tablas |
|---|---|
| `Player` | `characters`, `character_skills`, `character_subclasses`, `character_variables`, `character_transmogs` y otras consultas del flujo de jugador |
| `Item` / inventarios | `items`, `item_attributes`, `item_elementals`, `item_variables` |
| `Quest` | `character_quests` |
| `ClanTable` / `Clan` | `clan_data`, `clan_subpledges`, y tablas auxiliares del clan |
| `AnnouncementsTable` | `announcements` |
| `CrestTable` | `crests` |
| `OfflinePlayTable` / `OfflineTraderTable` | tablas de comercio/juego offline definidas en SQL |

Las asociaciones entre tablas son principalmente lógicas, mediante columnas y consultas Java. Los SQL inspeccionados no contienen declaraciones `FOREIGN KEY` ni `REFERENCES`; el detalle está clasificado en [SQL_SCHEMA.md](SQL_SCHEMA.md).

## 4. IDs y limpieza

`IdManager` usa un `BitSet` de IDs relativos al rango configurado y `DatabaseIdManager` obtiene IDs ocupados desde `characters`, `items`, `clan_data`, `itemsonground` y `messages`. Al iniciar, puede limpiar registros huérfanos, marcar personajes offline y eliminar timestamps expirados según `IdManagerConfig`.

Ver [ID_MANAGEMENT.md](ID_MANAGEMENT.md) para el algoritmo verificado y sus límites.

## 5. Límites

- No se afirma soporte de MySQL/MariaDB como abstracción múltiple: el código revisado usa driver y URL configurables, y el instalador construye JDBC MySQL.
- No se afirma backup automático, frecuencia de respaldo ni duración de carga; esos detalles son **UNKNOWN / REQUIRES CODE VERIFICATION** si no aparecen en el flujo concreto.
- No se sustituyen nombres reales por tablas clásicas como `character_items`, `character_quest` o `clans`: no son las tablas observadas en este build.

## Ver también

- [DATABASE_CONFIGURATION.md](DATABASE_CONFIGURATION.md)
- [DATABASE_TRANSACTIONS.md](DATABASE_TRANSACTIONS.md)
- [SQL_SCHEMA.md](SQL_SCHEMA.md)
- [XML_DATA_LOADING.md](XML_DATA_LOADING.md)
- [ID_MANAGEMENT.md](ID_MANAGEMENT.md)
- [SYSTEMS/PLAYER_SYSTEM.md](../SYSTEMS/PLAYER_SYSTEM.md)
- [SYSTEMS/ITEM_SYSTEM.md](../SYSTEMS/ITEM_SYSTEM.md)
- [QUESTS/QUEST_STATES_VARIABLES.md](../QUESTS/QUEST_STATES_VARIABLES.md)
