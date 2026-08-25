# QUEST_PILOT_Q00001 — Letters of Love

**Última actualización**: 2026-08-25 (Fase 3)
**Caso**: Q00001_LettersOfLove · Quest ID 1
**Método**: cruce SERVER_SOURCE + SERVER_RUNTIME (+ cliente para correlación de nombres)
**Taxonomía**: cada dato indica procedencia y estado KB v2.0

> Aviso de método: no se asume que SERVER_SOURCE y SERVER_RUNTIME son idénticos. Se confirma que **difieren** (ver §7).

---

## 1. Identificación

| Campo | Valor | Procedencia | Estado |
|---|---|---|---|
| Quest ID | `1` | SOURCE `super(1)` / RUNTIME `super(1)` | VERIFIED |
| Nombre (código) | `Q00001_LettersOfLove` (carpeta y clase) | SOURCE y RUNTIME | VERIFIED |
| Nombre mostrado | "Letters of Love" (en claro en el lado server) | SOURCE/RUNTIME | VERIFIED |
| Nombre en CLIENTE | (no comprobable; `questname-e.dat` cifrado) | CLIENT | **UNKNOWN** |

## 2. Scripts y ubicación

| Script | SOURCE | RUNTIME |
|---|---|---|
| `.java` | `UPSTREAM/.../dist/game/data/scripts/quests/Q00001_LettersOfLove/` | `L2J_Mobius_CT_2.6_HighFive/game/data/scripts/quests/Q00001_LettersOfLove/` |
| HTML (15) | presente (`30006-*`, `30033-*`, `30048-*`) | presente (15) — **mismo conteo** |

- Clase: `Q00001_LettersOfLove extends Quest` — VERIFIED (SOURCE y RUNTIME).
- Constructor: `super(1)`, `addStartNpc(DARIN)`, `addTalkId(DARIN, ROXXY, BAULRO)`, `registerQuestItems(...)`.

## 3. NPCs (verificados)

| Nombre | ID | Tipo | Título | SOURCE | RUNTIME |
|---|---|---|---|---|---|
| Darin | 30048 | Folk | — | script + `30000-30099.xml` | script + `30000-30099.xml` |
| Roxxy | 30006 | Teleporter | Gatekeeper | idem | idem |
| Baulro | 30033 | Trainer | Magister | idem | idem |

- Constantes del script: `DARIN=30048`, `ROXXY=30006`, `BAULRO=30033` — VERIFIED.
- Nombres/títulos en `stats/npcs/30000-30099.xml` (RUNTIME) — VERIFIED (Darin 30048, Roxxy 30006 Gatekeeper, Baulro 30033 Magister).
- **Correlación cliente**: nombres que el cliente muestra en `NpcName-e.dat` → **UNKNOWN** (cifrado).

## 4. Items de quest (verificados)

| Item | ID | Tipo | Nombre (server) | Fuente (RUNTIME stats/items) |
|---|---|---|---|---|
| Carta de Darin | 687 | EtcItem | "Darin's Letter" | `00600-00699.xml` |
| Pañuelo de Roxxy | 688 | EtcItem | "Roxxy's Kerchief" | `00600-00699.xml` |
| Recibo de Darin | 1079 | EtcItem | "Darin's Receipt" | `01000-01099.xml` |
| Poción de Baulro | 1080 | EtcItem | "Baulro's Potion" | `01000-01099.xml` |
| Collar de Conocimiento | 906 | Armor | "Necklace of Knowledge" | `00900-00999.xml` |

- Constantes del script (SOURCE y RUNTIME): `DARINS_LETTER=687`, `ROXXYS_KERCHIEF=688`, `DARINS_RECEIPT=1079`, `BAULROS_POTION=1080`, `NECKLACE_OF_KNOWLEDGE=906` — VERIFIED.
- Nombres en `stats/items` (RUNTIME) — VERIFIED.
- **Correlación cliente** en `itemname-e.dat` → **UNKNOWN** (cifrado).

## 5. Estados, condiciones y flujo completo

Máquina de estados del motor (ver `QUESTS/QUEST_STATES_VARIABLES.md`): `CREATED → STARTED → COMPLETED`.

| Cond | Paso | Descripción | HTML | NPC |
|---|---|---|---|---|

---

## 7. DIFERENCIA SOURCE ↔ RUNTIME (CONFLICT / registrada)

Aunque el archivo y las 15 páginas HTML coinciden, el **código `.java` difiere** en la rama de finalización:

- **SERVER_SOURCE**: en cond 4 (Darin) incluye **integración con `ai.others.NewbieGuide.NewbieGuide`**: recupera el script `NewbieGuide`, consulta/avanza una **misión guía `GUIDE_MISSION = 41`** con `haveNRMemo/setNRMemo/setNRMemoState`, y solo en ciertos casos emite `showOnScreenMsg(...DELIVERY_DUTY...)`. Importa `ScriptManager` y `NewbieGuide`.
- **SERVER_RUNTIME**: en cond 4 (Darin) **NO hay integración NewbieGuide**; simplemente emite `showOnScreenMsg(...DELIVERY_DUTY...)`, da 906, 5672/446 EXP/SP, 2466 adena, y `exitQuest`.

