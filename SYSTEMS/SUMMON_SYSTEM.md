# SUMMON SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Fase**: Criaturas invocadas  
**Source of Truth**: `entity/actor/Summon.java`, `entity/actor/instance/Pet.java`, `entity/actor/instance/Servitor.java`, `gameserver/ai/SummonAI.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED (estructura/relaciones; AI se profundiza en fase AI)

---

## 1. CLASE `Summon` (abstracta)

- **Class**: `Summon`
- **Package**: `org.l2jmobius.gameserver.entity.actor`
- **Path**: `entity/actor/Summon.java`
- **Extends**: `Playable` (abstract)
- **Implements**: —

Responsibility: entidad controlada por el jugador (`Player`). Es hija de `Playable` (al igual que `Player`), NO de `Npc`. Esto es un punto de diseño clave y verificado.

Métodos verificados:
- `getOwner(): Player`
- `unSummon(Player owner)`
- Usa `SummonAI` y `ExperienceData`, `ItemData`.

---

## 2. SUBTIPOS CONCRETOS

### Pet
- **Class**: `Pet`
- **Package**: org.l2jmobius.gameserver.entity.actor.instance
- **Path**: `entity/actor/instance/Pet.java`
- **Extends**: `Summon`
- **Implements**: —
- Respons: mascota (controlada por item), montable, alimentable.
- Campos: `_controlObjectId`, `_respawned`, `_mountable`, `_feedTask`, `_data` (PetData), `_leveldata` (PetLevelData), `_expBeforeDeath`, `_curWeightPenalty`.

### Servitor
- **Class**: `Servitor`
- **Package**: ídem
- **Path**: `entity/actor/instance/Servitor.java`
- **Extends**: `Summon`
- **Implements**: `Runnable`
- Responsability: invocado por skill, con lifetime limitado y consumo de item.
- Campos: `_expMultiplier`, `_itemConsume`, `_lifeTime`, `_lifeTimeRemaining`, `_consumeItemInterval-Remaining`, `_summonLifeTask`, `_referenceSkill`.

### Otros relacionados
- `BabyPet` (verificado como clase en instance/, submascota).
- `TamedBeast` (Monster) y `FeedableBeast` (Monster) **no** son Summon (están en la rama Monster), pero están relacionados con mascotas/mascota—documentación de MONSTER_SYSTEM.

---

## 3. RELACIÓN CON PLAYER

- El jugador tiene un `Summon` accesible vía `Player.getSummon()`.
- `World` mantiene `_petsInstance: Map<Integer,Pet>` indexada por `ownerId` (`World.getPet(ownerId)`/`addPet`/`removePet`).

---

## 4. RELACIÓN CON WORLD Y SPAWN/DESPAWN

- Desde el punto de vista del mundo no es un agregado especial; se registra/elimina como un `WorldObject` (heredado).
- `Summon` implementa `spawnMe/decayMe` vía `WorldObject`; la muerte/lifetime dispara `unSummon`/limpieza.
- NOTA: el puente exacto (qué método del `Summon` llama `World.removeObject` al despawnar) quedó `REQUIRES CODE VERIFICATION`.

---

## 5. AI

`SummonAI` es una subclase de `PlayableAI`/`CreatureAI`; su bucle se documenta en fase AI_SYSTEM.

---

## 6. UNKNOWN / REQUIRES CODE VERIFICATION

- El método interno de despawn (`doDie`/`doDespawn`) no se confirmó por nombre exacto en Summon en esta fase → marcar `REQUIRES CODE VERIFICATION`.
- El cálculo exacto de consumo de item / lifetime se deja para fase de lógica de invocación.

---
**Fuente**: `Summon.java`, `Pet.java`, `Servitor.java`, `Player.java`, `World.java`  
**Status**: VERIFIED (estructura/relaciones)  
**Verified**: 2026-08-23