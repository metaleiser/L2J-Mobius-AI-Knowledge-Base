# MANAGERS INDEX

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Source of Truth**: `java/org/l2jmobius/gameserver/managers/` (relativa a la raíz del servidor)  
**Verified**: 2026-08-23  
**Actualizado**: 2026-08-24 (inventario regenerado desde código en auditoría FASE 2)  
**Status**: VERIFIED (recuento y firmas verificados contra código)

---

## HISTORIAL DE CORRECCIONES (histórico — no usar como conteo vigente)

- Fase 1 decía **52 managers** e incluía `World`, `ClanTable` y `ClanHallTable`: incorrecto.
  - `World` NO es manager: utilidad estática en `entity/World.java` (sin `getInstance()`).
  - `ClanTable` / `ClanHallTable` viven en `data/sql/` como cargadores de tabla.
- Auditoría 2026-08-23 detectó 58 archivos pero dejó tablas internas inconsistentes (mezclaba 48/53).
- **2026-08-24**: catálogo regenerado por script contra el código. Cifras vigentes: **58 clases = 52 con firma + 6 sin firma**.

## UBICACIONES (verificado)

- `managers/` raíz: **53 clases**
- Subpaquete `managers/games/`: **5 clases** → `BlockCheckerManager`, `KrateisCubeManager`, `LotteryManager`, `MonsterRaceManager`, `UndergroundColiseumManager`

## TABLA A — CON FIRMA `public static … getInstance()` (52)

> Detección por patrón sobre cada archivo (2026-08-24). La descripción se hereda de observaciones previas; «pendiente» = sin descripción verificada.

| # | Manager | Ubicación | Responsabilidad |
|---|---------|-----------|-----------------|
| 1 | AirShipManager | raíz | Dirige naves aéreas (airships) |
| 2 | AntiFeedManager | raíz | Control anti-feed PvP |
| 3 | BoatManager | raíz | Dirige barcos (boats) |
| 4 | CaptchaManager | raíz | Captchas anti-bot |
| 5 | CastleManager | raíz | Castillos y su estado |
| 6 | CastleManorManager | raíz | Manor (aldeas de castillo) |
| 7 | CHSiegeManager | raíz | Asedios de Clan Hall |
| 8 | ClanHallAuctionManager | raíz | Subastas de Clan Hall |
| 9 | CoupleManager | raíz | Matrimonios entre jugadores |
| 10 | CursedWeaponsManager | raíz | Armas malditas |
| 11 | CustomMailManager | raíz | Correo personalizado |
| 12 | DailyResetManager | raíz | Reinicios diarios |
| 13 | DayNightSpawnManager | raíz | Spawns de día/noche |
| 14 | DimensionalRiftManager | raíz | Rifts dimensionales |
| 15 | DuelManager | raíz | Duelos 1vs1 |
| 16 | EventDropManager | raíz | Drops de eventos |
| 17 | FakePlayerChatManager | raíz | Chat de fake players |
| 18 | FishingChampionshipManager | raíz | Campeonato de pesca |
| 19 | FortManager | raíz | Fuertes |
| 20 | FortSiegeManager | raíz | Asedios de fuertes |
| 21 | GlobalVariablesManager | raíz | Variables globales de servidor |
| 22 | GrandBossManager | raíz | Grand bosses (epic) |
| 23 | HandysBlockCheckerManager | raíz | Minijuego Block Checker |
| 24 | IdManager | raíz | IDs globales de objetos |
| 25 | InstanceManager | raíz | Instancias/dungeons |
| 26 | ItemAuctionManager | raíz | Subastas de items |
| 27 | ItemsOnGroundManager | raíz | Items en el suelo |
| 28 | KrateisCubeManager | games/ | (descripción pendiente) |
| 29 | LotteryManager | games/ | (descripción pendiente) |
| 30 | MailManager | raíz | Correo del jugador |
| 31 | MercTicketManager | raíz | Tickets de mercenarios |
| 32 | MonsterRaceManager | games/ | (descripción pendiente) |
| 33 | PcCafePointsManager | raíz | Puntos PC Cafe |
| 34 | PetitionManager | raíz | Peticiones de soporte |
| 35 | PrecautionaryRestartManager | raíz | Reinicios preventivos |
| 36 | PremiumManager | raíz | Cuentas premium |
| 37 | PunishmentManager | raíz | Sanciones, baneos, mutes |
| 38 | RaidBossPointsManager | raíz | Puntos por raid bosses |
| 39 | RaidBossSpawnManager | raíz | Spawns de raid bosses |
| 40 | RebirthManager | raíz | Sistema de rebirth |
| 41 | RecipeManager | raíz | Recetas de crafting |
| 42 | ScriptManager | raíz | Carga de scripts |
| 43 | SeedOfDestructionManager | raíz | Evento Seed of Destruction |
| 44 | SeedOfInfinityManager | raíz | Evento Seed of Infinity |
| 45 | SellBuffsManager | raíz | Venta de buffs entre jugadores |
| 46 | ServerRestartManager | raíz | Reinicios programados |
| 47 | SiegeManager | raíz | Asedios de castillos |
| 48 | TerritoryWarManager | raíz | Guerra de territorios |
| 49 | UndergroundColiseumManager | games/ | (descripción pendiente) |
| 50 | WalkingManager | raíz | Rutas de NPCs |
| 51 | ZoneBuildManager | raíz | Construcción de zonas |
| 52 | ZoneManager | raíz | Zonas (carga y consulta) |

