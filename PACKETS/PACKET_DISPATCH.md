# Packet Dispatch (Fase 2E)

## 1. Arquitectura REAL de dispatch (verificado)

### Capa de commons (shared)
- **`commons.network.handler.ReadHandler<T>`** (`implements CompletionHandler<Integer,T>`) — `completed()`, `handleHeader()`, `handlePayload()`, `parseAndExecutePacket()`:
  1. `buffer.flip()`; `dataSize = Short.toUnsignedInt(buffer.getShort()) - HEADER_SIZE`.
  2. `dataSize <= 0` (keep-alive) → silent skip.
  3. `dataSize > MAX_PACKET_SIZE (65533)` → **`client.disconnect()`** (protección OOM).
  4. `client.readPayload(dataSize)` → `handlePayload`.
  5. `handlePayload`: `buffer.flip()`; `decrypt(buffer,0,remaining)` → si falla, se cierra; si OK → `packetHandler.handle(buffer, client)` → `packet.init(client,buffer)`; si `packet.read()` OK → `_executor.execute(packet)`.
- **`commons.network.packet.PacketHandler<T>`** (FunctionalInterface): `ReadablePacket<T> handle(ReadBuffer, T client)`.
- **`commons.network.packet.PacketExecutor<T>`** — ejecuta el `Runnable` packet (NO el cliente thread).
- El cliente NIO (`org.l2jmobius.commons.network.*`) mantiene la conexión; `Connection<GameClient>`.

### Capa de GameServer
- **`GamePacketHandler implements PacketHandler<GameClient>`** (`gameserver/network/`):
  - Lee primer byte → `packetId`.
  - `packetId == 0xD0` → ex-packet: sub-opcode short → `ExClientPackets.PACKET_ARRAY[exPacketId]`.
  - subopcode `0x51` (bookmarks) → switch manual → `RequestBookMarkSlotInfo/RequestSaveBookMarkSlot/RequestModifyBookMarkSlot/RequestDeleteBookMarkSlot/RequestTeleportBookMark/RequestChangeBookMarkSlot`.
  - Resto de ex: `packetEnum.newPacket()`.
  - **Validación de estado**: `if (!packetEnum.getConnectionStates().contains(client.getConnectionState())) return null;` — descartado silenciosamente.
  - Opcode fuera de rango `[0, ClientPackets.PACKET_ARRAY.length]` → `null`.
- **`ClientPackets`** (enum) — `PACKET_ARRAY` estático indexado por opcode; cada entry `(packetId, Supplier<ClientPacket>, ConnectionState...)` (p.ej. `ATTACK(0x01, AttackRequest::new, IN_GAME)`).
- **`ExClientPackets`** (enum) — para ex-packets, con `PACKET_ARRAY` indexado por sub-opcode.

### No existe "PacketHandler" genérico alternativo
- `AttackRequest` no existe como clase → el packet real es **`AttackRequest`** (opcode `0x01`, estado `IN_GAME`). El Javadoc antiguo usaba `Attack`; en este build el enum se llama ATTACK y la clase AttackRequest.

## 2. Mapeo opcode → clase (verificado)

| Enum | Opcode base | Substructura | Estados |
|---|---|---|---|
| `ClientPackets` | 1 byte (0x00–0xFF) | `PACKET_ARRAY[packetId]` → entry con `Supplier` | CONNECTED/AUTHENTICATED/ENTERING/IN_GAME |
| `ServerPackets` | 1 byte | para ex: `(0xFE, subId)` | server-only |
| `ExClientPackets` | byte + short | `PACKET_ARRAY[exPacketId]` | varios |

### Nota sobre nombres de clase
- `ClientPackets.ATTACK(0x01, AttackRequest::new, ConnectionState.IN_GAME)` — el enum se llama `ATTACK`, instancia `AttackRequest`.
- `ServerPackets.ATTACK(0x33)` → `serverpackets/Attack.java` (la clase serverpacket).

## 3. Thread de ejecución
1. **NIO client thread** — solo I/O: lectura del header/payload + decrypt.
2. **`PacketExecutor` ThreadPool** — `_executor.execute(packet)` → `ClientPacket.run()` → `runImpl()`.
   - NO ejecuta en el client thread (verificado en ReadHandler).
3. Cada packet heredado (`Runnable`) puede lanzar más `ThreadPool` task si necesita asíncrono.

## 4. Estado de verificación
- `GamePacketHandler.handle` dispatch + state check + ex-packet + bookmark switch: **VERIFIED** (L1–144).
- `ClientPackets` enum, `PACKET_ARRAY`, `newPacket`, `getConnectionStates`, `getPacketId`: **VERIFIED** (L208–269).
- `ServerPackets` enum con writeId + debug flag: **VERIFIED** (L30–549).
- `ReadHandler` header/payload/decrypt/dispatch: **VERIFIED** (L64–163).
- Catálogo completo de los ~130 enums `ClientPackets`/`ExClientPackets`: **REQUIRES CODE VERIFICATION** (no se enumeran todos).

## 5. Conexiones
- **Security**: state check antes de instanciar; flood-check dentro del propio packet (p.ej. `AttackRequest` → `getFloodProtectors().canPerformPlayerAction()`).
- **Threading**: ejecución en `PacketExecutor`, no en client thread.
- **Player flow**: `runImpl → getPlayer()` → acciones.
- **Quest dialog**: `RequestBypassToServer` (verificado) → `BypassHandler`/`Quest` events.

Cross-ref: `PACKET_ARCHITECTURE.md`, `INCOMING_PACKETS.md`, `OUTGOING_PACKETS.md`, `PACKET_THREADING.md`, `PACKET_SECURITY.md`, `QUESTS/QUEST_PLAYER_NPC_DIALOG.md`
