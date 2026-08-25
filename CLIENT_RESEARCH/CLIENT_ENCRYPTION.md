# CLIENT_ENCRYPTION

**Última actualización**: 2026-08-25 (Fase 3)
**Objeto**: estado del cifrado de los archivos de texto del cliente H5
**Procedencia**: CLIENT
**Estado**: VERIFIED (encabezado/estado cifrado observado) · **algoritmo de descifrado NO demostrado** (UNKNOWN)

> Regla aplicada: NO se documenta ningún algoritmo de descifrado como hecho porque **todavía no fue demostrado con este cliente**. Solo se registra lo observado.

---

## 1. Síntesis

Los archivos de **texto/datos** del cliente (los que contienen nombres de quests, NPC, items, skills, mensajes y HTML) **NO son legibles directamente**: comienzan con un **encabezado en claro** y continúan con **datos cifrados (blob binario no legible)**.

Los archivos de **assets** (`.utx/.usx/.unr/.uax/.ukx/...`) son binarios de Unreal Engine, no texto cifrado; se tratan aparte.

## 2. Encabezado observado

El archivo `system/questname-e.dat` (y análogos) comienza así (hex):
```
4C 00 69 00 6E 00 65 00 61 00 67 00 65 00 32 00 56 00 65 00 72 00 34 00 31 00 33 00
```
que decodifica como **"Lineage2 Ver 413"** (UTF-16LE). Tras ese encabezado el resto es un **blob binario no descifrado** (bytes sin patrón de texto claro).

> Se observó el mismo patrón (cifrado) en `L2text/*.htm`: "Lineage2 Ver 413" (aquí puede variar ligeramente el número de versión según archivo) + datos binarios.

## 3. Archivos `.dat` de texto identificados (system/)

| Archivo | Lo que parece contener | Estado |
|---|---|---|
| `questname-e.dat` | Nombres de quests | Cifrado |
| `NpcName-e.dat` | Nombres de NPC | Cifrado |
| `NpcString-e.dat` | Strings de NPC (mensajes on-screen) | Cifrado |
| `itemname-e.dat` | Nombres de items | Cifrado |
| `skillname-e.dat` | Nombres de skills | Cifrado |
| `systemmsg-e.dat` | Mensajes de sistema | Cifrado |
| `sysstring-e.dat` | Strings de sistema | Cifrado |
| `zonename-e.dat` | Nombres de zonas | Cifrado |
| `huntingzone-e.dat` | Zonas de caza | Cifrado |
| `CastleName-e.dat` | Nombres de castillos | Cifrado |
| `ClassInfo-e.dat` | Info de clases | Cifrado |
| `ActionName-e.dat` / `commandname-e.dat` | Nombres de acciones/comandos | Cifrado |
| `servername-e.dat` | Nombres de servidores | Cifrado |
| `gametip-e.dat` | Consejos de juego | Cifrado |

> El sufijo `-e` sugiere variante **localizada/inglesa** del pack; no se ha verificado otra variante.

## 4. ¿Qué información parece contener cada archivo?

Como son los archivos clásicos de texto de un cliente L2, se **presume** (ASSUMPTION) que cada uno contiene la tabla de nombres/strings del ID correspondiente (quest id → nombre; npc id → nombre/título; item id → nombre; skill id → nombre; system message id → texto). **Esto es una hipótesis razonada, no un dato verificado** (el contenido concreto no se ha descifrado).

## 5. Lo que NO puede verificarse todavía (UNKNOWN)

- El **contenido concreto** de cada `.dat` (los nombres reales tal como los muestra el cliente).
- El **algoritmo de cifrado/descifrado** exacto de ESTE cliente.
- El mapeo **ID → texto** real en el lado cliente (quest id 1 → "Letters of Love", etc.).
- La posible diferencia entre los `.dat` ingleses/locales y los datos del servidor (que está en claro).

## 6. Bloqueo

La lectura directa del texto del cliente está **BLOQUEADA** por el cifrado. Las opciones seguras para superarlo (a decidir/implementar en una fase futura, **sin instalar software de terceros de forma automática**):
1. Analizar y replicar el esquema de descifrado del cliente L2 (requiere demostrarlo sin modificar el cliente).
2. Utilizar como autoridad el lado del SERVIDOR (en claro) para nombres/textos de quests/NPCs/items (fuente primaria investigable hoy).
3. Comparar con datos observados en ejecución (OBSERVED).

**No se ha instalado ni ejecutado ninguna herramienta de terceros.**

---

## Enlaces

- [CLIENT_STRUCTURE.md](CLIENT_STRUCTURE.md) — mapa del cliente
- [QUEST_PILOT_Q00001.md](QUEST_PILOT_Q00001.md) — caso de estudio (usa el lado servidor en claro)
- [CLIENT_SERVER_MAPPING.md](CLIENT_SERVER_MAPPING.md) — autoridad por entidad