## TABLA B — SIN FIRMA `public static … getInstance()` (6)

| # | Clase | Ubicación | Observación |
|---|-------|-----------|-------------|
| 53 | BlockCheckerManager | games/ | Funcionalidad de Block Checker; sin firma de singleton detectada |
| 54 | DatabaseIdManager | raíz | Limpieza/revisión de IDs en BD; NO extiende `IdManager` (verificado) |
| 55 | FortSiegeGuardManager | raíz | Guardias de asedio de fuerte |
| 56 | ItemManager | raíz | Fábrica/destruye `Item`; métodos estáticos; usa `IdManager.getInstance()` |
| 57 | SiegeGuardManager | raíz | Guardias de castillo (hired mercs) |
| 58 | TownManager | raíz | Utilidades de towns/castillos |

_NOTAS_: `ItemManager` usa `IdManager.getInstance()` → los objetos creados quedan registrados en `World` vía su `Item` (no a través del `ItemManager`). `DatabaseIdManager` es complementario de `IdManager` pero NO lo extiende (verificado).

## PATRÓN GENERAL

- La mayoría de managers exponen `public static Xxx getInstance()`.
- **NO afirmar que los 58 son singletons**: solo los 52 de Tabla A siguen el patrón; los 6 de Tabla B usan métodos estáticos u otros patrones.
- Inventario completo regenerado desde código (2026-08-24); sin huecos pendientes.

## DEPENDENCIAS TÍPICAS (verificadas en imports/código, auditoría previa)

- `SiegeManager` depende de `CastleManager`, `MercTicketManager` (vía SiegeGuardManager).
- `TownManager` depende de `CastleManager`, `MapRegionData`, `ZoneManager`.
- `ItemManager` depende de `IdManager`, `EventDispatcher`.
- `World` (entity/) es usado por casi todo, pero no es un manager.

## INICIALIZACIÓN

Ver [GAMESERVER_ARCHITECTURE.md](../GAMESERVER_ARCHITECTURE.md). `World.init()` se invoca en la fase de World System (NO es un `getInstance()`).

---

**Source of Truth**: `managers/` + `entity/World.java` + `data/sql/ClanTable.java`  
**Status**: VERIFIED (recuento/firmas 2026-08-24; descripciones heredadas donde indicado)  
**Verified**: 2026-08-23 · **Actualizado**: 2026-08-24