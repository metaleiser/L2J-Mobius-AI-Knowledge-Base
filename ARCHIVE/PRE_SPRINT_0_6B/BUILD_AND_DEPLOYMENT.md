# BUILD AND DEPLOYMENT

**Source of Truth**: `build.xml`, launcher files  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## BUILD SYSTEM

**Build Tool**: Apache Ant  (`build.xml` en la raíz del SERVER_SOURCE `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive`)
**Build File**: `build.xml` (root directory)
**Required Ant Version**: 1.8.2 or higher
**Required Java**: Java 25

> **Nota (KB v2.0)**: el sistema de build de H5 es **Apache Ant, NO Gradle**. No existen `build.gradle`/`gradlew` en el source. El build debe ejecutarse sobre el **SERVER_SOURCE** (UPSTREAM), no sobre el runtime desplegado.


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

Flujo real de `build.xml` (`default="cleanup"`, cadena de dependencias):

```
checkRequirements
   ↓
init
   ↓
compile
   ↓
jar                      → genera LoginServer.jar, GameServer.jar, DatabaseInstaller.jar
   ↓
adding-core              → añade ${build.dist} al ZIP de distribución
   ↓
adding-datapack          → añade el datapack (data/scripts/config) al mismo ZIP
   ↓
adding-readme            → añade readme.txt al ZIP
   ↓
cleanup
```

### Artefactos que genera el build

| Artefacto | Detalle |
|---|---|
| `LoginServer.jar` | `build/dist/libs/` — Main-Class `org.l2jmobius.loginserver.LoginServer` |
| `GameServer.jar` | `build/dist/libs/` — Main-Class `org.l2jmobius.gameserver.GameServer` |
| `DatabaseInstaller.jar` | `build/dist/databaseinstaller/` — instalador de BD |
| Líbrerías de terceros | copiadas a `dist/libs/*.jar` (p. ej. `mariadb-java-client`) |
| ZIP de distribución | `L2J_Mobius_CT_2.6_HighFive.zip` (core + datapack + readme) |

### Detalle por target

1. **checkRequirements** — verifica Ant ≥1.8.2 y Java requerido.
2. **init** — crea directorios de salida (`build/bin`, `build/dist`).
3. **compile** — compila `java/org/l2jmobius/**/*.java` a `build/bin/`.
4. **jar** — empaqueta los JARs (ver artefactos) con manifiesto (Main-Class, Build-Date, Class-Path).
5. **adding-core** — `zip` de `${build.dist}` → `L2J_Mobius_CT_2.6_HighFive.zip`.
6. **adding-datapack** — `zip update` del datapack (data/scripts/config) al mismo ZIP.
7. **adding-readme** — añade `readme.txt` al ZIP.
8. **cleanup** — limpia la carpeta `build/`.

> **El datapack/data se distribuye aparte del core compilado**: el código Java compilado va en los JARs; los datos (XML, scripts, configs de datos) se empaquetan por separado al ZIP, no dentro de los JARs.


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

### SERVER_RUNTIME (desplegado) vs SERVER_SOURCE (build) — KB v2.0

El `SERVER_RUNTIME` del workspace (`L2J_Mobius_CT_2.6_HighFive/`) es el **contenido de `dist/` + JARs compilados + datapack + geodata**, es decir, el **resultado descomprimido de un build/distribución**:

- Es el servidor que **realmente se ejecuta y se usa para jugar/probar** (build **26/05/2024**).
- **Sin `.git`**, sin `java/org/l2jmobius` (no es source).
- Contiene `game/`, `login/`, `db_installer/`, `libs/`, `backup/`, `images/`, `MultisellCreator*`.
- El source (`UPSTREAM/...`) es donde **se compila**; el runtime es donde **se ejecuta**.

> Relación correcta: **SOURCE (build Ant) → ZIP de distribución → runtime desplegado**. Ver `00_PROJECT/PROJECT_CONTEXT.md` y `VERSIONING/UPSTREAM_BASELINE.md`.

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
