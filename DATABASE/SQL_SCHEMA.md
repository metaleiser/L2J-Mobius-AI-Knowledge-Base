# SQL Schema

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Fase**: 2F - Database  
**Source of Truth**: `dist/db_installer/sql/login/*.sql`, `dist/db_installer/sql/game/*.sql` y consultas JDBC del servidor  
**Verificado**: 2026-08-23  
**Status**: VERIFIED para los artefactos inspeccionados; catálogo semántico completo → REQUIRES CODE VERIFICATION

## 1. Alcance y método

El esquema documentado aquí procede de los archivos SQL incluidos en este servidor. La carpeta contiene 4 scripts de login y 105 scripts de game con `CREATE TABLE IF NOT EXISTS` detectado, además de scripts que pueden contener índices, datos o sentencias auxiliares. El inventario completo de columnas debe mantenerse contra los archivos SQL, no contra nombres de tablas de otras revisiones.

El instalador real es `org.l2jmobius.tools.DatabaseInstaller`. Para la instalación consola busca `sql/<dbType>`, ordena los archivos `.sql` por nombre y ejecuta sus sentencias. El tipo concreto usado por el entorno debe verificarse con la operación/configuración elegida: el código revisado construye URLs JDBC MySQL.

## 2. Relaciones: clasificación obligatoria

- **Relación explícita por FOREIGN KEY**: no se encontró ninguna declaración `FOREIGN KEY` ni `REFERENCES` en los scripts SQL inspeccionados. Por tanto, no se documentan constraints declarativas.
- **Relación lógica inferida por código**: existe cuando una consulta o limpieza compara columnas entre tablas. Ejemplos: `items.owner_id` con `characters.charId` o `clan_data.clan_id`; `character_quests.charId` con `characters.charId`; `item_attributes.itemId` con `items.object_id`.
- **Relación no verificada**: cualquier asociación basada solo en nombres parecidos queda en esta categoría hasta encontrar una consulta, uso Java o constraint SQL que la demuestre.

La ausencia de una foreign key no significa que la relación lógica sea inexistente; significa que el SQL no la impone declarativamente.

## 3. Tablas confirmadas por dominio

### Login

- `accounts` (`login` como clave observada en el script).
- `account_data`.
- `accounts_ipauth`.
- `gameservers`.

Columnas, índices y claves completas: verificar directamente en cada script SQL. Relaciones con tablas game: **REQUIRES CODE VERIFICATION** como contrato entre bases.

### Personajes y estado

- `characters`
- `character_subclasses`
- `character_skills`
- `character_skills_save`
- `character_quests`
- `character_variables`
- `character_hennas`
- `character_shortcuts`
- `character_macroses`
- `character_contacts`
- `character_friends`
- `character_reco_bonus`
- `character_recipebook`
- `character_recipeshoplist`
- `character_raid_points`
- `character_instance_time`
- `character_tpbookmark`
- `character_transmogs`
- `character_premium_items`
- `character_item_reuse_save`
- `character_offline_trade`
- `character_offline_trade_items`
- `character_offline_play`
- `character_offline_play_group`

Ejemplo verificado: `character_quests` tiene clave primaria `(charId, name, var)` y `value VARCHAR(255)`; no se debe confundir con `character_quest` singular.

### Items y auxiliares

- `items`: clave primaria `object_id`; contiene `owner_id`, `item_id`, `count`, `loc`, `loc_data`, `enchant_level`, `time_of_use`, `mana_left` y otros campos del script.
- `item_attributes`, `item_elementals`, `item_variables`.
- `itemsonground`.
- `pets`.
- `messages`, `custom_mail`.
- `item_auction`, `item_auction_bid`.

El código de `Item`, inventarios, mail y managers usa `DatabaseFactory.getConnection()` y consultas directas. La interpretación completa de cada valor de `loc`/`loc_data` se verifica en esas clases y en `items.sql`; no se infiere solo del nombre.

### Clanes y residencias

- `clan_data`, `clan_privs`, `clan_skills`, `clan_subpledges`, `clan_wars`, `clan_notices`.
- `crests`.
- `castle`, `castle_doorupgrade`, `castle_functions`, `castle_manor_production`, `castle_manor_procure`, `castle_siege_guards`, `castle_trapupgrade`.
- `clanhall`, `clanhall_functions`, `clanhall_siege_attackers`, `clanhall_siege_guards`, `siegable_clanhall`, `siegable_hall_flagwar_attackers`, `siegable_hall_flagwar_attackers_members`.
- `fort`, `fort_doorupgrade`, `fort_functions`, `fort_siege_guards`, `fortsiege_clans`.
- `siege_clans`.

Las relaciones de propiedad y participación están demostradas lógicamente por consultas de `ClanTable` y `DatabaseIdManager`; no están declaradas como foreign keys en los SQL inspeccionados.

### Sistemas y registros especializados

También existen scripts para anuncios, subastas, pesca, lotería, Seven Signs, Olympiad, héroes, raid bosses, territorios, foros, sanciones, wedding, comercio offline, eventos y otros sistemas. La lista de tablas y sus columnas es la unión de los `CREATE TABLE` de los scripts reales; el significado funcional de cada una requiere correlación con su consumidor Java.

## 4. Integridad y limpieza

`DatabaseIdManager` ejecuta consultas de limpieza de registros huérfanos y updates de consistencia cuando `IdManagerConfig.DATABASE_CLEAN_UP` está habilitado. Esto constituye integridad lógica aplicada por el servidor, no integridad referencial del motor SQL.

## 5. Límites de verificación

- No se afirma un catálogo exhaustivo de relaciones porque no existen foreign keys que lo proporcionen y no se revisaron semánticamente los 109 scripts con tabla uno por uno.
- Tipos, defaults, índices y claves deben citar el script concreto al documentarse en detalle.
- Esquema efectivo de una base ya instalada, diferencias por migraciones y datos existentes: **UNKNOWN / REQUIRES CODE VERIFICATION**.

## Ver también

- [DATABASE_ARCHITECTURE.md](DATABASE_ARCHITECTURE.md)
- [DATABASE_CONFIGURATION.md](DATABASE_CONFIGURATION.md)
- [DATABASE_TRANSACTIONS.md](DATABASE_TRANSACTIONS.md)
- [XML_DATA_LOADING.md](XML_DATA_LOADING.md)
- [ID_MANAGEMENT.md](ID_MANAGEMENT.md)
- [QUESTS/QUEST_STATES_VARIABLES.md](../QUESTS/QUEST_STATES_VARIABLES.md)
- [SYSTEMS/ITEM_SYSTEM.md](../SYSTEMS/ITEM_SYSTEM.md)
- [SYSTEMS/PLAYER_SYSTEM.md](../SYSTEMS/PLAYER_SYSTEM.md)
