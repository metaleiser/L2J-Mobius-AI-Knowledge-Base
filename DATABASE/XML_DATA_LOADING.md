# XML Data Loading

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Fase**: 2F - Database  
**Source of Truth**: `commons.util.IXmlReader`, `gameserver.data.xml.*`, `GameServer.java` y `ServerConfig.DATAPACK_ROOT`  
**Verificado**: 2026-08-23  
**Status**: VERIFIED para el mecanismo descrito

## 1. Contrato común

Los loaders que implementan `org.l2jmobius.commons.util.IXmlReader` exponen `load()` y, normalmente, `parseDocument(Document, File)`. `parseFile(File)`:

1. valida que el archivo exista y sea XML válido;
2. crea un `DocumentBuilderFactory` namespace-aware;
3. configura validación según `isValidating()`;
4. ignora comentarios;
5. parsea el DOM y delega en `parseDocument`;
6. registra errores XML con archivo, línea y columna cuando están disponibles.

`parseDatapackFile` y `parseDatapackDirectory` resuelven rutas relativas desde `.`. Los loaders que usan `ServerConfig.DATAPACK_ROOT` construyen explícitamente la ruta del datapack.

## 2. Ubicaciones reales

El código distingue varias familias, no una única carpeta `data/xml`:

- `data/stats/skills` y opcionalmente `data/stats/skills/custom`: `SkillData`, mediante `DocumentSkill`.
- `data/stats/items` y opcionalmente `data/stats/items/custom`: `ItemData`, mediante `DocumentItem`.
- `data/stats/npcs`, con custom recursivo: `NpcData`.
- `data/stats/players/templates`: `PlayerTemplateData`.
- `data/stats/players/skillTrees`: `SkillTreeData`.
- `data/stats/cubics`: `CubicData`, recursivo.
- `data/stats/armorsets`, `data/stats/pets`, `data/stats/transformations`, `data/stats/augmentation/options` y otras rutas específicas de sus loaders.
- `data/buylists` y `data/buylists/custom`: `BuyListData`.
- `data/multisell` y `data/multisell/custom`: `MultisellData`.
- `data/mapregion`: `MapRegionData`.
- `data/teleporters`: `TeleporterData`, recursivo.
- `data/spawns`: `SpawnData`, recursivo.

`data/xml` aparece en documentación histórica, pero no es una ruta universal demostrada para todos los loaders. Cada loader debe documentarse con su llamada real.

## 3. Flujo observado en GameServer

En el constructor de `GameServer` se verifica este orden general:

1. `ConfigLoader.init()`.
2. `DatabaseFactory.init()`.
3. `ThreadPool.init()`.
4. inicialización de `IdManager`, scripting y mundo.
5. loaders iniciales como `CategoryData`, `CubicData`, `DynamicExpRateData` y `SecondaryAuthData`.
6. loaders de skills: handlers, enchant groups, skill trees, `SkillData` y pet skills.
7. loaders de items: `ItemData`, enchant, atributos, opciones, buy lists, multisell, recetas y armor sets.

La continuación incluye loaders de personajes, NPCs, spawns y otros sistemas; el orden exacto completo debe citarse desde el tramo correspondiente de `GameServer.java` cuando se necesite una auditoría exhaustiva.

## 4. Paralelismo y reload

Cuando `ThreadConfig.THREADS_FOR_LOADING` está activo, `IXmlReader.parseDirectory` programa tareas en un executor y espera cada `Future`; si está desactivado, recorre los archivos secuencialmente. `SkillData` e `ItemData` tienen además lógica propia para recopilar archivos y cargar en paralelo usando `ThreadPool`.

Los loaders pueden limpiar sus estructuras antes de recargar. `SpawnData.load()` elimina grupos y tareas anteriores antes de parsear `data/spawns`. La disponibilidad de reload y sus efectos concretos son propios de cada loader: no se generaliza a todos.

## 5. XML, parsers y XSD

- `IXmlReader` usa JAXP DOM y puede activar validación.
- `SkillData` delega el contenido en `DocumentSkill`.
- `ItemData` delega el contenido en `DocumentItem`.
- `SpawnData` implementa su propio `parseDocument` y además registra archivos XML no procesados.

No se afirma que todos los XML tengan un XSD asociado. La existencia, ruta y uso efectivo de un XSD deben verificarse por loader y archivo. El parseo de tags específicos, defaults y errores semánticos fuera del código leído queda **UNKNOWN / REQUIRES CODE VERIFICATION**.

## 6. Referencias

- [DATABASE_ARCHITECTURE.md](DATABASE_ARCHITECTURE.md)
- [DATABASE_CONFIGURATION.md](DATABASE_CONFIGURATION.md)
- [SKILLS/SKILL_LOADING.md](../SKILLS/SKILL_LOADING.md)
- [QUESTS/QUEST_LIFECYCLE.md](../QUESTS/QUEST_LIFECYCLE.md)
- [SYSTEMS/WORLD_SYSTEM.md](../SYSTEMS/WORLD_SYSTEM.md)
