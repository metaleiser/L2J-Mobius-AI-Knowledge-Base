# QUEST LIFECYCLE

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Quests — ciclo de vida, carga y registro  
**Source of Truth**: `scripting/ScriptEngine.java`, `managers/ScriptManager.java`, `mechanics/script/Quest.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## 1. CARGA AL ARRANQUE

```
GameServer startup (fase scripts)
   → ScriptEngine.getInstance().load()          [implements IXmlReader]
   → executeScriptList()
       → processDirectory(SCRIPT_FOLDER.toFile(), files)   // recolecta .java
       → ScriptExecutor.executeScripts(files)               // compila y ejecuta
```

Detalles verificados en `ScriptEngine.java`:
- `SCRIPT_FOLDER = ServerConfig.SCRIPT_ROOT.toPath()` (raíz: `dist/game/data/scripts/`).
- `Files.walkFileTree` recorre el árbol; incluye solo `.java` que no estén en `EXCLUSIONS`.
- `MASTER_HANDLER_FILE` y `EFFECT_MASTER_HANDLER_FILE` son entradas especiales (`handlers/MasterHandler.java`, `handlers/EffectMasterHandler.java`).
- **Gate**: `DevelopmentConfig.NO_QUESTS` (L191) permite desactivar la carga de quests.
- Errores: `"ScriptEngine: <file> failed execution!"`.

## 2. REGISTRO

- Cada script quest se instancia al ejecutarse; su constructor llama a `ScriptManager.addQuest(this)` si `questId > 0`, o `addScript(this)` si no.
- `ScriptManager` (singleton implícito, `managers/ScriptManager.java`):
  - `_quests: Map<String, Quest>` (clave = `quest.getName()`), `_scripts: Map<String, Quest>` (clave = simple name de clase).
  - `addQuest(Quest)` L181 / `addScript(Quest)` L256.
  - `getScript(String name)` consulta primero `_quests` y luego `_scripts`.
- **No existe `QuestManager`** en este servidor.

## 3. RECARGA / DESCARGA / GUARDADO (ScriptManager)

| Método | Función |
|--------|---------|
| `reload(String questFolder)` / `reload(int questId)` | unload + load de una quest concreta |
| `reloadAllScripts()` / `unloadAllScripts()` | masivo |
| `save()` | llama `onSave()` en todas las quests/scripts |
| `report()` | log `"Loaded N quests"` |

En `Quest`: `reload()` (L341), `unload([removeFromList])` (L355/360), `onSave()` (L406), `onLoad()` hook.

## 4. CICLO DE VIDA DE UNA QUEST INDIVIDUAL

```
ScriptEngine compila Q00001.java
   ↓ new Q00001_LettersOfLove()
      → super(id): captura _scriptFile + initializeAnnotationListeners()
      → ScriptManager.addQuest(this)
      → onLoad()                                  // hook opcional del script
   ↓ runtime: callbacks por eventos (ver QUEST_EVENTS.md)
   ↓ shutdown/reload: onSave() / unload() / removeListeners
```

## 5. CICLO DEL JUGADOR CON LA QUEST

- Entrada al mundo: `Quest.playerEnter(Player)` (estático, L1239) — notifica a quests con listener de login.
- Estado por jugador: `quest.newQuestState(player)` / `getQuestState(player, initIfNone)`.
- Persistencia: métodos estáticos DB de Quest (ver QUEST_STATES_VARIABLES.md §persistencia).

## 6. TRAZABILIDAD

| Elemento | Path |
|----------|------|
| Engine | `gameserver/scripting/ScriptEngine.java` (+ `ScriptExecutor`, `ScriptClassLoader`, `ScriptFileManager`) |
| Manager | `gameserver/managers/ScriptManager.java` |
| Auto-registro | `mechanics/script/Quest.java` L217–232 |
| Gate NO_QUESTS | `config/custom/DevelopmentConfig` (uso en ScriptEngine L191) |

---
**Status**: VERIFIED · lista exacta de EXCLUSIONS → REQUIRES CODE VERIFICATION  
**Verified**: 2026-08-23
