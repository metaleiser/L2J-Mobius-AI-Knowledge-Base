# Packet Security (Fase 2E)

## 1. Validaciones reales verificadas

### a) Estado de conexión (pre-despacho)
- `GamePacketHandler.handle`: `if (!packetEnum.getConnectionStates().contains(client.getConnectionState())) return null;`
  - El packet **ni siquiera se instancia** si el estado no coincide (verificado en `GamePacketHandler.java:78`).
  - Ejemplo: `ENTER_WORLD` solo se acepta en estado `ENTERING`.

### b) Flood protection (intra-packet, real)
```java
// AttackRequest.runImpl (verificado, L58–61):
if (!getClient().getFloodProtectors().canPerformPlayerAction()) { return; }
```
- `GameClient._floodProtectors` (FloodProtectors, 16 protecciones: useItem, rollDice, itemPetSummon, heroVoice, globalChat, subClass, dropItem, serverBypass, multiSell, transaction, manufacture, sendMail, characterSelect, itemAuction, playerAction).
- **Cada packet elige su flood protector** (no hay protección global).

### c) Validaciones dentro del packet (ejemplo AttackRequest)
```java
final Player player = getPlayer(); if (player == null) return;              // L64
if (player.isPlayable() && player.isInBoat()) { ...return; }               // L70 (no attack en barco)
// bot penalty check (S10) → ActionFailed                                   // L77
if (!target.isTargetable() && !player.isGM()) return;                     // L108
if ((target.getInstanceId() != player.getInstanceId()) && ...) return;     // L116 (instancia)
if (!target.isVisibleFor(player)) return;                                  // L123 (GM bypass)
```

### d) Bypass security (RequestBypassToServer, verificado)
- `_link.contains("..")` → `PacketLogger.warning` + return (path traversal) (L53–57).
- `player.validateHtmlAction("link " + _link)` (L59) — `htmlObjectId == -1` → return.
- Rango `Npc.INTERACTION_DISTANCE` (L66).
- Mensajes `packet` y `Disconnection.of(getClient(), player).storeAndDeleteWith(LeaveWorld.STATIC_PACKET)` (L90) — empty command → kick + delete.

### e) Tamaño máximo de packet (commons, verificado en ReadHandler)
- `MAX_PACKET_SIZE = 65533`; `dataSize > MAX_PACKET_SIZE` → `client.disconnect()` inmediato (L116–119).

### f) Opcode desconocido / fuera de rango
- En `GamePacketHandler`: rango inválido → `return null` (descartado silenciosamente); no hay log ni kick por opcode desconocido aquí (el logging DEBUG ocurre solo si `DevelopmentConfig.DEBUG_CLIENT_PACKETS`).

## 2. Estado de verificación
- FloodProtectors campos + canX(): **VERIFIED**.
- `AttackRequest` validaciones: **VERIFIED** (L56–146).
- `RequestBypassToServer` validaciones: **VERIFIED**.
- `ReadHandler` MAX_PACKET_SIZE + state check: **VERIFIED**.
- Lista completa de cuál fraud-manager aplica a cada packet: **REQUIRES CODE VERIFICATION**.

Cross-ref: `PACKET_DISPATCH.md`, `PACKET_ARCHITECTURE.md`, `SYSTEMS/PLAYER_SYSTEM.md`
