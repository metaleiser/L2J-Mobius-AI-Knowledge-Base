# Outgoing Packets (Fase 2E)

## 1. Clase base real

**Class:** `ServerPacket` (nombre real en este build)
**Package:** `org.l2jmobius.gameserver.network.serverpackets`
**Path:** `gameserver/network/serverpackets/ServerPacket.java`
**Extends:** `WritablePacket<GameClient>`

> NOTA 1: el Javadoc antiguo habla de `ServerBasePacket`; el nombre real en este código es `ServerPacket` (verificado).
> NOTA 2 (corrección auditoría FASE 2, 2026-08-24): la clase base saliente de commons es `WritablePacket` (`commons/network/packet/WritablePacket.java`). No existe ninguna clase `SendablePacket` en este build. Declaración verificada: `public abstract class ServerPacket extends WritablePacket<GameClient>` (`ServerPacket.java:35`).

## 2. Envío al cliente (verificado indirectamente)
- `GameClient.sendPacket(ServerPacket packet)` — método real heredado del `Client` base de commons: `getConnection().sendPacket(packet)`.
- Broadcast: `World.broadcastToVisiblePlayers/VisibleObjects` + helper `packet.sendPacket(caster)` / `broadcast()`.
- En `WriteBuffer` (commons) y `ServerPackets` enum para el opcode.

## 3. Serialización (writeImpl)
- Método real: `public void writeImpl(GameClient client, WriteBuffer buffer)`.
- `ServerPackets.ATTACK.writeId(this, buffer)` escribe el opcode (visto en `serverpackets/Attack.java`).
- El orden de escritura (opcode primero, luego fields) se ve en `Attack.writeImpl`.

## 4. Ejemplos reales (389 archivos .java en serverpackets/)
- `Attack` (`serverpackets/Attack.java`): constructor `(Creature attacker, Creature target, boolean useShots, int ssGrade)`; `addHit`; en `writeImpl` serializa attackerObjId, soulshot, hits y coords.
- `NpcHtmlMessage`, `MagicSkillUse`, `StatusUpdate`, `PartySpelled`, `UserList`, etc.

## 5. Estado de verificación
- `ServerPacket extends WritablePacket<GameClient>`: **VERIFIED** (código leído, `ServerPacket.java:35`).
- `writeImpl(GameClient, WriteBuffer)` + `ServerPackets.writeId`: **VERIFIED**.
- `GameClient.sendPacket` heredado de commons `Client`: **VERIFIED** (referenciado).
- Conteo de archivos en `serverpackets/`: **389** (**VERIFIED** por recuento recursivo de filesystem 2026-08-24; antes figuraba 398/251 en distintos documentos).
- Catálogo completo y opcodes: **REQUIRES CODE VERIFICATION** (no se enumeraron todos).

Cross-ref: `PACKET_ARCHITECTURE.md`, `COMBAT/ATTACK_FLOW.md`.

