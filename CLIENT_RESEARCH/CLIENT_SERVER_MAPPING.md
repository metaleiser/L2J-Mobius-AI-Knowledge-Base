# CLIENT_SERVER_MAPPING

**Última actualización**: 2026-08-25 (Fase 3)
**Objeto**: mapa conceptual de dónde vive la información de cada ENTIDAD y quién es la autoridad
**Taxonomía**: VERIFIED (rutas/IDs del lado server) · UNKNOWN (contenido cliente cifrado)

> Objetivo: establecer dónde reside cada dato y qué lado es autoridad. El lado cliente (archivos `.dat` de texto) está **cifrado** en este cliente, por lo que su contenido concreto es UNKNOWN hasta superar el bloqueo (ver `CLIENT_ENCRYPTION.md`).

---

## 1. Tabla conceptual ENTIDAD × FUENTE

**Leyenda de rutas** (relativas a la raíz del workspace):
- **CLIENT**: `Lineage2-TCT-273-client/`
- **SERVER_SOURCE**: `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive/`
- **SERVER_RUNTIME**: `L2J_Mobius_CT_2.6_HighFive/`

| ENTIDAD | CLIENT SOURCE | SERVER SOURCE | SERVER RUNTIME | STATUS |
|---|---|---|---|---|
| **Quest** | `system/questname-e.dat` (cifrado) | `dist/game/data/scripts/quests/Q00XXX_*/Q…java` + HTML | `game/data/scripts/quests/Q00XXX_*/` + HTML | Server VERIFIED; cliente UNKNOWN |
| **NPC** | `system/NpcName-e.dat` (cifrado) | `dist/.../stats/npcs/*.xml` | `game/data/stats/npcs/*.xml` | Server VERIFIED; cliente UNKNOWN |
| **Item** | `system/itemname-e.dat` (cifrado) | `dist/.../stats/items/*.xml` | `game/data/stats/items/*.xml` | Server VERIFIED; cliente UNKNOWN |
| **Skill** | `system/skillname-e.dat` (cifrado) | `dist/.../stats/skills/*.xml` | `game/data/stats/skills/*.xml` | Server VERIFIED; cliente UNKNOWN |
| **NpcString** | `system/NpcString-e.dat` (cifrado) | `java/.../network/NpcStringId.java` (enum) | (el server usa el enum, no el texto) | Enum VERIFIED; texto cliente UNKNOWN |
| **Zone** | `system/zonename-e.dat`, `huntingzone-e.dat` (cifrado) | `dist/.../data/zones/*.xml`, `mapregion` | `game/data/zones/*.xml`, `mapregion` | Server VERIFIED; cliente UNKNOWN |

---

## 2. Autoridad por entidad

| Caso | Autoridad |
|---|---|
| Implementación/lógica de quests (NPC, items, condiciones, recompensas) | **SERVER** (scripts `.java` en claro) |
| Nombres/títulos técnicos y datos de NPC/item/skill | **SERVER** (`stats/*.xml` en claro) |
| Texto que el cliente muestra (nombres de quest/NPC/item, NpcString, systemmsg, nombres de zona) | **CLIENT** (`.dat` de texto) — **pero cifrado** → no legible ahora |
| Enum de NpcStringId (constantes que el server referencia) | **SERVER_SOURCE** (`NpcStringId.java`) |

Regla: cuando se pueda leer el contenido del cliente, será la autoridad para los **strings mostrados**; el server es la autoridad para **lógica e IDs**. Ambos deben correlacionarse por ID (quest id, npc id, item id, skill id, NpcStringId).

---

## 3. Correlación por ID (metodología)

Para cada entidad, la correlación CLIENT↔SERVER se hace por **ID numérico**:

```
Quest ID   (1, 2, ...)   ↕   questname-e.dat (texto cliente)  ·  quest Q00XXX (server)
NPC ID     (30006, ...)  ↕   NpcName-e.dat  (texto cliente)   ·  stats/npcs/*.xml
Item ID    (687, ...)    ↕   itemname-e.dat (texto cliente)   ·  stats/items/*.xml
Skill ID                 ↕   skillname-e.dat                  ·  stats/skills/*.xml
NpcStringId              ↕   NpcString-e.dat                  ·  NpcStringId.java (enum)
Zone ID                  ↕   zonename-e.dat                   ·  data/zones/*.xml
```

Hoy **solo el lado server es verificable** (en claro). El lado cliente queda UNKNOWN hasta superar el cifrado.

---

## 4. Caso verificado (Q00001)

Ver `QUEST_PILOT_Q00001.md` para el ejemplo aplicado: IDs de quest/NPC/items/NpcStringId presentes en el server (VERIFIED) y su contraparte cliente (UNKNOWN por cifrado).

---

## Enlaces

- [CLIENT_STRUCTURE.md](CLIENT_STRUCTURE.md) — mapa del cliente
- [CLIENT_ENCRYPTION.md](CLIENT_ENCRYPTION.md) — cifrado
- [QUEST_PILOT_Q00001.md](QUEST_PILOT_Q00001.md) — caso de estudio
- [../00_PROJECT/PROJECT_CONTEXT.md](../00_PROJECT/PROJECT_CONTEXT.md) — entidades del workspace
