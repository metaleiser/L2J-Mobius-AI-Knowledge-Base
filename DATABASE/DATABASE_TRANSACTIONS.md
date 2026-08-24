# Database Transactions

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Fase**: 2F - Database  
**Source of Truth**: usos JDBC de `DatabaseFactory.getConnection()` en entidades, tablas y managers  
**Verificado**: 2026-08-23  
**Status**: VERIFIED para los patrones citados

## 1. Recursos JDBC

El patrón dominante es try-with-resources:

```java
try (Connection con = DatabaseFactory.getConnection();
     PreparedStatement ps = con.prepareStatement(SQL)) {
    ps.executeUpdate();
}
```

Se verificó en métodos de `Player`, `Quest`, `Inventory` y tablas SQL. El cierre ocurre en orden inverso y la conexión se entrega de nuevo al pool HikariCP.

## 2. Operaciones simples y por lotes

Las operaciones de una sola sentencia usan `executeQuery()` o `executeUpdate()` sin crear una transacción manual en el método revisado. El valor efectivo de `autoCommit` depende de la configuración JDBC/pool y no debe afirmarse como contrato universal sin una comprobación adicional; por eso queda **UNKNOWN / REQUIRES CODE VERIFICATION**.

Hay operaciones por lotes con `addBatch()`/`executeBatch()` en `Player`, `OfflinePlayTable`, `CastleManorManager`, variables y otros componentes. El uso de batch no demuestra por sí solo atomicidad transaccional.

## 3. Transacciones explícitas

Se verificó el patrón de `setAutoCommit(false)`, varias escrituras, `commit()` y `rollback()` en flujos de persistencia de `Player`, incluyendo `storeCharBase()` y métodos relacionados. El alcance exacto de cada grupo de statements debe leerse por método; no se generaliza a todas las operaciones de guardado.

No se encontró evidencia documentada en el código revisado de `Savepoint` o transacciones anidadas. Soporte de nested transactions, retry automático y aislamiento configurable: **UNKNOWN / REQUIRES CODE VERIFICATION**.

## 4. Errores

El código registra excepciones mediante `java.util.logging`, normalmente con mensajes contextuales. Algunos métodos controlan rollback en el `catch`; otros registran el error de una operación aislada. No se verificó un mecanismo común de reintento.

## Ver también

- [DATABASE_CONFIGURATION.md](DATABASE_CONFIGURATION.md)
- [DATABASE_ARCHITECTURE.md](DATABASE_ARCHITECTURE.md)
- [SQL_SCHEMA.md](SQL_SCHEMA.md)
