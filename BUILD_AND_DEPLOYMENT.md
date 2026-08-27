# BUILD AND DEPLOYMENT

**Document Type**: SOURCE_ENTRY  
**Source of Truth**: \uild.xml\ in SERVER_SOURCE root (\UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive\)  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## 1. BUILD TOOL

**Apache Ant**, NOT Gradle. No \uild.gradle\ or \gradlew\ exist in the source tree.  
**Required Ant version**: 1.8.2 or higher (checked by \checkRequirements\ target).  
**Required Java**: Java 25 (source and target level).

---

## 2. VERIFIED BUILD TARGETS

| Target | Description | Evidence (build.xml) |
|--------|-------------|---------------------|
| \checkRequirements\ | Validate Ant ≥1.8.2 and Java 25 | \<fail>\ with \nt.version\ / \java.version\ condition |
| \init\ | Create \uild/bin\, \uild/dist\ directories | \<mkdir>\ tasks |
| \compile\ | Compile \java/org/l2jmobius/**/*.java\ to \uild/bin/\ | \<javac>\ with source/target=25, UTF-8, debug |
| \jar\ | Package LoginServer.jar, GameServer.jar, DatabaseInstaller.jar | \<jar>\ with \Main-Class\ manifest entries |
| \dding-core\ | ZIP \uild/dist\ → \L2J_Mobius_CT_2.6_HighFive.zip\ | \<zip>\ task |
| \dding-datapack\ | Add data/scripts/config to same ZIP | \<zip update>\ task |
| \dding-readme\ | Add \eadme.txt\ to ZIP | \<zip update>\ task |
| \cleanup\ (default) | Remove \uild/bin\, \uild/dist\ | \<delete>\ tasks |

---

## 3. BUILD FLOW

\\\
checkRequirements → init → compile → jar → adding-core → adding-datapack → adding-readme → cleanup
\\\

**Note**: The default target is \cleanup\, so a bare \nt\ command builds everything then deletes the artifacts.  
To produce deployable JARs: \nt jar\ (stops after the \jar\ target).  
The datapack (XML, scripts, config) is distributed alongside JARs in the ZIP, not embedded inside them.

---

## 4. ARTIFACTS / OUTPUTS

| Artifact | Output Path | Main-Class |
|----------|-------------|------------|
| LoginServer.jar | \uild/dist/libs/LoginServer.jar\ | \org.l2jmobius.loginserver.LoginServer\ |
| GameServer.jar | \uild/dist/libs/GameServer.jar\ | \org.l2jmobius.gameserver.GameServer\ |
| DatabaseInstaller.jar | \uild/dist/databaseinstaller/\ | \org.l2jmobius.tools.DatabaseInstaller\ |
| Third-party libs | Copied to \dist/libs/*.jar\ (mariadb-java-client, etc.) | — |
| Distribution ZIP | \L2J_Mobius_CT_2.6_HighFive.zip\ | Core JARs + datapack + readme |

JAR manifests include \Main-Class\, \Build-Date\, \Class-Path\, and git build info.

---

## 5. DEPLOYMENT STRUCTURE

After unpacking the distribution ZIP (or from existing runtime):

\\\
dist/
├── game/
│   ├── GameServer.jar
│   ├── config/          ← *.ini and *.xml config files
│   ├── data/            ← XML data (skills, items, NPCs, etc.)
│   └── log/             ← server logs
├── login/
│   ├── LoginServer.jar
│   └── config/
└── libs/                ← third-party JARs
\\\

**Eclipse launchers** exist at \launcher/Gameserver.launch\ and \launcher/Loginserver.launch\, configured for the Eclipse workspace path. Working directory for GameServer launcher: \\/dist/game/\.

---

## 6. EVIDENCE REFERENCES

- **Build file**: \UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive/build.xml\
- **Launcher configs**: \UPSTREAM/.../launcher/Gameserver.launch\, \launcher/Loginserver.launch\
- **JVM options** (from Eclipse launchers): \-server -Dfile.encoding=UTF-8 -Djava.util.logging.manager=org.l2jmobius.log.ServerLogManager -Dorg.slf4j.simpleLogger.log.com.zaxxer.hikari=warn -XX:+UseZGC\
- **Heap recommendation**: \-Xms2G -Xmx4G\ for 2000-3000 concurrent players (UNKNOWN — verify against production deployments)
- **Database installer**: \java -cp build/dist/libs/*:java/ org.l2jmobius.tools.DatabaseInstaller\

---

## 7. RELEVANT CROSS-LINKS

- Start-up sequence: [GAMESERVER_ARCHITECTURE.md](GAMESERVER_ARCHITECTURE.md)
- Configuration files: [CONFIGURATION/CONFIGURATION_SYSTEM.md](CONFIGURATION/CONFIGURATION_SYSTEM.md)
- Database setup: [DATABASE/DATABASE_ARCHITECTURE.md](DATABASE/DATABASE_ARCHITECTURE.md)

---

**Source of Truth**: build.xml, launcher configurations  
**Verified**: 2026-08-23  
**Status**: VERIFIED
