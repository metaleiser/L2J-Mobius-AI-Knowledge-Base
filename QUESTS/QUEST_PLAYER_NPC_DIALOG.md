# QUEST PLAYER / NPC / DIALOG

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Quests — interacción Player↔NPC↔diálogo  
**Source of Truth**: `mechanics/script/Quest.java` (showHtmlFile L2266+, showResult L1190+, notifyTalk/FirstTalk), `network/serverpackets/NpcHtmlMessage.java` (referencia)  
**Verified**: 2026-08-23  
**Status**: VERIFIED (conexión) · detalle de packets → PACKETS PHASE

---

## 1. FLUJO DE DIÁLOGO REAL

```
[Cliente] click/bypass sobre NPC
   ↓
Packet handler (RequestBypassToServer / Action)  →  resuelve NPC objetivo
   ↓
ScriptManager → quest registrada para ese npcId (addStartNpc/addTalkId/addFirstTalkId)
   ↓
quest.notifyFirstTalk(npc, player) → onFirstTalk(...)   // primera interacción
quest.notifyTalk(npc, player)      → onTalk(...)        // siguientes
quest.notifyEvent(event, npc, player) → onEvent(...)    // bypass con evento ("30048-06.htm" etc.)
   ↓ retorno String htmltext (o null)
showResult(player, res, npc)
   ├─ res termina en .html/.htm → showHtmlFile(player, res, npc)   // carga HTML del datapack
   └─ si no → new NpcHtmlMessage(npcObjId, res)                    // contenido directo
   ↓
NpcHtmlMessage packet → cliente
```

## 2. MÉTODOS VERIFICADOS

| Método | Línea | Función |
|--------|-------|---------|
| `notifyTalk(Npc, Player)` | L638 | despacha onTalk |
| `notifyFirstTalk(Npc, Player)` | L669 | despacha onFirstTalk |
| `notifyEvent(String, Npc, Player)` | L618 | despacha onEvent |
| `onEvent(String, Npc, Player) → String` | L874 | hook principal de bypass |
| `showResult(Player, String)` / `(Player, String, Npc)` | L1190/1209 | envía resultado como NpcHtmlMessage |
| `showHtmlFile(Player, String filename[, Npc]) → String` | L2266/2279 | carga HTML y lo envía |
| `showError(Player, Throwable) → boolean` | L1167 | muestra error al GM/log |

- Import verificado: `network.serverpackets.NpcHtmlMessage` (Quest L178).
- `showHtmlFile` construye `new NpcHtmlMessage(objectId, content)` (L2303).

## 3. CONVENCIÓN DE BYPASS EN SCRIPTS

En el ejemplo real (`Q00001_LettersOfLove.onEvent`):
```java
switch (event) {
    case "30048-03.html": case "30048-04.html": ...   // solo mostrar HTML
    case "30048-06.htm":                              // acción + HTML
        if (player.getLevel() >= MIN_LEVEL) { qs.startQuest(); giveItems(...); }
}
return htmltext;
```
- El nombre de archivo HTML suele codificar `npcId-página`.
- Los botones del cliente envían bypass que llega como `event` a `onEvent`.

## 4. UBICACIÓN DE LOS HTML DE QUEST

- Los HTML viven en los directorios de datapack de diálogos (`dist/game/data/html/...`); la carga la realiza el mecanismo de HTML cacheado del servidor.
- Detalle de cache/ruta exacta → conexión con sistema HTML (fase futura); aquí solo queda mapeada la interfaz.

## 5. RELACIÓN CON AI/NPC

- `addStartNpc` marca NPCs iniciadores; `addTalkId` habilita diálogo; ambos alimentan los registros que los packets consultan vía ScriptManager/registros de listeners NPC.
- El AI del NPC no participa en el diálogo: es una ruta directa packet→quest.

## 6. TRAZABILIDAD

| Elemento | Path |
|----------|------|
| Despachadores | `mechanics/script/Quest.java` L618/L638/L669 |
| Salida HTML | ídem L1190–1230, L2266–2310 |
| Packet destino | `network/serverpackets/NpcHtmlMessage.java` |
| Registro NPCs | helpers `addStartNpc/addTalkId/addFirstTalkId` en Quest.java |

---
## Ver también

- [SYSTEMS/PLAYER_SYSTEM.md](../SYSTEMS/PLAYER_SYSTEM.md) — mapa de dependencias del jugador
- [SYSTEMS/NPC_SYSTEM.md](../SYSTEMS/NPC_SYSTEM.md) — NPCs, templates e interacción
- [AI/TARGET_SYSTEM.md](../AI/TARGET_SYSTEM.md) — cómo se resuelve el target semilla
- [CLIENT_RESEARCH/QUEST_PILOT_Q00001.md](../CLIENT_RESEARCH/QUEST_PILOT_Q00001.md) — diálogos/NPC de quest vistos del lado CLIENT↔SERVER

---
**Status**: VERIFIED (conexión completa hasta NpcHtmlMessage) · interior de packets → PACKETS PHASE  
**Verified**: 2026-08-23