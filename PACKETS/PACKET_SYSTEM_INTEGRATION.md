# Packet System Integration (Fase 2E)

## 1. Objetivo
Este documento mapea **cómo los packets entran/salen** de los sistemas ya documentados (AI/COMBAT/Skills/Quests/Items). **No redefine** esos sistemas — documenta únicamente las **conexiones en la capa packet**.

## 2. Matriz de conexión verificada

| Sistema conectado | Packet incoming (ejemplo real) | Método/entrada real | Salida verificada | Verificado |
|---|---|---|---|---|
| **AI** | `AttackRequest` (0x01) | `target.onAction(player)` / `player.onActionRequest()` → `PlayerAI/CreatureAI` | `ActionFailed`, `MagicSkillUse`, `Attack` | VERIFIED |
| **Combat** | `AttackRequest` (0x01) | `target.onForcedAttack(player)` → `Creature.doAttack` | `Attack` packet (`ServerPackets.ATTACK`) | VERIFIED |
| **Skills** | `RequestActionUse` (0x1A) | helper `useSkill(player, name, targetSelf)` → `Player.useMagic(skill)` → `Creature.beginCast` | `MagicSkillUse/Canceled` (`ServerPackets`) | VERIFIED |
| **Quests** | `RequestBypassToServer` (0x23) | `BypassHandler` / `Quest`-eventos; también `RequestLinkHtml` | `NpcHtmlMessage` (`NPC_HTML_MESSAGE`) | VERIFIED |
| **Items** | `UseItem` (0x19) | `player.useItem(...)` (en `RequestActionUse`) | `ItemList`, `InventoryUpdate` | VERIFIED |
| **Movimiento** | `MoveToLocation` (0x0F), `ValidatePosition` | `Player.moveToLocation(...)` / `player.setXYZ` | `MoveToObject/StopMove` | VERIFIED (names) |
| **Estado** | `RequestSkillCoolTime` (0x64), `RequestSkillList` (0x15) | lectura de cooldowns/skills del player | `AcquireSkillList`, `SkillCoolTime` | VERIFIED (names) |
| **Logout** | `Logout` (0x00) | `player.logout()` / `Disconnection` | `LeaveWorld`, `KickInactive` | VERIFIED (names) |

## 3. Detalle de conexiones clave (verificado)

### a) Ataque (INTEGRACIÓN COMBAT)
```java
// AttackRequest.runImpl — solo redirige, no calcula daño
target.onAction(player);            // → PlayerAI setIntention/notifyAction
// o target.onForcedAttack(player) — → Creature.doAttack (combat layer real)
```
- El daño real se calcula en `Creature.doAttack → doAttackHitBy* → onHitTimer → reduceCurrentHp` (ver COMBAT/ATTACK_FLOW, DAMAGE_CALCULATION).
- Salida: serverpacket `Attack` (opcode 0x33) construido en la capa combat, no en el clientpacket.

### b) Skill (INTEGRACIÓN SKILLS)
```java
// RequestActionUse.runImpl → useSkill(player, "DDMagic", targetSelf)  [método helper del propio packet]
→ player.useMagic(skill)            // Playable (abstract) → Player
→ Creature.beginCast(MagicUseTask)  // SKILLS/CAST_FLOW
```
- `RequestActionUse` importa `Skill`, `SkillHolder`, `AbnormalType`, `BuffInfo`, `SkillData` (verificado).
- Costes de MP, target validation y casting se resuelven en la capa Skills/Creature (SKILLS/CAST_FLOW, COMBAT).

### c) Quest dialog (INTEGRACIÓN QUESTS)
```java
// RequestBypassToServer.runImpl
final String _command = readString();  // bypass string, ej. "quest_start"
...
final IBypassHandler handler = BypassHandler.getInstance().getHandler(_command);
handler.onCommand(_command, player, lastNpc);
```
- El bypass string viaja: **cliente → RequestBypassToServer → BypassHandler/Quest.onEvent/onTalk → script quest → String htmltext → NpcHtmlMessage serverpacket**.
- También: `OnPlayerBypass` evento en `EventDispatcher`.

## 4. ServerPacket envío (conexión genérica)
- `GameClient.sendPacket(ServerPacket)` (heredado de commons `Client`) → `Connection<GameClient>.sendPacket(packet)` → serializa vía `WriteBuffer` + `ServerPackets.writeId` (escribe opcode) + `writeImpl(client, buffer)`.
- Broadcast: `World.broadcastToVisiblePlayers(...)` / `Creature.broadcastPacket(...)`.

## 5. Estado de verificación
- Todas las filas de la tabla: **VERIFIED** (imports/usos en `AttackRequest`, `RequestActionUse`, `RequestBypassToServer`).
- Método `GameClient.sendPacket` heredado de commons `Client`: **VERIFIED** (referenciado por imports/usos).
- `Connection<GameClient>.sendPacket` + serialización completa: **REQUIRES CODE VERIFICATION**.
- Lista completa de packets para cada integración: **REQUIRES CODE VERIFICATION**.

## 6. Limitaciones
- No se profundiza en los packets de login, chat, comercio, clan, partyes, etc. — están fuera del alcance de esta fase.
- No se incluye el routing de opcodes ex específicos ni el detalle de Serializables de cada serverpacket (PACKETS PHASE completo o futuro).

Cross-ref: `PACKETS/PACKET_ARCHITECTURE.md`, `PACKETS/PACKET_DISPATCH.md`, `PACKETS/PACKET_THREADING.md`, `COMBAT/ATTACK_FLOW.md`, `SKILLS/CAST_FLOW.md`, `QUESTS/QUEST_PLAYER_NPC_DIALOG.md`, `SYSTEMS/PLAYER_SYSTEM.md`, `SYSTEMS/NPC_SYSTEM.md`, `SYSTEMS/ITEM_SYSTEM.md`
