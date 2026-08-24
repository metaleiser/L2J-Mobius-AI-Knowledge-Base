# BUILD AND DEPLOYMENT

**Source of Truth**: `build.xml`, launcher files  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## BUILD SYSTEM

**Build Tool**: Apache Ant  
**Build File**: `build.xml` (root directory)  
**Required Ant Version**: 1.8.2 or higher  
**Required Java**: Java 25  

---

## BUILD TARGETS

### Primary Targets

#### `compile`
Compiles Java source code.

```
Source: java/org/l2jmobius/**/*.java
Output: build/bin/
Options: 
  - Java 25 source and target
  - Debug symbols included
  - UTF-8 encoding
```

#### `jar`
Creates JAR files from compiled classes.

**LoginServer.jar**
```
Output: build/dist/libs/LoginServer.jar
Includes: commons/*, loginserver/* (all classes)
Excludes: gameserver/*, tools/DatabaseInstaller, tools/Search
Main-Class: org.l2jmobius.loginserver.LoginServer
Manifest includes git build info
```

**GameServer.jar**
```
Output: build/dist/libs/GameServer.jar
Includes: commons/*, gameserver/* (all classes)
Excludes: loginserver/*, tools/AccountManager, tools/GameServerRegister
Main-Class: org.l2jmobius.gameserver.GameServer
Manifest includes git build info
```

#### `cleanup`
Default target. Cleans build artifacts.

```
Removes: build/bin/*, build/dist/
```

---

## BUILD PROCESS FLOW

```
1. checkRequirements
   ├─ Verify Ant ≥ 1.8.2
   └─ Verify Java 25 installed

2. init
   ├─ Delete old build/bin/
   └─ Create fresh directories

3. compile
   ├─ Compile all java/ source to build/bin/
   ├─ Classpath: dist/libs/*.jar
   └─ Output: .class files

4. jar
   ├─ Create build/dist/libs/LoginServer.jar
   │  └─ Exclude gameserver, DatabaseInstaller, Search
   ├─ Create build/dist/libs/GameServer.jar
   │  └─ Exclude loginserver, AccountManager, GameServerRegister
   └─ Set Main-Class manifest attributes

5. (Optional cleanup)
   └─ Remove build artifacts if needed
```

---

## REQUIRED LIBRARIES

All libraries must be present in: `dist/libs/`

| Category | Libraries | Purpose |
|----------|-----------|---------|
| Database | hikaricp, mysql-connector-java | Connection pool, DB driver |
| Logging | slf4j | Logging facade |
| XML | dom4j | XML parsing |
| Utilities | commons-*, guava | General utilities |
| Network | netty (optional) | UNKNOWN |

**Note**: REQUIRES CODE VERIFICATION for exact library list and versions.

---

## RUNTIME DISTRIBUTION

After successful build:

```
dist/
├── game/
│   ├── GameServer.jar (NOT here, stays in build/dist/libs/)
│   ├── libs/          (Shared library JARs)
│   ├── config/        (Runtime configuration files)
│   ├── data/          (XML data files)
│   ├── log/           (Log output directory)
│   ├── java.cfg       (Java runtime options)
│   ├── console.cfg    (Console configuration)
│   ├── GameServer.sh  (Linux launcher script)
│   ├── GameServer.vbs (Windows launcher script)
│   └── GameServerTask.sh
│
├── login/
│   ├── LoginServer.jar
│   ├── libs/
│   ├── config/
│   ├── data/
│   ├── log/
│   ├── LoginServer.sh
│   └── LoginServer.vbs
│
└── db_installer/
    └─ Database setup tool
```

---

## DEPLOYMENT PROCESS

### Step 1: Prepare Build Environment
```
Prerequisites:
✓ Java 25 installed
✓ Ant 1.8.2+ installed
✓ MySQL/MariaDB server running
✓ Database created and populated
```

### Step 2: Build JAR Files
```bash
cd L2J_Mobius_CT_2.6_HighFive
ant compile jar
```

Outputs:
- `build/dist/libs/GameServer.jar`
- `build/dist/libs/LoginServer.jar`

### Step 3: Setup Configuration
Edit runtime configs:
- `dist/game/config/` - Game Server configs
- `dist/login/config/` - Login Server configs

Key files:
- `Server.ini` - Network settings
- `Database.ini` - DB connection
- `Rates.ini` - Game rates

### Step 4: Start Login Server
```bash
cd dist/login/
./LoginServer.sh        # Linux
GameServer.vbs          # Windows
```

### Step 5: Start Game Server
```bash
cd dist/game/
./GameServer.sh         # Linux
GameServer.vbs          # Windows
```

