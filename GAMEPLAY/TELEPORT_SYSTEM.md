# TELEPORT SYSTEM

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: GAMEPLAY — Sistema de teletransporte  
**Evidence Date**: 2026-08-27 (Sprint 0.7 — Checkpoint 2)  
**Status**: VERIFIED / CRITICAL DIVERGENCE

---

## ⚠️ CRITICAL DIVERGENCE: SOURCE vs RUNTIME

El sistema de teletransporte es **arquitectónicamente diferente** entre SOURCE y RUNTIME. [FACT]

| Aspecto | SOURCE (upstream) | RUNTIME (deployed) |
|---|---|---|
| **Sistema primario** | `TeleporterData` (XML-based) | Community Board `_bbsteleport` |
| **Data directory** | `data/teleporters/` (198 XMLs) | **NO EXISTE** |
| **NPC handler** | `Teleporter.java`, `DungeonGatekeeper.java` | `HomeBoard.java` |
| **Data source** | XML por NPC ID | `CommunityBoard.ini` + `teleport.sql` |
| **Bypass** | NPC `onBypassFeedback` | `_bbsteleport` bypass |
| **Config** | XML per-NPC (`feeCount`, `feeId`) | `CommunityBoard.ini` |
| **Precio** | Adena o tokens | `CommunityTeleportPrice` (currency) |

**Relación**: STRUCTURAL DIFFERENCE. Sistemas implementados independientemente.  
**Estado**: RUNTIME es autoritativo para comportamiento desplegado. SOURCE es autoritativo para diseño upstream.

---

## 2. SOURCE TELEPORT SYSTEM

### Clase central: `TeleporterData`

| Propiedad | Valor |
|---|---|
| **Class** | `TeleporterData` |
| **Path** | `data/xml/TeleporterData.java` |
| **Tipo** | Singleton |
| **Carga** | `parseDatapackDirectory("data/teleporters", true)` |
| **Estructura** | `_teleporters: Map<Integer, Map<String, TeleportHolder>>` (npcId → listName → holder) |

**Métodos**: `getHolder(int npcId, String listName)` — obtiene lista de teleport para un NPC

### Clases de soporte [FACT]

| Clase | Path | Propósito |
|---|---|---|
| `TeleportHolder` | `entity/teleporter/TeleportHolder.java` | Lista de destinos para un NPC |
| `TeleportLocation` | `entity/teleporter/TeleportLocation.java` | Un destino (x, y, z, name, feeCount, feeId) |
| `Teleporter` | `entity/actor/instance/Teleporter.java` | NPC teleporter (extends Merchant) |
| `DungeonGatekeeper` | `entity/actor/instance/DungeonGatekeeper.java` | Gatekeeper de mazmorra |
| `ClanHallManager` | `entity/actor/instance/ClanHallManager.java` | Teleport de clan hall |
| `FortManager` | `entity/actor/instance/FortManager.java` | Teleport de fuerte |

### Teleport XML Structure [FACT]

```xml
<npc id="30006"> <!-- Roxxy -->
  <teleport type="NORMAL">
    <location name="The Village of Gludin" x="-80684" y="149770" z="-3040" feeCount="18000" />
    <location name="Dark Elven Village" x="9709" y="15566" z="-4568" feeCount="24000" />
    ...
  </teleport>
  <teleport type="NOBLES_TOKEN">
    <location name="Gludin Arena" x="-87328" y="142266" z="-3640" feeId="13722" feeCount="1" />
  </teleport>
  <teleport type="NOBLES_ADENA">
    <location name="Gludin Arena" feeCount="1000" />
  </teleport>
  <teleport type="OTHER">
    <location x="-149406" y="255247" z="-80" /> <!-- Airship dock -->
  </teleport>
</npc>
```

### Teleport Types [FACT]

| Tipo | Costo | Disponibilidad |
|---|---|---|
| `NORMAL` | Adena (`itemId=57`) | Todos los jugadores |
| `NOBLES_TOKEN` | Token de noble (`itemId=13722`) | Nobles |
| `NOBLES_ADENA` | Adena (tarifa reducida) | Nobles |
| `OTHER` | Variable o gratis | Especial (airship, etc.) |

### SOURCE Teleporter Categories [FACT]

| Categoría | NPCs típicos | Propósito |
|---|---|---|
| `town/` | 30006 (Roxxy), 30059, 30080, 30134, etc. | Gatekeepers de ciudad |
| `dungeon/` | 31095-31125+ | Gatekeepers de mazmorras |
| `chamberlain/` | Chamberlains de castillos | Castle teleport |
| `clanhall/` | Messengers | Clan hall teleport |
| `doorman/` | Doormen | Fort teleport |
| `others/` | 30256+, IvoryTower, Ketra, Varka | Especiales |

**Total**: 198 archivos XML en SOURCE. [FACT]

---

## 3. RUNTIME TELEPORT SYSTEM

### Community Board `_bbsteleport` [FACT]

```
HomeBoard.java L267-283:
  command.startsWith("_bbsteleport")
  → teleBuypass = command.replace("_bbsteleport;", "")
  → Check player inventory for COMMUNITYBOARD_CURRENCY
  → If not enough: "Not enough currency!"
  → If COMMUNITY_AVAILABLE_TELEPORTS.get(teleBuypass) != null:
      → player.disableAllSkills()
      → destroyItemByItemId("CB_Teleport", currency, price)
      → setIn7sDungeon(false)
      → setInstanceId(0)
      → teleToLocation(...)
      → ThreadPool.schedule(player::enableAllSkills, 3000)
```

