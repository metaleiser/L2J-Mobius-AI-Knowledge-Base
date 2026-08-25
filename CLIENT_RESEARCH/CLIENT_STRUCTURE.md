# CLIENT_STRUCTURE

**Última actualización**: 2026-08-25 (Fase 3)
**Objeto**: estructura real del cliente H5 `Lineage2-TCT-273-client/`
**Procedencia**: CLIENT
**Estado**: VERIFIED (looserver del árbol, recuentos y formatos) · UNKNOWN (contenido interno cifrado/no inspeccionado)

> Este documento describe SOLO lo verificado durante la Fase 3. Cada afirmación indica formato, función, legibilidad/estado de cifrado y qué investigación permite.

---

## 1. Resumen de carpetas

| Carpeta | Archivos | Formato | Función / Contenido |
|---|---|---|---|
| `system/` | 162 | `.dat`·`.dll`·`.int`·`.u`·`.ini`·`.bin`·`.sys`... | Motor del juego + **archivos de texto/datos del cliente** (nombres, strings) |
| `L2text/` | 276 | `.htm`/`.html` | HTML del cliente (eventos, UI, créditos, ayuda) |
| `Textures/` | 492 | `.utx` | Texturas (motor Unreal) |
| `StaticMeshes/` | 259 | `.usx` | Mallas estáticas 3D (Unreal) |
| `Maps/` | 208 | `.unr` | Mapas (Unreal) |
| `Sounds/` | 39 | `.uax` | Sonidos/efectos (Unreal) |
| `Animations/` | 44 | `.ukx`/`.uix` | Animaciones esqueléticas (Unreal) |
| `SysTextures/` | 70 | `.utx` | Texturas de sistema/UI (Unreal) |
| `Voice/` | 77 | — | Voces de personajes/NPC |
| `music/` | 403 | — | Música BGM |

---

## 2. Detalle de cada área

### system/
- **Formato**: mixto — `.dat` (52, textos/datos), `.dll` (28, librerías), `.int` (27, configuración lógica Unreal), `.u` (26, código Unreal), `.ini` (9, config), `.bin`/`.sys`/etc.
- **Función**: motor del cliente y **todos los archivos de nombres/texto** que el cliente muestra.
- **Legibilidad**: los archivos `.dat` de texto **NO son legibles** (están cifrados; ver `CLIENT_ENCRYPTION.md`).
- **Investigación que permite**: localizar los archivos fuente de nombres (quests, NPC, items, skills, system messages, zonas) e identificar IDs; su **contenido requiere descifrado** (bloqueado en esta fase).

### L2text/
- **Formato**: `.htm`/`.html` (276 + 1).
- **Función**: HTML empotrados del cliente (eventos, UI, ayuda, créditos).
- **Legibilidad**: **NO legibles** — cifrados (mismo esquema que `.dat`).
- **Investigación que permite**: identificar qué contenido gráfico/HTML está en el cliente; contenido bloqueado por cifrado.

### Textures/ (`.utx`, Unreal)
- **Función**: texturas (incluye iconos de items, NPC, mobs, efectos).
- **Legibilidad**: formato binario Unreal; **no legible como texto**; requiere visualizador Unreal.
- **Investigación que permite**: identificar iconos/gráficos por nombre de archivo; correlación visual de assets.

### StaticMeshes/ (`.usx`, Unreal)
- **Función**: mallas 3D de objetos/estructuras.
- **Legibilidad**: binario Unreal; requiere visor.
- **Investigación**: modelado/recursos 3D.

### Maps/ (`.unr`, Unreal)
- **Función**: mapas/términos del mundo.
- **Legibilidad**: binario Unreal; requiere visor.
- **Investigación**: zonas, lugares, distribución espacial.

### Sounds/ (`.uax`), Voice/ , music/
- **Función**: efectos de sonido, voces, música BGM.
- **Legibilidad**: binario/códec; requiere reproductor.
- **Investigación**: audio asociado a eventos/NPC/zonas.

### Animations/ (`.ukx`/`.uix`)
- **Función**: animaciones esqueléticas de personajes/NPC.
- **Legibilidad**: binario Unreal.

### SysTextures/ (`.utx`)
- **Función**: texturas de interfaz/sistema (UI).

---

## 3. Motor / versión

- Cliente indica **"Lineage2 Ver 413"** en el encabezado de sus archivos de datos (verificable en `system/*.dat` y `L2text/*.htm`), concordante con la identificación previa (Ver 413).
- Los assets usan el **formato del motor Unreal** (`.utx/.usx/.unr/.uax/.ukx/.uix/.u/.int`).

---

## 4. Qué puede investigarse directamente vs bloqueado

| Caso | Vía |
|---|---|
| Nombres de archivos del cliente (qué existe) | ✓ directo (filesystem) |
| Estructura de carpetas/formatos | ✓ directo |
| Métricas de recuento | ✓ directo |
| **Contenido de texto del cliente** (questnames, NPC names, itemnames, systemmsg, L2text HTML) | ✗ **bloqueado por cifrado** |
| Assets visuales/audio 3D | parcial (nombre de archivo sí; contenido requiere herramienta Unreal) |

---

## Enlaces

- [CLIENT_ENCRYPTION.md](CLIENT_ENCRYPTION.md) — qué está cifrado y estado
- [CLIENT_SERVER_MAPPING.md](CLIENT_SERVER_MAPPING.md) — dónde vive la información por entidad
- [QUEST_PILOT_Q00001.md](QUEST_PILOT_Q00001.md) — caso de estudio CLIENT↔SERVER
- [../00_PROJECT/PROJECT_CONTEXT.md](../00_PROJECT/PROJECT_CONTEXT.md) — entidades del workspace
