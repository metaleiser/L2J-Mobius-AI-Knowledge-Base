# Packet Player Flow (Fase 2E)

## 1. Flujo de conexión real (verificado)

```
CLIENTE (socket NIO)
   → aceptación por commons NetworkServer
   → GameClient (org.l2jmobius.gameserver.network)       [extiende Client<Connection<GameClient>>]
   → ReadHandler.completed()  (thread NIO: I/O + decrypt)
      → GamePacketHandler.handle(buffer, client)   (commons PacketHandler<GameClient>)
         → ClientPackets.PACKET_ARRAY[opcode]  → state check (ConnectionState) → newPacket()
         → packet.init(client, buffer); packet.read()  (readImpl)
         → PacketExecutor.execute(packet)   (ThreadPool — NO es el client thread)
            → ClientPacket.run()  (Runnable)  → runImpl()
               → getPlayer()  [getClient().getPlayer()] → acción sobre Player
```

### `GameClient` (verificado)
**Class:** `GameClient` | **Path:** `gameserver/network/GameClient.java` | **Extends:** `Client<Connection<GameClient>>`
- Campos reales: `_floodProtectors` (por-sesión), `_playerLock` (ReentrantLock), `_connectionState` (ConnectionState, default CONNECTED), `_encryption` (Encryption), `_accountName`, `_sessionKey`, `_player` (Player, null hasta login exitoso).
- `public Player getPlayer()` → retorna `_player`.
- `ConnectionState`: enum `CONNECTED, DISCONNECTED, CLOSING, AUTHENTICATED, ENTERING, IN_GAME` (verificado).

### Player attachment
- `_player` se asigna en `GameClient` tras login/exitosa (verificado en `ClientPackets.ENTER_WORLD(0x11, EnterWorld.class, ENTERING)` — `EnterWorld.runImpl` produce el Player y lo enlaza al client).
- Packets con estado `IN_GAME` requieren `player != null`; si es null el packet típicamente retorna temprano (ej.: `AttackRequest`: `if (player == null) return;`).

## 2. Estados de conexión (verificado)
- `CONNECTED` — socket conectado, antes del handshake (p.ej. `PROTOCOL_VERSION(0x0E)` aceptado en CONNECTED).
- `AUTHENTICATED` — sesión válida de login server.
- `ENTERING` — transición (p.ej `ENTER_WORLD` aceptado solo en ENTERING).
- `IN_GAME` — jugador en el mundo; la gran mayoría de los packets requieren este estado.

## 3. Estado de verificación
- `GameClient` campos/extends/getPlayer/getConnectionState: **VERIFIED** (L56–691).
- `ConnectionState`: **VERIFIED** (6 valores: CONNECTED/DISCONNECTED/CLOSING/AUTHENTICATED/ENTERING/IN_GAME).
- `ClientPacket implements Runnable`, `ReadHandler` ejecuta vía `PacketExecutor`: **VERIFIED**.
- Flujo completo de la handshake de login (session key, GG, etc.): parcialmente verificado — parte marcada `REQUIRES CODE VERIFICATION`.

Cross-ref: `PACKET_DISPATCH.md`, `PACKET_SECURITY.md`, `SYSTEMS/PLAYER_SYSTEM.md`
