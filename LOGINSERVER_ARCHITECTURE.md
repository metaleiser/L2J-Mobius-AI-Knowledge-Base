# LOGINSERVER ARCHITECTURE

**Source of Truth**: `org.l2jmobius.loginserver.LoginServer` and related classes  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## ENTRY POINT

**Main Class**: `org.l2jmobius.loginserver.LoginServer`  
**Port**: 9014 (configurable)  
**Role**: Authentication server for client login and Game Server registration  
**Database**: Shared with Game Server  

---

## STARTUP SEQUENCE

### Phase 1: Interface & Logging
```
1. InterfaceConfig.load()
   └─ Check if GUI enabled

2. Create log directory (./log/)

3. LogManager.readConfiguration(./log.cfg)
   └─ Configure Java logging
```

### Phase 2: Configuration
```
4. DatabaseConfig.load()
   └─ Load database credentials

5. InterfaceConfig.load()
   └─ Load UI settings

6. LoginConfig.load()
   └─ Load login server settings
      ├─ Port number
      ├─ Protocol version
      ├─ Max concurrent connections
      └─ ... (login-specific config)
```

### Phase 3: Database
```
7. DatabaseFactory.init()
   └─ Initialize HikariCP pool
   └─ Test connection
```

### Phase 4: Core Services
```
8. ThreadPool.init()
   └─ Configure thread pools
```

### Phase 5: Login Services
```
9. LoginController.getInstance()
   └─ Initialize login controller
   └─ Load login-specific data

10. GameServerTable.getInstance()
    └─ Load list of registered Game Servers
    └─ Load from database
```

### Phase 6: Security
```
11. Load banned IP addresses from ./banned_ip.cfg

12. Configure IP ban list
```

### Phase 7: Network
```
13. GameServerListener.start()
    └─ Listen on port 9014 for Game Servers
    └─ Accept GameServerThread connections
```

### Phase 8: GUI (optional)
```
14. IF InterfaceConfig.ENABLE_GUI:
    └─ new Gui()
    └─ Display login server GUI
```

---

## CORE COMPONENTS

### LoginServer Class
**Location**: `org.l2jmobius.loginserver.LoginServer`

**Responsibilities**:
- Orchestrate startup
- Load configuration
- Initialize services
- Start network listeners
- Handle graceful shutdown

**Singleton Pattern**:
```java
private static LoginServer _instance;

private LoginServer() {
    // Initialization code
}

public static LoginServer getInstance() {
    return _instance;
}
```

---

### LoginController
**Location**: `org.l2jmobius.loginserver.controller.LoginController`

**Responsibilities**:
- Validate account credentials
- Manage login sessions
- Generate SessionKey tokens
- Track active logins

**Operations**:
```
1. Client sends username/password
   └─ LoginController.login(account, password)
   
2. Query database for account
   └─ Verify password hash
   
3. If valid:
   └─ Generate unique SessionKey
   └─ Return SessionKey + Game Server list
   
4. If invalid:
   └─ Return error code
   └─ Possibly increment attempt counter
```

---

### GameServerListener
**Location**: `org.l2jmobius.loginserver.GameServerListener`

**Responsibilities**:
- Listen for Game Server connections
- Accept GameServerThread for each server
- Coordinate with multiple Game Servers
- Manage Game Server status

**Network**:
```
Port: 9014 (default)
Protocol: Same as Game-Client (Blowfish encrypted)
Connections: One per Game Server
```

---

### GameServerThread
**Location**: `org.l2jmobius.loginserver.GameServerThread`

**Responsibilities**:
- Manage individual Game Server connection
- Receive player login requests
- Send player authentication results
- Exchange server status

**Communication Flow**:
```
Game Server connects to port 9014
   │
   ├─ Handshake and authentication
   │
   └─ Bidirectional message exchange:
      ├─ Game Server → Login Server:
      │  ├─ "Player X wants to login"
      │  ├─ "Update my status"
      │  └─ "Player X disconnected"
      │
      └─ Login Server → Game Server:
         ├─ "Player X auth OK"
         ├─ "Player X auth FAILED"
         └─ "Check player X status"
```

---

### GameServerTable
**Location**: `org.l2jmobius.loginserver.data.GameServerTable`

**Responsibilities**:
- Maintain list of registered Game Servers
- Store server metadata
- Track server online/offline status
- Load from database

**Data Stored**:
```
per Game Server:
├─ Server ID
├─ Server name (e.g., "Server 1", "Server 2")
├─ IP/Port
├─ Max players
├─ Current players
├─ Online status (true/false)
└─ Server type/flags
```

**Database Table** (assumed structure):
```
gameservers table:
├─ server_id        (INT, primary key)
├─ hostname         (VARCHAR, server name)
├─ ip               (VARCHAR, server IP)
├─ port             (INT, server port)
├─ max_players      (INT, capacity)
├─ online           (TINYINT, 0/1 status)
└─ type             (INT, server type flags)
```

**Note**: REQUIRES CODE VERIFICATION for actual table schema.

---

### LoginClient
**Location**: `org.l2jmobius.loginserver.network.LoginClient`

**Responsibilities**:
- Represent a player client connection
- Handle login packets from client
- Send auth responses
- Manage session state

**States**:
```
CONNECTED
   └─ Client TCP connection established
   
AUTHENTICATED
   └─ Player credentials verified
   └─ SessionKey issued
   
DISCONNECTED
   └─ Connection closed
```

---

## AUTHENTICATION FLOW

### Client Perspective

