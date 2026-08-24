# Packet Architecture (Fase 2E)

> **Corrección auditoría FASE 2 (2026-08-24)**: conteos de archivos verificados contra código (clientpackets = 280, serverpackets = 389); estructura de `commons/network` corregida según listado real; referencias a clases inexistentes eliminadas (`NetworkThread`, `SendablePacket`).

## Capas reales del stack de red

El sistema de packets de Mobius se divide en **dos capas físicas**:

### Capa 1 — `commons/network` (shared library)
Base genérica de red reutilizable. NO contiene opcodes de juego.

```
commons/network/
├── packet/
│   ├── PacketHandler              — registro/despacho de packets
│   ├── ReadablePacket             — base para parsing entrante
│   ├── SimpleReadablePacket       — variante simple de lectura
│   ├── WritablePacket             — base para escritura saliente
│   ├── SimpleWritablePacket       — variante simple de escritura
│   ├── PacketExecutor             — ejecución de packets
│   └── PacketThreadFactory        — fábrica de threads de packet
├── handler/
│   ├── ReadHandler                — lectura asíncrona
│   └── WriteHandler               — escritura asíncrona
├── buffer/                        — gestión de buffers de E/S
├── pool/                          — pooling
└── (raíz) Client, Connection, ConnectionConfig, ConnectionManager
```

> NOTA: la existencia y ubicación de estas clases está verificada por listado del paquete. El comportamiento interno detallado de cada clase: **REQUIRES CODE VERIFICATION**. Este build NO debe describirse como "NIO select-loop": no existe ninguna clase `NetworkThread` en el código.

### Capa 2 — `gameserver/network` (específico juego)
Sobre la capa 1, contiene todos los packets reales.

```
gameserver/network/
├── GamePacketHandler                — tabla/despacho de opcodes de juego
├── ClientPackets                    — enum de los clientpackets
├── ExClientPackets                  — enum de los Ex clientpackets (0xD0)
├── ServerPackets                    — enum de los serverpackets
├── ConnectionState                  — enum: BEFORE_LOGIN, IN_GAME, etc.
├── GameClient                       — sesión cliente-jugador
├── Encryption                       — cifrado de packets
├── clientpackets/   (280 archivos .java)
└── serverpackets/    (389 archivos .java)
```

> NOTA: `FloodProtectors` NO vive en `gameserver/network`; está en `gameserver/util/FloodProtectors.java`.

## Clase base real (verificada)

| Role | Class | Package | Extends | Implements |
|---|---|---|---|---|
| Base incoming | `ClientPacket` | `...gameserver.network.clientpackets` | `extends ReadablePacket<GameClient>` | `Runnable` |
| Base outgoing | `ServerPacket` | `...gameserver.network.serverpackets` | `extends WritablePacket<GameClient>` | — |
| Dispatcher | `GamePacketHandler` | `...gameserver.network` | — | `Runnable` |
| Session | `GameClient` | `...gameserver.network` | — | — |

> Declaraciones verificadas en código: `ClientPacket.java:32`, `ServerPacket.java:35`.
> El Javadoc antiguo puede hablar de `ServerBasePacket`; la clase real es `ServerPacket`.

> NOTA: `ClientPacket implements Runnable` → se ejecuta en thread pool de packets, NO en el hilo de E/S directamente.

## Registro / Despacho (verificado)

### GameServer
- `GamePacketHandler(GameClient client)` constructor registra **todos** los clientpackets.
- Cada `ClientPacket subclass` se registra via `ClientPackets` enum (enum → clase + opcode).
- `ClientPackets` mapa **estático precalculado** (`CLIENT_PACKET_MAX` + lookup por opcode).
- Packets Ex (0xD0): registro análogo vía `ExClientPackets`.

### LoginServer
- `LoginPacketHandler` (no confundir con `GamePacketHandler`); registro separado y **diferente**.

## Estado de verificación
- `commons/network` estructura de paquete: **VERIFIED** (listado real 2026-08-24).
- `GamePacketHandler` tabla opcodes y constructor: **VERIFIED**.
- `ClientPackets` / `ExClientPackets` / `ServerPackets` enums: **VERIFIED** (existencia confirmada).
- Conteo de clases: clientpackets = **280**, serverpackets = **389** (**VERIFIED** por recuento recursivo de filesystem 2026-08-24).
- Total de entradas de enum y sus opcodes: **REQUIRES CODE VERIFICATION**.
- Versionamiento multi-cliente (diferencias cliente por versión): **UNKNOWN**.

Ver también: `PACKETS/PACKET_DISPATCH.md`, `PACKETS/PACKET_PLAYER_FLOW.md`

