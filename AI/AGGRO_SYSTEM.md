# AGGRO / THREAT SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: AI — aggro y amenaza  
**Source of Truth**: `entity/actor/Attackable.java`, `entity/actor/holders/npc/AggroInfo.java`, `ai/AttackableAI.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## 1. IMPORTANTE: no existe clase `AggroList`

En este servidor **no** existe una clase `AggroList`. El sistema de amenaza real es:

```
Attackable._aggroList = ConcurrentHashMap<Creature, AggroInfo>
```

- Clave: el atacante (`Creature`).
- Valor: `AggroInfo` (por atacante).

## 2. `AggroInfo` — holder de dos contadores

**Path**: `entity/actor/holders/npc/AggroInfo.java`

| Campo | Propósito |
|-------|-----------|
| `_aggressor` | atacante (Creature) |
| `_damageTotal` | daño total hecho (contribución a rewards) |
| `_hateTotal` | nivel de odio/hate (selección de target) |
| `LIMIT = 1000000000000000L` | cap de tanto daño como hate |

- `addDamage(long)`, `addHate(long)` — suman con cap si positivo; valores negativos se aplican directos.
- `stopHate()` — resetea hate a 0.
- `checkHate(owner)` — **valida y resetea hate a 0 si**:
  - `owner` o `aggressor` null;
  - el atacante `isAlikeDead()`;
  - el atacante no está `isSpawned()`;
  - el atacante no está en `owner.isInSurroundingRegion(atacante)`.

Esto significa: si el atacante muere, se des-spawnea o se aleja de la región, pierde la amenaza automáticamente.

---

## 3. Métodos de aggro en `Attackable`

| Método | Descripción |
|--------|-------------|
| `_aggroList` (campo) | `Map<Creature, AggroInfo>` `ConcurrentHashMap` |
| `getAggroList()` | devuelve el mapa |
| `addDamage(Creature attacker, int damage, Skill skill)` | añade daño y hate |
| `addDamageHate(attacker, damage, aggroValue)` | suma daño y hate al `AggroInfo` del atacante |
| `reduceHate(target, amount)` | reduce hate (target null → todos) |
| `getMostHated()` | devuelve el `Creature` con mayor `hate` válido |
| `getHateList()` | lista validada (llama `checkHate`) |
| `getHating(target)` | hate de un target |
| `isInAggroList(target)` | si existe entrada |
| `clearAggroList()` | limpia el mapa y el overhit |
| `getMainDamageDealer()` | máx. `damageTotal` dentro de rango |

### `addDamage` (detalle real, verificado)
```java
public void addDamage(Creature attacker, int damage, Skill skill) {
    if (attacker == null) return;
    if (!isDead()) {
        if (isWalker() && ...) stopMove;
        getAI().notifyActionAttacked(attacker);
        long hateValue = (long) ((((long) damage * 100) / (getLevel() + 7)) * getHateRatio(attacker));
        addDamageHate(attacker, damage, (int) hateValue);
        if (EventDispatcher.getInstance().hasListener(EventType.ON_ATTACKABLE_ATTACK, this)) {
            EventDispatcher.getInstance().notifyEventAsync(new OnAttackableAttack(...), this);
        }
    }
}
```

### `addDamageHate` (verificado)
```java
public void addDamageHate(Creature attacker, long damage, long aggroValue) {
    // guarda fake players / traps
    AggroInfo ai = _aggroList.computeIfAbsent(attacker, AggroInfo::new);
    ai.addDamage(damage);
    if (targetPlayer.getTrap() == null) ai.addHate(aggroValue);
    // ... etc
}
```
Notas:
- Las trampas (`Trap`) **no** causan hate.
- Si el aggroValue es 0, se agrega `addDamageHate(attacker, 0, 1)` (fake min).

---

## 4. Selección de target (aggro→threat)

`AttackableAI.thinkAttack()`:

```
mostHate = npc.getMostHated();
if (mostHate == null) { setIntentionActive(); return; }
if (getAttackTarget() != mostHate) setAttackTarget(mostHate);
if (getTarget() != mostHate) setTarget(mostHate);
```

`getMostHated()` devuelve el atacante con mayor `hate` entre las entradas **validas** (tras `Ai.checkHate`).

**Target change en raids**: para `RaidBoss`/`GrandBoss`/minions, `targetReconsider(boolean randomTarget)` usa la aggro list (y en modo random) para cambiar target según `_chaosTime`.

---

## 5. Qué pasa cuando...

- **El target muere**: en `thinkAttack`, si `mostHate.isAlikeDead()` → `npc.stopHating(mostHate)`.
- **El target desaparece (despawn/lejos)**: `AggroInfo.checkHate` resetea el hate; el target deja de aparecer en `getMostHated`.
- **El monstruo pierde aggro**: cuando `mostHate == null` → `setIntentionActive()`. Si además el tiempo de ataque expiró → `clearAggroList()` + `getAttackByList().clear()` + vuelta a spawn (monstruos).
- **Una trampa hace daño**: no genera hate (por diseño).
- **Fake players**: `FAKE_PLAYER_AGGRO_FPC` en config; si fake vs fake y desactivado, no aggro.

---

## 6. Recompensas relacionadas con aggro (en `Attackable`)

- `getMainDamageDealer()` — el atacante con mayor `damageTotal` dentro de `ALT_PARTY_RANGE` → usado en `doDie` para `calculateRewards`.
- `calculateRewards(lastAttacker)` — usa la aggro list para dar EXP/SP/drops al mayor daño y su party.
- `doItemDrop(mainDamageDealer)` — calcula drops base/quests/evento.

(Detalles en `COMBAT/DEATH_FLOW.md`.)

---

## 6b. ⚠️ QUEST PARTY CREDIT ≠ GENERAL MOB DROP/REWARD DISTRIBUTION

**No confundir dos mecanismos distintos:**

| Mecanismo | Qué decide | Implementación | Documento |
|---|---|---|---|
| **NORMAL MOB REWARD/DROP** (EXP/SP/drops del mob) | Quién recibe EXP/SP/drop **normal**: el `mainDamageDealer` (mayor `damageTotal` dentro de `ALT_PARTY_RANGE`) y su party vía `calculateRewards`; drops vía `doItemDrop(mainDamageDealer)`. | `Attackable#getMainDamageDealer()` · `calculateRewards(lastAttacker)` · `doItemDrop(...)` (flujo en `Creature.doDie`) | Este doc §6 + [COMBAT/DEATH_FLOW.md](../COMBAT/DEATH_FLOW.md) |
| **QUEST PARTY CREDIT** (quest items / progreso de quest) | A qué **miembro de party con la quest elegible** (`cond` match) se le atribuye el ítem/progreso de quest al morir un mob registrado. | `Quest#getRandomPartyMember*(...)` · `getRandomPartyMemberState(...)` — selección aleatoria (uniforme o ponderada por killer). | [QUESTS/QUEST_PARTY_CREDIT.md](../QUESTS/QUEST_PARTY_CREDIT.md) |

- Ambos usan el mismo radio configurado (`ALT_PARTY_RANGE = 1500`) pero **deciden cosas distintas**.
- Un jugador puede recibir el **drop normal** (por ser main damage dealer) mientras el **quest item** va a otro miembro de party que cumpla la condición de la quest — y viceversa.

---


## 7. Trazabilidad

| Clase/Método | Path |
|--------------|------|
| `Attackable._aggroList` | `entity/actor/Attackable.java` |
| `AggroInfo` | `entity/actor/holders/npc/AggroInfo.java` |
| `addDamage`, `addDamageHate`, `reduceHate`, `getMostHated`, `getHateList`, `getHating`, `clearAggroList`, `getMainDamageDealer` | `entity/actor/Attackable.java` |
| `AttackableAI.thinkAttack`, `notifyActionAggression`, `targetReconsider` | `ai/AttackableAI.java` |
| Eventos `OnAttackableHate`, `OnAttackableAggroRangeEnter`, `OnAttackableFactionCall` | `mechanics/events/holders/actor/npc/attackable/` |

---
**Status**: VERIFIED  
**Verified**: 2026-08-23