# CONFIGURATION SYSTEM

**Document Type**: SOURCE_ENTRY  
**Source of Truth**: \java/org/l2jmobius/gameserver/config/\ + \java/org/l2jmobius/commons/config/\ (config classes) and \dist/game/config/\ (runtime files)  
**Verified**: 2026-08-23  
**Status**: PARTIAL (inventory VERIFIED against code; hot-reload/validation UNKNOWN)

---

## 1. SOURCE-OF-TRUTH PACKAGES / CLASSES

### Core config (gameserver)
\org.l2jmobius.gameserver.config\:
- \GeneralConfig\, \ServerConfig\, \PlayerConfig\, \RatesConfig\, \PvpConfig\, \NpcConfig\
- \FeatureConfig\, \OlympiadConfig\, \GeoEngineConfig\, \IdManagerConfig\, \DevelopmentConfig\
- \FloodProtectorConfig\, \GrandBossConfig\, \UndergroundColiseumConfig\, \ConquerableHallSiegeConfig\
- \GraciaSeedsConfig\

### Commons config (shared)
\org.l2jmobius.commons.config\:
- \DatabaseConfig\, \InterfaceConfig\, \ThreadConfig\

### Custom config (gameserver)
\org.l2jmobius.gameserver.config.custom\ — 44+ classes (SchemeBufferConfig, WeddingConfig, TransmogConfig, etc.)

---

## 2. ARCHITECTURE SUMMARY

Each \XxxConfig\ class follows a uniform pattern:
- Defines a \private static final String\ path constant (e.g. \\"./config/Server.ini\"\)
- Provides a static \load()\ method
- Uses \ConfigReader\ (\commons.util.ConfigReader\, wrapper over \java.util.Properties\) to read \.ini\ files
- Stores values in public static fields with hardcoded defaults as fallback
- \ConfigLoader.init()\ in GameServer calls all core config \load()\ methods sequentially

---

## 3. CONFIGURATION / LOADING RELATIONSHIPS

\\\
GameServer.init()
  └─ ConfigLoader.init()
       ├─ Core configs: 16 classes, each loads its *.ini via ConfigReader
       └─ Custom configs: 44+ classes loaded conditionally by feature flag

ConfigReader (commons.util)
  └─ Reads *.ini files
  └─ getString(key, default), getInt(key, default), getBoolean(key, default)
  └─ Wraps java.util.Properties
\\\

### File locations (runtime working directory = \dist/game/\)
- Core configs: \config/*.ini\ (e.g. \Server.ini\, \Rates.ini\, \Feature.ini\)
- Custom configs: \config/custom/*.ini\ (44 files)
- Database config: \./config/Database.ini\ (read by DatabaseConfig)
- Thread config: \./config/Threads.ini\ (read by ThreadConfig)

---

## 4. USEFUL EVIDENCE REFERENCES

- Path constant example: \ServerConfig.java:67\ → \\"./config/Server.ini\"\
- Loading pattern: \DatabaseConfig.load()\ — creates \ConfigReader\, calls \getString()\ / \getInt()\ with defaults
- Default override hierarchy: hardcoded default (2nd arg) → ini file value → runtime value
- Hot-reload / env override: **UNKNOWN** — no evidence in audited code

---

## 5. RELEVANT CROSS-LINKS

- [COMMONS_ARCHITECTURE.md](../COMMONS_ARCHITECTURE.md) — commons package (DatabaseConfig, ThreadConfig)
- [DATABASE/DATABASE_ARCHITECTURE.md](../DATABASE/DATABASE_ARCHITECTURE.md) — DB connection flow
- [THREADING/THREADING_ARCHITECTURE.md](../THREADING/THREADING_ARCHITECTURE.md) — ThreadConfig usage
- [BUILD_AND_DEPLOYMENT.md](../BUILD_AND_DEPLOYMENT.md) — deployment paths for config files

---

**Source of Truth**: gameserver/config/, commons/config/, commons/util/ConfigReader.java, dist/game/config/  
**Verified**: 2026-08-23  
**Status**: PARTIAL

