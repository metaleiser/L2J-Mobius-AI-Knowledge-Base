# CONFIGURATION SYSTEM

**Source of Truth**: `java/org/l2jmobius/gameserver/config/` + `dist/game/config/` (rutas relativas a la raíz del servidor)  
**Verified**: 2026-08-23  
**Actualizado**: 2026-08-24 (auditoría FASE 2: ejemplos corregidos al mecanismo INI real)  
**Status**: PARTIAL (inventario de configs VERIFIED contra código; ejemplos corregidos; hot-reload/validation UNKNOWN)

---

## CORE CONFIG FILES (16 load calls)

Located: `dist/game/config/` (files loaded at runtime)

| Config File | Java Class | Purpose |
|-------------|-----------|---------|
| `*.ini` under `dist/game/config/` | `GeneralConfig`, `ServerConfig`, `PlayerConfig`, `RatesConfig`, `PvpConfig`, `NpcConfig`, `FeatureConfig`, `OlympiadConfig`, `GeoEngineConfig`, `IdManagerConfig`, `DevelopmentConfig`, `FloodProtectorConfig`, `GrandBossConfig`, `UndergroundColiseumConfig`, `ConquerableHallSiegeConfig` | Main configuration classes loaded by `ConfigLoader` |
| `GraciaSeedsConfig` | `GraciaSeeds.ini` | Gracia Seeds settings |

---

## CUSTOM CONFIG FILES (44+)

Located: `dist/game/config/custom/`

| Config | Purpose |
|--------|---------|
| AllowedPlayerRacesConfig | Restrict character races |
| AutoPlayConfig | AFK auto-play |
| AutoPotionsConfig | Auto-consume potions |
| BankingConfig | Banking system |
| BossAnnouncementsConfig | Announce boss kills |
| CancelReturnConfig | Cancel return feature |
| CaptchaConfig | Anti-bot captchas |
| ChampionMonstersConfig | Champion monster system |
| ChatModerationConfig | Chat filtering |
| ClassBalanceConfig | Class balancing tweaks |
| CommunityBoardConfig | Forum system |
| CustomMailManagerConfig | Custom mail features |
| DelevelManagerConfig | Level reduction |
| DualboxCheckConfig | Multi-account detection |
| FactionSystemConfig | Faction systems |
| FakePlayersConfig | Fake player NPCs |
| FindPvpConfig | PvP finder |
| FreeMountsConfig | Free mounts |
| HellboundStatusConfig | Hellbound progression |
| MerchantZeroSellPriceConfig | Merchant selling |
| MultilingualSupportConfig | Multi-language |
| NoblessMasterConfig | Noble system |
| NpcStatMultipliersConfig | NPC stat adjustments |
| OfflinePlayConfig | Offline play |
| OfflineTradeConfig | Offline trading |
| OnlineInfoConfig | Online information |
| PasswordChangeConfig | Password security |
| PremiumSystemConfig | Premium accounts |
| PrivateStoreRangeConfig | Private shop range |
| PvpAnnounceConfig | PvP announcements |
| PvpRewardItemConfig | PvP rewards |
| PvpTitleColorConfig | PvP titles |
| RandomSpawnsConfig | Random NPC spawns |
| RebirthConfig | Rebirth system |
| SchemeBufferConfig | Buff selling |
| ScreenWelcomeMessageConfig | Welcome messages |
| SellBuffsConfig | Buff trading |
| ServerTimeConfig | Game time settings |
| StartingLocationConfig | New player start |
| StartingTitleConfig | Starting titles |
| TransmogConfig | Item transmog |
| WalkerBotProtectionConfig | Bot detection |
| WarehouseSortingConfig | Warehouse auto-sort |
| WeddingConfig | Wedding system |

---

## LOADING MECHANISM

### On GameServer Startup

```
1. ConfigLoader.init()
   └─ Sequentially load the configuration classes in ConfigLoader
   └─ Each class reads its .ini file through ConfigReader
   └─ Values stored in static fields
   
2. Separately load 44+ custom configs
   └─ Only if corresponding feature enabled
   
3. All values accessible as:
   └─ GeneralConfig.SOME_SETTING
   └─ ServerConfig.PORT
   └─ RatesConfig.RATE_XP
   └─ etc.
```

### Hot Reload

**UNKNOWN**: Whether configs can be reloaded without server restart

### Validation

**On load**: 
- Type checking (int, boolean, string)
- Range validation (e.g., rates 0.1-10.0)
- UNKNOWN - exact validation rules

---

## IMPORTANT SETTINGS