### Teleport Configuration (RUNTIME) [FACT]

**File**: `game/config/Custom/CommunityBoard.ini`

| Config | Default | Descripción |
|---|---|---|
| `CommunityEnableTeleports` | True | Habilitar teleport en CB |
| `CommunityTeleportPrice` | 0 | Precio del teleport (0 = gratis) |
| `CommunityTeleportList` | (lista de destinos) | Destinos disponibles |

**Lista de destinos (RUNTIME)** [FACT]:
```
Giran,82698,148638,-3473; Aden,147450,27064,-2208;
Goddard,147725,-56517,-2780; Rune,43850,-48092,-800;
Dion,15554,143050,-2708; Oren,82752,53581,-1499;
Gludio,-12737,122704,-3120; Schuttgart,87358,-141982,-1341;
Heine,111115,219017,-3547; Gludin,-80616,149952,-3047;
Hunters,116589,76268,-2734; TalkingIsland,-84103,243450,-3732;
Dwarven,115330,-177999,-931; Orc,-44211,-113521,-241;
---

## 4. RUNTIME NPC TELEPORT (SOURCE-compatible)

Aunque RUNTIME no tiene el directorio `teleporters/`, las clases `DungeonGatekeeper.java` y `Teleporter.java` que usan `TeleporterData.getHolder()` **existen compiladas** en el runtime. [FACT]

`DungeonGatekeeper.java L174-187`:
```
doTeleport(player, value):
  holder = TeleporterData.getInstance().getHolder(getId(), TeleportType.OTHER.name())
  if (holder != null):
    holder.doTeleport(player, this, value)
```

Esto sugiere que:
1. Los NPCs `DungeonGatekeeper` y `Teleporter` pueden **aún funcionar** si tienen datos cargados
2. La ausencia del directorio `teleporters/` significa que esos datos no se cargan desde XML
3. **Alternativa posible**: Los datos de teleport podrían venir de la tabla SQL `teleport` en RUNTIME

**UNKNOWN**: No se verificó si existe un loader para la tabla SQL teleport.

---

## 5. OTHER TELEPORT MECHANISMS

| Mecanismo | Implementación | SOURCE/RUNTIME |
|---|---|---|
| **Quest teleport** | `player.teleToLocation(x, y, z)` desde script | Ambos |
| **Instance teleport** | `player.setInstanceId(id)` + `teleToLocation()` | Ambos |
| **Scroll/Escape** | `ItemSkills` con `doCast()` | Ambos |
| **Gatekeeper NPC** | `DungeonGatekeeper.doTeleport()` | SOURCE vía XML / RUNTIME vía SQL(?) |
| **CB Teleport** | `_bbsteleport` bypass | **RUNTIME_ONLY** |
| **Admin teleport** | AdminCommandHandler | Ambos |

---

## 6. UNRESOLVED QUESTIONS

| # | Pregunta | Estado |
|---|---|---|
| 1 | ¿El RUNTIME carga la tabla `teleport.sql` para alimentar `TeleporterData`? | **UNKNOWN** |
| 2 | ¿`DungeonGatekeeper` NPCs funcionan en RUNTIME sin XML? | **UNKNOWN** |
| 3 | ¿Los valores de la lista CB teleport coinciden con los XML de SOURCE? | **UNKNOWN** |
| 4 | ¿Los NPCs `Teleporter` (30006, 30059, etc.) siguen funcionales? | **UNKNOWN** |

## Cross-links

- [PVE_CONTENT_MODEL.md](PVE_CONTENT_MODEL.md) — Zonas por nivel y teleport
- [HUNTING_ZONES.md](HUNTING_ZONES.md) — Acceso a zonas
- [INSTANCE_SYSTEM.md](INSTANCE_SYSTEM.md) — Acceso a instancias
- [../BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md](../BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md) — CB routing
- [../SOURCE_VS_RUNTIME.md](../SOURCE_VS_RUNTIME.md) — Divergencias documentadas
- [../DATABASE/SQL_SCHEMA.md](../DATABASE/SQL_SCHEMA.md) — SQL teleport table

---

**Status**: VERIFIED (ambos sistemas documentados). ⚠️ CRITICAL DIVERGENCE entre SOURCE y RUNTIME.  
**Evidence Date**: 2026-08-27
DarkElven,12428,16551,-4588; Elven,47130,50970,-2999;
Kamael,-116934,46616,368; Blazing Swamp,155310,-16339,-3320;
The Forbidden Gateway,188611,20588,-3696; ...
```

### CB Teleport HTML Pages [FACT]

`game/data/html/CommunityBoard/Custom/gatekeeper/` — 17 HTMLs:
- `main.html` — Página principal
- `Aden.html`, `Dion.html`, `Giran.html`, `Gludin.html`, `Gludio.html`, `Goddard.html`, `Heine.html`, `Hunters.html`, `Oren.html`, `Rune.html`, `Schuttgart.html`
- `Catacomb.html`, `Necropolis.html`, `Seed.html`, `Toi.html`

### SQL Teleport Table [FACT]

```sql
CREATE TABLE teleport (
  Description varchar(75), id mediumint(7), loc_x mediumint(6),
  loc_y mediumint(6), loc_z mediumint(6), price int(10),
  fornoble tinyint(1), itemId smallint(5) DEFAULT '57'
);
```

Contiene rutas como: 'DE Village -> Town of Gludio', 'Gludio -> Elven Village', etc. [FACT]