| Aspecto | SOURCE | RUNTIME | Estado |
|---|---|---|---|
| Integración NewbieGuide (misión 41) | **SÍ** | **NO** | **CONFLICT** |
| `showOnScreenMsg(DELIVERY_DUTY...)` | SÍ (condicional) | SÍ (directo) | CONFLICT |
| Import de `ScriptManager`/`NewbieGuide` | SÍ | No | CONFLICT |

**Conclusión**: el runtime es una versión **más simple/anterior** respecto al source en esta quest (el source añade la integración con la guía de novatos). Es consistente con el hecho conocido de que **SOURCE baseline ≠ RUNTIME build** (ver `00_PROJECT/PROJECT_CONTEXT.md`). No se corrige ninguno; se registra la discrepancia.

> Nota: esta es una **diferencia real** corroborada (los dos `.java` fueron leídos completos). La KB previa ya documentaba Q00001 en `QUESTS/QUEST_ARCHITECTURE.md` como ejemplo técnico; este documento añade la dimensión CLIENT↔SERVER.

---

## 8. Correlación CLIENTE (cuando sea comprobable)

| Entidad | ID | Servidor (claro) | Cliente (archivo) | Estado |
|---|---|---|---|---|
| Quest | 1 | Letters of Love | `questname-e.dat` | **UNKNOWN** (cifrado) |
| NPC Darin | 30048 | Folk "Darin" | `NpcName-e.dat` | **UNKNOWN** (cifrado) |
| NPC Roxxy | 30006 | Teleporter "Roxxy" Gatekeeper | `NpcName-e.dat` | **UNKNOWN** (cifrado) |
| NPC Baulro | 30033 | Trainer "Baulro" Magister | `NpcName-e.dat` | **UNKNOWN** (cifrado) |
| Item 687 | 687 | "Darin's Letter" | `itemname-e.dat` | **UNKNOWN** (cifrado) |
| NpcString | — | `DELIVERY_DUTY...` (id en `NpcStringId`) | `NpcString-e.dat` | **UNKNOWN** (cifrado) |

Ningún texto del lado cliente pudo comprobarse (bloqueo por cifrado, ver `CLIENT_ENCRYPTION.md`).

---

## 9. Síntesis de estados

- **VERIFIED**: todo el flujo del lado servidor (NPC, items, condiciones, recompensas, NpcStringId) y la diferencia SOURCE↔RUNTIME.
- **UNKNOWN**: todos los textos del lado cliente (nombres mostrados por el cliente).
- **CONFLICT**: integración NewbieGuide presente en SOURCE, ausente en RUNTIME.

---

## Enlaces

- [CLIENT_SERVER_MAPPING.md](CLIENT_SERVER_MAPPING.md) — autoridad por entidad
- [CLIENT_ENCRYPTION.md](CLIENT_ENCRYPTION.md) — cifrado del cliente
- [../QUESTS/QUEST_ARCHITECTURE.md](../QUESTS/QUEST_ARCHITECTURE.md) — motor de quests (sin duplicar)
- [../QUESTS/QUEST_PLAYER_NPC_DIALOG.md](../QUESTS/QUEST_PLAYER_NPC_DIALOG.md) — flujo de diálogo
- [../00_PROJECT/PROJECT_CONTEXT.md](../00_PROJECT/PROJECT_CONTEXT.md) — entidades del workspace

| — | Inicio | `onEvent("30048-06.htm")` si `level>=2` → `startQuest()` + dar carta (687) | `30048-06.htm` | Darin |
| 1 | — | Lleva la carta (687) | `30048-07.html` | Darin |
| 1→2 | Dar carta a Roxxy | toma 687, da pañuelo 688, `setCond(2)` | `30006-01.html` | Roxxy |
| 2→3 | Volver a Darin | toma 688, da recibo 1079, `setCond(3)` | `30048-08.html` | Darin |
| 3→4 | Baulro | toma 1079, da poción 1080, `setCond(4)` | `30033-01.html` | Baulro |
| 4 | Entrega/recompensa | da 906, `addExpAndSp(5672,446)`, `giveAdena(2466)`, `exitQuest` | `30048-10.html` | Darin |

- `MIN_LEVEL = 2` — VERIFIED.
- Otros HTMLs: `30048-01` (bajo nivel), `30048-02` (aceptar), `30048-03/04/05` (conversación), `30006-02/03`, `30033-02` — VERIFIED (en scripts).

## 6. Recompensas y NpcStringId

| Recompensa | Valor | Procedencia | Estado |
|---|---|---|---|
| Item | Necklace of Knowledge (906) ×1 | SOURCE/RUNTIME | VERIFIED |
| EXP | 5672 | SOURCE/RUNTIME | VERIFIED |
| SP | 446 | SOURCE/RUNTIME | VERIFIED |
| Adena | 2466 | SOURCE/RUNTIME | VERIFIED |

- **NpcStringId**: `DELIVERY_DUTY_COMPLETE_N_GO_FIND_THE_NEWBIE_GUIDE` — VERIFIED (definido en `SOURCE/java/.../network/NpcStringId.java` y usado en el script).
- Texto legible que el cliente mostrará para ese NpcStringId → **UNKNOWN** (el string del cliente está en `NpcString-e.dat` cifrado).