> **Corrección auditoría FASE 2 (2026-08-24)**: los ejemplos anteriores usaban claves inventadas en formato `.properties`. Las claves reales de este build viven en archivos `.ini` con formato `Clave = valor`. Solo se transcriben a continuación claves verificadas directamente contra los archivos reales.

### Server Network (Server.ini)
```ini
# Dónde conecta el gameserver (las IPs externas van en ipconfig.xml)
LoginHost = 127.0.0.1
LoginPort = 9014
GameserverHostname = 0.0.0.0
GameserverPort = 7777
```

### Rates (Rates.ini)
```ini
RateXp = 1
RateSp = 1
RatePartyXp = 1
RateExtractable = 1
```

### PvP / Feature
Las claves reales están en `dist/game/config/PVP.ini` y `dist/game/config/Feature.ini`; no se transcriben aquí sin verificación previa (**REQUIRES CODE VERIFICATION** por clave).

---

## STRUCTURE OF A CONFIG FILE

**Mecanismo real**: cada clase `XxxConfig` define una constante con la ruta de su `.ini` y un método estático `load()` que lee mediante `ConfigReader` (`commons/util/ConfigReader.java`, wrapper sobre `java.util.Properties`) aplicando defaults hardcodeados por clave.

Evidencia de ruta (`gameserver/config/ServerConfig.java`, línea 67):
```java
private static final String SERVER_CONFIG_FILE = "./config/Server.ini";
```

Ejemplo real completo (`commons/config/DatabaseConfig.java`, verificado):
```java
public class DatabaseConfig
{
	private static final String DATABASE_CONFIG_FILE = "./config/Database.ini";

	public static String DATABASE_DRIVER;
	public static String DATABASE_URL;
	// ... más campos públicos estáticos

	public static void load()
	{
		final ConfigReader config = new ConfigReader(DATABASE_CONFIG_FILE);
		DATABASE_DRIVER = config.getString("Driver", "com.mysql.cj.jdbc.Driver");
		DATABASE_URL = config.getString("URL", "jdbc:mysql://localhost/l2jmobius");
		DATABASE_LOGIN = config.getString("Login", "root");
		DATABASE_PASSWORD = config.getString("Password", "");
		DATABASE_MAX_CONNECTIONS = config.getInt("MaximumDatabaseConnections", 10);
		// ... resto de claves con el mismo patrón
	}
}
```

---

## OVERRIDE MECHANISM

Jerarquía demostrada en código:
1. Defaults hardcodeados como segundo argumento de cada llamada `config.getXxx(clave, default)`.
2. El valor presente en el `.ini` correspondiente sobreescribe ese default.

Otros posibles mecanismos de override (variables de entorno, recarga en caliente): **UNKNOWN / REQUIRES CODE VERIFICATION** — sin evidencia en el código auditado.

---

## EXAMPLES OF CONFIG IMPACT

(Ilustración conceptual; las claves citadas son las reales.)

### Si se sube la experiencia
Modificar `RateXp` en `Rates.ini`.

### Si se reduce el drop
Ajustar la clave de drop correspondiente en `Rates.ini`.

---

## COMMON CONFIGURATIONS

Los perfiles "Development"/"Production" anteriores citaban claves inventadas y fueron retirados. Para valores reales, consultar los `.ini` de `dist/game/config/`: cada clave documenta su `Default` en comentario.

---

## FILE LOCATIONS

**Core configs**: `dist/game/config/*.ini`
**Custom configs**: `dist/game/config/custom/*.ini` (44 archivos)
**Database config**: `./config/Database.ini` (cargado por `DatabaseConfig`)
**Thread config**: `./config/Threads.ini` (cargado por `ThreadConfig`) — corrección 2026-08-24: el archivo real es `Threads.ini`

---

## ADDING NEW CONFIGURATION

Pasos coherentes con el patrón custom existente de este build:
1. Crear clase `NewFeatureConfig` en `java/org/l2jmobius/gameserver/config/custom/` con método `load()` que use `ConfigReader` (no se extiende `ConfigReader`; se instancia).
2. Crear `dist/game/config/custom/new_feature.ini` con las claves.
3. Registrar la clase/carga en `ConfigLoader` (los customs se cargan condicionalmente según feature).
4. Acceder vía campos estáticos `NewFeatureConfig.SETTING_NAME`.

---

**Source of Truth**: `gameserver/config/`, `commons/config/`, `commons/util/ConfigReader.java`, `dist/game/config/`
**Verified**: 2026-08-23
**Actualizado**: 2026-08-24 (auditoría FASE 2)
**Status**: PARTIAL (inventario VERIFIED; ejemplos corregidos a INI; hot-reload/validation UNKNOWN)