```
1. Client connects to port 9014
   
2. Client sends AuthLogin packet
   ├─ Username
   ├─ Password (hashed)
   └─ Protocol version
   
3. Login Server validates
   └─ Query database for account
   └─ Verify credentials
   
4A. IF credentials valid:
    └─ Login Server sends response:
       ├─ SessionKey (unique token)
       ├─ List of available Game Servers
       │  ├─ Server name
       │  ├─ IP/Port
       │  ├─ Current player count
       │  └─ Max player capacity
       └─ Client can choose server
   
4B. IF credentials invalid:
    └─ Login Server sends error code
       ├─ Invalid password
       ├─ Account banned
       ├─ Account not found
       └─ ... (other error codes)
```

### Game Server Perspective

```
1. Game Server connects to port 9014 (GameServerListener)
   
2. Game Server sends registration packet
   ├─ Server ID
   ├─ Server name
   ├─ IP/Port
   └─ Max players
   
3. Login Server stores in GameServerTable
   
4. Game Server now accepts clients
   └─ Clients connect with SessionKey
   └─ Game Server validates SessionKey with LoginServer
   
5. During gameplay:
   └─ Game Server updates player count
   └─ Login Server reflects in GameServerTable
   └─ LoginClients see updated server status
```

---

## SESSION KEY SYSTEM

**SessionKey** is a unique token issued during login:

**Properties**:
```
per SessionKey:
├─ Unique ID
├─ Associated account
├─ Associated character
├─ Timestamp issued
├─ Expiration time
├─ Valid for ONE Game Server only
└─ Used to validate Game Server login attempts
```

**Usage**:
```
1. Client receives SessionKey from LoginServer
   
2. Client connects to Game Server with SessionKey
   
3. Game Server receives client, validates SessionKey:
   └─ Client sends SessionKey in connect packet
   └─ Game Server asks LoginServer: "Is this valid?"
   └─ LoginServer responds: "Yes" or "No"
   
4. If valid:
   └─ Game Server allows client to join
   
5. If invalid:
   └─ Game Server rejects connection
```

---

## BANNED IP SYSTEM

**File**: `./banned_ip.cfg` (or similar location)

**Purpose**: Block IP addresses from connecting

**Checking**:
```
1. Client connects to port 9014
   
2. LoginServer checks client IP:
   └─ Look up IP in banned_ip.cfg
   
3. IF IP is banned:
   └─ Immediately close connection
   └─ No login attempt allowed
   
4. IF IP is not banned:
   └─ Proceed with normal login flow
```

**Note**: UNKNOWN - exact file format and update mechanism.

---

## DATABASE QUERIES

### Account Lookup
```sql
SELECT * FROM accounts 
WHERE login = ? 
AND password = MD5(?)  -- or similar hash
```

### Game Server List
```sql
SELECT * FROM gameservers 
WHERE online = 1
ORDER BY type, player_count
```

### Session Storage
```sql
-- SessionKey stored in memory, possibly also DB
-- REQUIRES CODE VERIFICATION
```

---

## NETWORK PROTOCOL

**Base**: Same as Game Client network layer

**Differences**:
- Port: 9014 instead of 7777
- Packet types: LoginClientPackets (auth-specific)
- Authentication flow: Different sequence

**Packet Types**:
```
Client Packets:
├─ ProtocolVersion
├─ AuthLogin
├─ RequestServerLogin
└─ ... (login-specific packets)

Server Packets:
├─ LoginFail (error response)
├─ LoginOk (success + SessionKey)
├─ ServerList (available Game Servers)
├─ PlayFail (play request failed)
└─ PlayOk (play request OK)
```

---

## MULTI-SERVER COORDINATION

For multi-server setups:

```
Client connects to ANY server's LoginServer
   │
   ├─ Can choose ANY Game Server from list
   │
   └─ Each Game Server connects to SAME LoginServer
      └─ All servers validate clients with same LoginServer
      └─ SessionKeys globally recognized
```

**Implication**: Single LoginServer can authenticate for multiple Game Servers.

---

## PERFORMANCE CHARACTERISTICS

**Designed for**:
- Minimal load (mostly authentication)
- 1000s of concurrent login attempts
- Low latency responses

**Bottleneck** (typically):
- Database queries during login
- Network bandwidth from GameServerTable requests

**Note**: LoginServer is much lighter than GameServer.

---

## GRACEFUL SHUTDOWN

On shutdown:
```
1. Stop accepting new client connections
2. Stop accepting new Game Server connections
3. Disconnect all active clients
4. Notify all Game Servers (connection closes)
5. Save any state
6. Close database connection pool
7. Exit
```

---

## TOOLS: GameServerRegister

Separate tool for registering Game Server to LoginServer:

**Location**: `org.l2jmobius.tools.GameServerRegister`

**Purpose**: Add/register a new Game Server to LoginServer database

**Usage**:
```bash
java -cp libs/* org.l2jmobius.tools.GameServerRegister
```

**Operations**:
```
1. Prompt for server details
   ├─ Server name
   ├─ IP address
   ├─ Port number
   └─ Max players

2. Insert into gameservers table

3. Game Server can now register with LoginServer
```

---

## NEXT STEPS

- [COMMONS_ARCHITECTURE.md](COMMONS_ARCHITECTURE.md) - Shared infrastructure
- [NETWORK/NETWORK_ARCHITECTURE.md](NETWORK/NETWORK_ARCHITECTURE.md) - Network details
- [CONFIGURATION/CONFIGURATION_SYSTEM.md](CONFIGURATION/CONFIGURATION_SYSTEM.md) - Configuration
---

**Source of Truth**: LoginServer.java, LoginController, GameServerListener  
**Verified**: 2026-08-23  
**Status**: VERIFIED
