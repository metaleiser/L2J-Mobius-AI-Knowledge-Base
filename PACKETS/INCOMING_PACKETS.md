# Incoming Packets (Fase 2E)

## 1. Clase base real

**Class:** `ClientPacket`
**Package:** `org.l2jmobius.gameserver.network.clientpackets`
**Path:** `gameserver/network/clientpackets/ClientPacket.java`
**Extends:** `ReadablePacket<GameClient>`
**Implements:** `Runnable`

### Ciclo de vida de ejecución (verificado)
`ReadHandler.completed()` → `packet.init(client, buffer)` → `packet.read()` (llama `readImpl()`) → si OK → `_executor.execute(packet)` → **`Packet.run()`** (Runnable) → `runImpl()`.

- `read()`: `try { readImpl(); } catch → PacketLogger.warning(...)` → `return false` si falla (no se ejecuta).
- `run()`: `try { runImpl(); } catch → PacketLogger.warning(...)` → si falla y `instanceof EnterWorld` → `getClient().closeNow()`.
- **No valida en el parseo**: las validaciones de estado/connection-state ocurrieron en `GamePacketHandler.handle` antes (*no* aquí) comparando `client.getConnectionState()` contra `packetEnum.getConnectionStates()`.

### Acceso a estado
- `protected Player getPlayer()` → `getClient().getPlayer()` (puede retornar null).
- `protected GameClient getClient()` → heredado de `ReadablePacket`.

## 2. Opcode → clase (verificado en `GamePacketHandler`)

- Opcode principal: byte → índice en `ClientPackets.PACKET_ARRAY[packetId]`.
- Ex packets (opcode `0xD0`): byte + short sub-opcode → `ExClientPackets.PACKET_ARRAY[exPacketId]`.
- Bookmark subopcode `0x51`: switch manual → `RequestBookMarkSlotInfo/Save/Modify/Delete/Teleport/Change` (caso único con switch explícito).
- Clase desconocida / fuera de rango → `return null` (descartado silenciosamente; NO hay login de error por opcode desconocido aquí).

## 3. ClientPackets enum (verificado)

**Enum real:** `org.l2jmobius.gameserver.network.ClientPackets`
- Cada entry: referencia a la clase concreta + `connectionStates` (Set<ConnectionState>) + factory `newPacket()`.
- Ejemplos reales: `ATTACK_REQUEST(AttackRequest.class, IN_GAME)`, `REQUEST_BYPASS_TO_SERVER(RequestBypassToServer.class, IN_GAME)`, `ENTER_WORLD(EnterWorld.class, ENTERING, IN_GAME)`.
- Los packets de login/char-select usan estados `CONNECTED/AUTHENTICATED/ENTERING`.

## 4. Ejemplos reales de clientpackets importantes (280 archivos .java)

| Packet | Propósito real |
|---|---|
| `AttackRequest` | ataque físico (→ `PlayerAI`/`AttackStanceTaskManager`, verificado) |
| `RequestActionUse` | skill/item/pickup (→ `Player.useMagic` etc., verificado) |
| `MoveToLocation` | movimiento (→ `Player.moveToLocation`/pathchecker) |
| `RequestBypassToServer` | bypass HTML (quest dialog → `RequestBypassToServer.runImpl` → quest events) |
| `RequestLinkHtml` | abrir HTML en link (verificado: valida rango NPC `Npc.INTERACTION_DISTANCE`) |
| `RequestQuestList` / `RequestQuestAbort` | gestion de quests |

## 5. Estado de verificación
- `ClientPacket base read/run/getPlayer()`: **VERIFIED** (código leído L21–82).
- `GamePacketHandler` dispatch y validación state: **VERIFIED**.
- `ClientPackets` enum y connectionStates: **VERIFIED** (referenciado en handler).
- Conteo de archivos bajo `network/clientpackets`: **280** (**VERIFIED** por recuento recursivo de filesystem, corrección auditoría FASE 2 del 2026-08-24; antes figuraba 293/285).
- Catálogo completo y opcodes: **REQUIRES CODE VERIFICATION**.

Cross-ref: `PACKET_ARCHITECTURE.md`, `PACKET_THREADING.md`, `PACKETS/PACKET_SECURITY.md`, `QUESTS/QUEST_PLAYER_NPC_DIALOG.md`.
