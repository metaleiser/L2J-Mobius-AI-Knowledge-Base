# Database Configuration & Connections

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Fase**: 2F - Database  
**Source of Truth**: `commons.config.DatabaseConfig`, `commons.database.DatabaseFactory` y `commons.util.ConfigReader`  
**Verificado**: 2026-08-23  
**Status**: VERIFIED para la configuración descrita

## 1. Archivo y claves reales

`DatabaseConfig.load()` lee `./config/Database.ini` desde el directorio de trabajo mediante `ConfigReader`. Las claves y defaults observados son:

| Clave | Campo | Default real |
|---|---|---|
| `Driver` | `DATABASE_DRIVER` | `com.mysql.cj.jdbc.Driver` |
| `URL` | `DATABASE_URL` | `jdbc:mysql://localhost/l2jmobius` |
| `Login` | `DATABASE_LOGIN` | `root` |
| `Password` | `DATABASE_PASSWORD` | vacío |
| `MaximumDatabaseConnections` | `DATABASE_MAX_CONNECTIONS` | `10` |
| `TestDatabaseConnections` | `DATABASE_TEST_CONNECTIONS` | `false` |
| `BackupDatabase` | `BACKUP_DATABASE` | `false` |
| `MySqlBinLocation` | `MYSQL_BIN_PATH` | `C:/xampp/mysql/bin/` |
| `BackupPath` | `BACKUP_PATH` | `../backup/` |
| `BackupDays` | `BACKUP_DAYS` | `30` |

No se verificó otro archivo `.properties` para esta configuración. Las referencias anteriores a `database.properties` quedan corregidas: no describen el loader real de esta build.

## 2. Inicialización del pool

`DatabaseFactory.init()` carga `DatabaseConfig`, crea una `HikariConfig` y asigna driver, URL, usuario y contraseña. El pool se crea como `HikariDataSource` con nombre `L2JMobiusPool`. Si ya existe y no está cerrado, la inicialización se omite con warning.

Parámetros verificados:

| Parámetro | Valor o cálculo |
|---|---|
| maximum pool size | `clamp(configured, 4, 1000)` |
| minimum idle | `max(maximum / 10, 2)` |
| connection timeout | `60000 ms` |
| idle timeout | `300000 ms` |
| max lifetime | `600000 ms` |
| leak detection | `600000 ms` |
| validation timeout | `5000 ms` |
| initialization fail timeout | `-1` |
| register MBeans | `true` |

El valor `100` que aparece en comentarios del código no es el default ni el límite: el default de configuración es `10` y el método aplica el rango `4..1000`.

## 3. Obtención y cierre

`DatabaseFactory.getConnection()` delega en `DATABASE_POOL.getConnection()`. Los usos revisados emplean try-with-resources; cerrar la conexión entregada por Hikari la devuelve al pool. El comportamiento global de `autoCommit` debe considerarse **UNKNOWN / REQUIRES CODE VERIFICATION** si se requiere una garantía formal: los métodos revisados de una sola sentencia no lo cambian y las transacciones compuestas deben establecerlo explícitamente.

## 4. Pruebas de conexión y backup

Con `TestDatabaseConnections=true`, `DatabaseFactory` intenta abrir el máximo de conexiones y puede ajustar el pool si no todas tienen éxito. Con `false`, ejecuta la prueba de una conexión. El resultado y manejo de errores se registran mediante `java.util.logging`.

`DatabaseConfig` expone opciones de backup y `DatabaseBackup` existe en `commons.database`, pero frecuencia efectiva, retención, restauración y uso de esas opciones: **UNKNOWN / REQUIRES CODE VERIFICATION**.

## Ver también

- [DATABASE_ARCHITECTURE.md](DATABASE_ARCHITECTURE.md)
- [DATABASE_TRANSACTIONS.md](DATABASE_TRANSACTIONS.md)
- [SQL_SCHEMA.md](SQL_SCHEMA.md)
- [ID_MANAGEMENT.md](ID_MANAGEMENT.md)
