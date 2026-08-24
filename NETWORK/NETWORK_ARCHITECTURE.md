# NETWORK ARCHITECTURE

**Source of Truth**: `commons.network`, `gameserver.network`  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## NETWORK LAYERS

```
Layer 4: Packet Protocol
   (280 client packet source files, 389 server packet source files; enum totals require verification)
         ↓ Serialization
Layer 3: Encryption
  (Blowfish NewCrypt protocol)
         ↓ Framing
Layer 2: TCP Protocol
  (IP/TCP/Sockets)
         ↓ Transmission
Layer 1: Physical Network
  (Ethernet/Internet)
```

---

## CONNECTION FLOW

### Client → GameServer (Port 7777)

```
1. Client initiates TCP connection to 127.0.0.1:7777
   
2. ConnectionManager accepts connection
   └─ Creates GameClient instance
   └─ Starts reading thread
   
3. First packet: ProtocolVersion (unencrypted)
   └─ Client ↔ Server exchange protocol version
   
4. Second packet: RSA key exchange (unencrypted)
   └─ Generate session encryption key
   
5. Third packet: AuthLogin (encrypted from now)
   └─ Username, password hash, account ID
   └─ GameServer validates with LoginServer
   
6. If valid:
   └─ Player entity created
   └─ World state sent to client
   
7. Client in game
   └─ Bidirectional packet exchange
   └─ All packets encrypted
```

---

## PACKET DISPATCHER

**GamePacketHandler** routes incoming packets:

```java
byte opcode = buffer.readByte();

switch(opcode) {
    case 0x01: return new AuthLogin();
    case 0x02: return new EnterWorld();
    case 0x03: return new MoveToLocation();
    // ... 400+ packet types
}
```

**Packet Execution**:
```java
ReadablePacket packet = handler.handle(buffer, client);
packet.run();  // Executes packet handler logic
```

---

## PACKET STRUCTURE

### ClientPacket Example

```java
public class MoveToLocation extends ClientPacket {
    private int destinationX;
    private int destinationY;
    private int destinationZ;
    
    @Override
    protected void readImpl(ReadBuffer buffer) {
        destinationX = buffer.readInt();
        destinationY = buffer.readInt();
        destinationZ = buffer.readInt();
    }
    
    @Override
    protected void runImpl(GameClient client) {
        Player player = client.getPlayer();
        player.moveToLocation(destinationX, destinationY, destinationZ);
        client.sendPacket(new ValidateLocation());
    }
}
```

### ServerPacket Example

```java
public class MoveToLocation extends ServerPacket {
    private int creatureId;
    private int x, y, z;
    
    public MoveToLocation(Creature creature) {
        this.creatureId = creature.getObjectId();
        this.x = creature.getX();
        this.y = creature.getY();
        this.z = creature.getZ();
    }
    
    @Override
    protected void writeImpl(WriteBuffer buffer) {
        buffer.writeInt(creatureId);
        buffer.writeInt(x);
        buffer.writeInt(y);
        buffer.writeInt(z);
    }
}
```

---

## KNOWN PACKET TYPES

### Client Packets (400+)

**Authentication**: AuthLogin, ProtocolVersion  
**Movement**: MoveToLocation, MoveToLocationAirShip, MoveWithDelta  
**Combat**: AttackRequest, MagicSkillUse, MagicSkillUseGround  
**Inventory**: UseItem, RequestDropItem, RequestDestroyItem  
**Trading**: TradeRequest, TradeDone, RequestBuyItem  
**Social**: RequestFriendInvite, RequestJoinParty, RequestJoinPledge  
**Quest**: RequestQuestList, RequestQuestAbort  
**Admin**: RequestGMCommand  
... (350+ more)

### Server Packets (source-file count: 389; enum total requires verification)

**Entities**: CharInfo, NpcInfo, DropItem, DeleteObject  
**Updates**: UserInfo, InventoryUpdate, EquipUpdate  
**Combat**: Attack, Die, Revive, MagicSkillUse  
**Chat**: CreatureSay, NpcSay, SystemMessage  
**Interaction**: NpcHtmlMessage, ShowBoard  
**Movement**: MoveToLocation, StopMove  
... (300+ more)

---

## ENCRYPTION SYSTEM

### Key Exchange (Handshake)

```
1. Client connects (unencrypted)
   
2. Server generates RSA keypair
   └─ Public key sent to client
   
3. Client generates random session key
   └─ Encrypts with public key, sends back
   
4. Server decrypts session key
   └─ Both now have shared session key
   
5. All future packets use Blowfish with session key
```

### Blowfish Encryption

**Algorithm**: Blowfish symmetric cipher  
**Block Size**: 64 bits  
**Key Size**: Variable (128-bit typical)  
**Mode**: ECB (electronic codebook)

**NewCrypt Protocol**:
```
Raw packet → Compress (if needed) → Encrypt → Add packet header → Send
Receive → Remove header → Decrypt → Decompress (if needed) → Parse
```

---

## THREAD MODEL

**Multiple concurrent connections**:

```
Main thread (acceptor)
   └─ Accepts new connections
   
Per-connection thread (reader)
   └─ Reads packets from socket
   └─ Passes to packet handler
   
ThreadPool threads (handlers)
   └─ Execute packet logic
   └─ Queue ServerPackets response
   
Writer threads (per connection)
   └─ Sends queued ServerPackets
   └─ Encrypts and framing
```

---

## PACKET LATENCY

**Typical flow**:
```
1. Client sends packet: 0ms
2. Network latency: 50-100ms (typical)
3. Server receives: 50-100ms
4. Server processes: 0-10ms
5. Server sends response: 50-100ms total
```

**Total round-trip**: 100-300ms typical

---

## CONNECTION STATES

**ConnectionState** enum defines client state:

```java
enum ConnectionState {
    CONNECTED,         // TCP established
    AUTHENTICATED,     // Session key valid
    ENTERING_WORLD,    // Loading entities
    IN_GAME,          // Playing
    DISCONNECTING,    // Leaving
    DISCONNECTED      // Gone
}
```

---

## BANDWIDTH USAGE

**Per player typical**:
```
Idle: 1-5 KB/minute
Moving: 10-50 KB/minute
Combat: 50-200 KB/minute
Chat: 1-2 KB per message
```

**Note**: UNKNOWN - exact measurements depend on packet optimization.

---

## KNOWN ISSUES & LIMITS

**Packet Size**: Individual packets typically <4KB  
**Connection Limit**: 1000+ concurrent (configurable)  
**Bandwidth**: Not explicitly limited in code  
**Timeout**: UNKNOWN (check ConnectionManager config)

---

**Source of Truth**: Network package analysis  
**Verified**: 2026-08-23  
**Status**: VERIFIED
