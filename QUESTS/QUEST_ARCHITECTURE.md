# QUEST ARCHITECTURE

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Sistema de Quests — arquitectura  
**Source of Truth**: `gameserver/mechanics/script/`, `gameserver/managers/ScriptManager.java`, `dist/game/data/scripts/quests/`  
**Verified**: 2026-08-23  
**Status**: VERIFIED

> El código fuente es la única fuente de verdad. Este documento describe la estructura REAL de ESTE servidor.

---

## 1. DOS CAPAS (core vs datapack)

| Capa | Ubicación | Contenido |
|------|-----------|-----------|
| **Core Java** | `java/org/l2jmobius/gameserver/mechanics/script/` (10 archivos) | Motor: `Quest`, `QuestState`, `State`, `QuestTimer`, `Script`, `Event`, `LongTimeEvent`, `InstanceScript`, `QuestType`, `QuestSound` |
| **Datapack (scripts compilados en runtime)** | `dist/game/data/scripts/quests/` — **543 archivos .java en 511 carpetas** | Una carpeta por quest, p.ej. `Q00001_LettersOfLove/Q00001_LettersOfLove.java` |

Las quests NO tienen XML propio: los directorios de quests contienen solo `.java` (verificado). El ScriptEngine compila y ejecuta estos scripts al arranque.

## 2. JERARQUÍA DE CLASES (declaraciones verificadas)

```
mechanics/script/
├── Quest.java            public class Quest implements IEventTimerEvent<String>, IEventTimerCancel<String>
│   ├── Script.java       public abstract class Script extends Quest
│   │   ├── Event.java    public abstract class Event extends Script
│   │   └── LongTimeEvent.java  public class LongTimeEvent extends Script
│   ├── QuestState.java   public class QuestState
│   ├── State.java        public class State          // constantes CREATED/STARTED/COMPLETED
│   ├── QuestTimer.java   public class QuestTimer
│   ├── QuestType.java    public enum QuestType
│   ├── QuestSound.java   public enum QuestSound
│   └── InstanceScript.java                          // para instancias
```

- `Quest` es una clase **concreta** (~5000 líneas): las quests del datapack extienden directamente `extends Quest`.
- `Script` es la base para scripts no-quest (features, AI scripts, etc.).
- `Event` / `LongTimeEvent` para eventos temporales del servidor.

## 3. EJEMPLO REAL VERIFICADO (`Q00001_LettersOfLove`)

```java
package quests.Q00001_LettersOfLove;
public class Q00001_LettersOfLove extends Quest {
    private static final int DARIN = 30048;      // NPCs como constantes int
    private static final int DARINS_LETTER = 687; // Items como constantes int
    private static final int MIN_LEVEL = 2;

    public Q00001_LettersOfLove() {
        super(1);                                 // id de quest
        addStartNpc(DARIN);
        addTalkId(DARIN, ROXXY, BAULRO);
        registerQuestItems(DARINS_LETTER, ...);
    }
    @Override public String onEvent(String event, Npc npc, Player player) {...}
}
```

Convenciones reales observadas:
- Constructor llama `super(questId)` y registra NPCs/items.
- Callbacks sobrescritos devuelven `String htmltext` (o `null`).
- Constantes numéricas para NPC/item ids.

## 4. AUTO-REGISTRO EN EL CONSTRUCTOR (Quest.java L217–232)

```java
public Quest(int questId) {
    _scriptFile = Path.of(ScriptEngine.getInstance().getCurrentLoadingScript().toUri());
    initializeAnnotationListeners();
    _questId = questId;
    if (questId > 0)  ScriptManager.getInstance().addQuest(this);   // → mapa _quests
    else              ScriptManager.getInstance().addScript(this);  // → mapa _scripts
    onLoad();
}
```
→ Una quest se registra sola al instanciarse. No existe un registro manual centralizado por-ID.

## 5. NOTA DE NO-DUPLICACIÓN

La referencia de quests vive **solo** en `QUESTS/`. El doc planificado `SYSTEMS/QUEST_SYSTEM.md` fue sustituido en `MASTER_INDEX.md`.

## 6. INVESTIGACIÓN CLIENT ↔ SERVER

La dimensión CLIENT↔SERVER de las quests (correlación de IDs de quest/NPC/item, textos del cliente vs servidor, y casos piloto como `Q00001_LettersOfLove`) está documentada en [../CLIENT_RESEARCH/QUEST_PILOT_Q00001.md](../CLIENT_RESEARCH/QUEST_PILOT_Q00001.md) y [../CLIENT_RESEARCH/CLIENT_SERVER_MAPPING.md](../CLIENT_RESEARCH/CLIENT_SERVER_MAPPING.md). No se duplica aquí.

---
**Fuente**: `mechanics/script/**`, `managers/ScriptManager.java`, `scripting/ScriptEngine.java`
**Status**: VERIFIED
**Verified**: 2026-08-23
