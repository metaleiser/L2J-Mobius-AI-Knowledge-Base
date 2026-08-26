# SPAWN QUERY GUIDE

**Proyecto**: L2J Mobius CT 2.6 HighFive
**Capa**: Mundo — cómo localizar y verificar spawns de NPCs/mobs
**Verified**: 2026-08-26
**Status**: VERIFIED (layouts y método comprobados con Q00003)

> Objetivo: responder "¿dónde y cuántas veces spawnea el NPC/mob X?" sin asumir que un único archivo es exhaustivo.

---

## 1. Dos layouts de spawns

| Raíz | Layout | Detalle |
|---|---|---|
| **SERVER_RUNTIME** `L2J_Mobius_CT_2.6_HighFive/game/data/spawns/` | **Consolidado** | Un único archivo: `HighFiveSpawns.xml` (`<spawn name="HighFiveSpawns">` con todos los `<npc .../>`). |
| **SERVER_SOURCE** `UPSTREAM/.../dist/game/data/spawns/` | **Por territorio + celda** | Carpetas por territorio (`Aden`, `DarkElfTerritory`, `Others`, …) y archivos nombrados `<mapX>_<mapY>.xml` (celda del grid; ver [../SYSTEMS/WORLD_SYSTEM.md](../SYSTEMS/WORLD_SYSTEM.md): TILE_SIZE=32768, SHIFT_BY=11). |

- RUNTIME es un **flatten** de SOURCE: mismos spawns, distinta organización.
- Los comentarios `<!-- Mob Name -->` en SOURCE ayudan a identificar; RUNTIME no los conserva.
- El atributo `<spawn name="...">` de SOURCE es el nombre de la celda/grupo (p.ej. `18_19`), **no** un nombre de zona jugable.

## 2. Cómo buscar por NPC/mob ID

```powershell
# RUNTIME (archivo único)
Select-String -Path 'game\data\spawns\HighFiveSpawns.xml' -Pattern 'npc id="<ID>"' -AllMatches

# SOURCE (árbol completo)
Get-ChildItem 'dist\game\data\spawns' -Recurse -Filter '*.xml' |
  Select-String -Pattern 'npc id="<ID>"'
```

- Contar ocurrencias → nº total de spawns.
- Agrupar por `Path` → en qué archivo(s)/celda(s) vive.
- **Comparar SOURCE vs RUNTIME**: los conteos deben coincidir (verificado p.ej. con Q00003).

## 3. Formato de línea y campos

```xml
<npc id="20031" x="-50853" y="45399" z="-5400" heading="47094" respawnDelay="60" />
```

| Campo | Significado |
|---|---|
| `id` | NPC/mob template ID (cruza con `stats/npcs/*.xml`) |
| `x,y,z` | coordenadas de spawn (mundo) |
| `heading` | orientación (0–65535) |
| `respawnDelay` | segundos hasta respawn |
| `count` (opcional) | nº deinstancias del punto (variantes) |
| `periodOfDay` (opcional) | spawns día/noche |

## 4. Identificar la celda SOURCE correcta

1. Busca el ID en todo el árbol SOURCE (no asumas la carpeta).
2. El nombre del archivo (`18_19`, `20_18`, …) = celda del grid donde caen esas coords.
3. Ejemplo verificado (Q00003): mobs 20031–20057 → `spawns/Others/18_19.xml`; NPC 30141 Talloth → `spawns/Others/20_18.xml`.
4. El territorio de la carpeta (p.ej. `DarkElfTerritory/DarkElfStarting.xml`) agrupa por zona narrativa; no siempre coincide con la celda — prioriza el contenido real.

## 5. Reglas anti-falsas-conclusiones

- **Nunca** concluyas desde un solo archivo: busca en TODO el árbol de spawns de cada raíz.
- **Compara conteos** SOURCE ↔ RUNTIME; si difieren, documenta el CONFLICT.
- Un mob puede tener spawns en varias celdas/archivos (p.ej. mobs que cruzan zonas).
- La ausencia en `spawns/` no significa "no existe": algunos NPCs se crean por scripts/AI (p.ej. quests que spawnean).
- No confundir el `name=` del grupo de spawn con zona formal del mundo (para zonas formales ver `mapregion/`; para lore-only marcar LORE/CONTEXT).

## 6. Checklist rápido

- [ ] Buscar ID en RUNTIME (`HighFiveSpawns.xml`) → conteo + líneas.
- [ ] Buscar ID en SOURCE (`spawns/**`) → conteo + archivo(s)/celda(s).
- [ ] ¿Conteos iguales? Si no → CONFLICT a documentar.
- [ ] Extraer x/y/z/heading/respawnDelay por spawn (o muestra representativa si son muchos).
- [ ] Cruzar ID con `stats/npcs/<rango>.xml` para nombre/nivel/tipo/título.
- [ ] Determinar región formal vía `mapregion/` (locId/town); lo no encontrado → LORE/CONTEXT.

## 7. Ejemplo completo verificado — Q00003

| ID | Nombre | RUNTIME | SOURCE | Coinciden |
|---|---|---|---|---|
| 20031 | Omen Beast | 12 | 12 (`Others/18_19.xml`) | ✅ |
| 20041 | Tainted Zombie | 12 | 12 | ✅ |
| 20046 | Stink Zombie | 13 | 13 | ✅ |
| 20048 | Lesser Succubus | 12 | 12 | ✅ |
| 20052 | LS Turen | 7 | 7 | ✅ |
| 20057 | LS Tilfo | 10 | 10 | ✅ |
| 30141 | Talloth (Folk Tetrarch) | 1 | 1 (`Others/20_18.xml`) | ✅ |

Detalle por entidad: [QUESTS/Q00003_PVE_VERTICAL_SLICE.md](../QUESTS/Q00003_PVE_VERTICAL_SLICE.md) §9.

---
**Status**: VERIFIED · Verified: 2026-08-26
