# QUEST STATES & VARIABLES

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Quests — estados, progreso, variables y persistencia  
**Source of Truth**: `mechanics/script/QuestState.java`, `mechanics/script/State.java`, `mechanics/script/Quest.java` (métodos DB estáticos)  
**Verified**: 2026-08-23  
**Status**: VERIFIED · schema SQL → DATABASE PHASE

---

## 1. ESTADOS

Definidos en `mechanics/script/State.java` (constantes de clase):
- `CREATED` — quest registrada pero no iniciada.
- `STARTED` — en progreso.
- `COMPLETED` — completada.

API verificada en `QuestState`:
```java
public byte getState()
public boolean isCreated() / isStarted() / isCompleted()
public void setState(byte state)
public void setState(byte state, boolean saveInDb)
```
Constructor: `QuestState(Quest quest, Player player, byte state)`.

> No existen estados FAILED/ABANDONED como constantes: el abandono se modela con `exitQuest(...)` (ver §3).

## 2. PROGRESO: COND

```java
public boolean isCond(int condition)
public void setCond(int value)
public void setCond(int value, boolean playQuestMiddle)
public int getCond()
```
`cond` es una variable especial (`"cond"`) usada como contador/fase dentro de STARTED. `playQuestMiddle` reproduce `QuestSound.ITEMSOUND_QUEST_MIDDLE`.

## 3. VARIABLES GENÉRICAS

```java
public void set(String variable, int value)
public void set(String variable, String value)
public void setInternal(String variable, String value)      // sin persistir/notificar
public String get(String variable)
public int getInt(String variable)
public boolean isSet(String variable)
public void unset(String variable)
```

## 4. MEMO STATE (slots estilo cliente)

```java
public void setMemoState(int value) / getMemoState() / isMemoState(int)
public void setMemoStateEx(int slot, int value) / getMemoStateEx(int slot) / isMemoStateEx(...)
```
Usado por quests modernas como bitmap/slot numérico en lugar de variables string.

## 5. INICIO / SALIDA / REINICIO

```java
public void startQuest()                                   // CREATED→STARTED (+sound/marca)
public void exitQuest(QuestType type)
public void exitQuest(QuestType type, boolean playExitQuest)
public void exitQuest(boolean repeatable)                  // borra o deja completada
public void exitQuest(boolean repeatable, boolean playExitQuest)
public void setRestartTime()                               // programa disponibilidad futura
public boolean isNowAvailable()
public void setIsExitQuestOnCleanUp(boolean) / isExitQuestOnCleanUp()
```
- `repeatable=true` → elimina registro (re-hacible); `false` → marca COMPLETED.
- `QuestType` (enum) distingue el tipo de quest al salir.

## 6. NOTIFICACIÓN DE MUERTE

```java
public void addNotifyOfDeath(Creature creature)
```
Permite que el script reaccione cuando una criatura vinculada muere (conexión con onDeath/onKill).

## 7. PERSISTENCIA (métodos estáticos de `Quest`, L1333–1442)

```java
createQuestVarInDb(qs, var, value) / updateQuestVarInDb(qs, var, value) / deleteQuestVarInDb(qs, var)
createQuestInDb(qs) / updateQuestInDb(qs) / deleteQuestInDb(qs, repeatable)
```
- Se invocan automáticamente desde `setState/set/unset/startQuest/exitQuest` según flags.
- **Schema SQL fuera de alcance** (DATABASE PHASE): solo queda mapeada la conexión.

## 8. TRAZABILIDAD

| Elemento | Path |
|----------|------|
| Estados | `mechanics/script/State.java` |
| API completa | `mechanics/script/QuestState.java` (~750 líneas) |
| Persistencia | `mechanics/script/Quest.java` L1333–1442 |
| Acceso | `quest.newQuestState(player)` / `getQuestState(player, initIfNone)` |

---
## Ver también

- [SYSTEMS/PLAYER_SYSTEM.md](../SYSTEMS/PLAYER_SYSTEM.md) — relación Player↔QuestState↔Quest

---
**Status**: VERIFIED · schema SQL → DATABASE PHASE  
**Verified**: 2026-08-23