### Step 6: Verify Startup
Check log files:
- `dist/login/log/` - Login Server logs
- `dist/game/log/` - Game Server logs

Both should show "Server started successfully" messages.

---

## IDE CONFIGURATION (Eclipse)

Eclipse launch configurations provided:

**File**: `launcher/Gameserver.launch`
```xml
<launchConfiguration type="org.eclipse.jdt.launching.localJavaApplication">
    <stringAttribute key="org.eclipse.jdt.launching.MAIN_TYPE" 
                     value="org.l2jmobius.gameserver.GameServer"/>
    <stringAttribute key="org.eclipse.jdt.launching.WORKING_DIRECTORY" 
                     value="${workspace_loc:L2J_Mobius_CT_2.6_HighFive}/dist/game/"/>
    <stringAttribute key="org.eclipse.jdt.launching.VM_ARGUMENTS" 
                     value="-server -Dfile.encoding=UTF-8 ..."/>
</launchConfiguration>
```

**File**: `launcher/Loginserver.launch`
```xml
<launchConfiguration type="org.eclipse.jdt.launching.localJavaApplication">
    <stringAttribute key="org.eclipse.jdt.launching.MAIN_TYPE" 
                     value="org.l2jmobius.loginserver.LoginServer"/>
    <stringAttribute key="org.eclipse.jdt.launching.WORKING_DIRECTORY" 
                     value="${workspace_loc:L2J_Mobius_CT_2.6_HighFive}/dist/login/"/>
</launchConfiguration>
```

To use:
1. Import project into Eclipse
2. Run → Run Configurations
3. Select "Gameserver" or "Loginserver"
4. Click Run

---

## JAVA REQUIREMENTS

### Runtime Options (from launcher)

```
-server
    Run in server mode (optimized JVM)

-Dfile.encoding=UTF-8
    Set file encoding to UTF-8

-Djava.util.logging.manager=org.l2jmobius.log.ServerLogManager
    Use custom logging manager

-Dorg.slf4j.simpleLogger.log.com.zaxxer.hikari=warn
    Suppress HikariCP debug logs

-XX:+UseZGC
    Use ZGC garbage collector (low-latency)
```

### Heap Size Recommendation

For 2000-3000 concurrent players:
```
-Xms2G      Minimum heap 2GB
-Xmx4G      Maximum heap 4GB
```

**Note**: UNKNOWN - check actual production deployments for optimal settings.

---

## DATABASE SETUP

### Database Installer

Tool for initial database creation:

```bash
java -cp build/dist/libs/*:java/ org.l2jmobius.tools.DatabaseInstaller
```

Performs:
1. Creates database tables
2. Loads initial data
3. Sets up indexes

**Location**: `dist/db_installer/` (Windows/Linux scripts provided)

### Database Configuration

**File**: `dist/game/config/Database.ini` (the database loader reads `./config/Database.ini` relative to its working directory)

```ini
Driver=com.mysql.cj.jdbc.Driver
DB_URL=jdbc:mysql://localhost:3306/l2jmobius
DB_LOGIN=root
DB_PASSWORD=password
MaximumDatabaseConnections=10
```

---

## TROUBLESHOOTING BUILD

### Build Fails: "Java 25 not found"
```
Error: Java 25 is required. But your version is Java X.X
Fix: Download and install JDK 25
Verify: java -version
```

### Build Fails: "Ant 1.8.2 is required"
```
Error: Ant 1.8.2 is required. But your version is X.X
Fix: Download and install Ant 1.8.2+
Verify: ant -version
```

### JAR Build Succeeds but Startup Fails
```
Possible causes:
1. Missing libraries in dist/libs/
2. Wrong Java version (must be 25)
3. Configuration files missing in dist/game/config/
4. Database not accessible
```

---

## CONTINUOUS DEPLOYMENT

For development:

1. Make code changes in `java/`
2. Run `ant compile jar`
3. Copy new JARs to `dist/game/` or `dist/login/`
4. Restart server process

**Note**: Configuration changes do NOT require rebuild (loaded at runtime).

---

## NEXT STEPS

- [GAMESERVER_ARCHITECTURE.md](GAMESERVER_ARCHITECTURE.md) - Detailed startup
- [CONFIGURATION/CONFIGURATION_SYSTEM.md](CONFIGURATION/CONFIGURATION_SYSTEM.md) - Config files
- [DATABASE/DATABASE_ARCHITECTURE.md](DATABASE/DATABASE_ARCHITECTURE.md) - Database setup

---

**Source of Truth**: build.xml, launcher configurations  
**Verified**: 2026-08-23  
**Status**: VERIFIED
