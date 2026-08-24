# CHANGELOG — AI_KNOWLEDGE_BASE

Formato: entradas concisas y cronológicas (descendentes). El detalle completo vive en AUDIT_HISTORY.md y en los informes de sesión.

---

## KB 1.0 — 2026-08-24

**Baseline**: `e2518ab10872b28cd4c6860e102b493656ba8728` (master, 2026-08-22 03:06:03 +0300)

### Consolidación inicial
- KB consolidada sobre snapshot local verificado estructuralmente contra el commit baseline (inventarios exactos: java 3194 · xml 2907 · sql 116 · ini 73 · properties 0 · total 27657).

### Auditoría FASE 1
- Inventario físico: 67 documentos técnicos en 13 áreas; fechas únicas 2026-08-23.
- Enlaces: ~85 destinos internos; 1 enlace malformado (`](SYSTEMS/)`); huérfano legítimo: MASTER_INDEX.
- Contradicciones internas detectadas (C1–C6): estados de fase, conteos packets, tablas MANAGERS_INDEX, README obsoleto.

### Auditoría FASE 2 (código ↔ KB @ baseline)
- Hallazgos H1–H15 clasificados por severidad (HIGH/MEDIUM/LOW) con evidencia citada.
- Áreas verificadas contra código: configuración (16+44), DatabaseConfig/DatabaseFactory (HikariCP/L2JMobiusPool), sin FK en SQL, arranque GameServer L210–216, managers 58/52/6, bases de packets, layout/build.

### Correcciones aplicadas (autorización explícita del usuario)
- H1–H4: conteos unificados clientpackets=280 / serverpackets=389 / total=669.
- H5: eliminado `commons/network/network/NetworkThread` (clase inexistente); sin claims NIO.
- H6: `FloodProtectors` → `gameserver/util/`.
- H7/H8: CONFIGURATION_SYSTEM reescrito al mecanismo real (`ConfigReader` + `load()` + `.ini`; ejemplo verbatim de `DatabaseConfig.java`).
- H9: override por variables de entorno → UNKNOWN.
- H10: rutas personales obsoletas (`c:\L2J KNOLEDGE`) eliminadas en 5 documentos.
- H11/H12: README sincronizado con MASTER_INDEX (fase 2F) y estructura real de carpetas.
- H13: MANAGERS_INDEX regenerado desde código (58=52+6).
- H14: `dist/game/log` marcado runtime-generated.
- H15: referencias "750+" corregidas a 669.
- Nuevas durante validación: **H16** `SendablePacket`→`WritablePacket` (3 docs) · **H17** genérico `<String>`→`<GameClient>` · **H18** residuos 285/398/750+/Thread.ini/sobreafirmación singletons.
- Total documentos modificados: **15**. Documentos técnicos creados: **0** (67 constantes).

### Sistema de mantenimiento (FASE 3–4)
- Creado `AI_INSTRUCTIONS/` (AI_README, VERIFICATION_RULES, AUDIT_PROTOCOL, UPDATE_PROTOCOL, CHANGE_DETECTION).
- Creado `VERSIONING/` (KB_VERSION, UPSTREAM_BASELINE, CHANGELOG, AUDIT_HISTORY).
- MASTER_INDEX: sección mínima de descubrimiento hacia ambos directorios.

### Áreas VERIFIED (contra e2518ab)
CONFIGURATION (inventario+Database.ini+ConfigReader) · DATABASE core (pool/FK/arranque) · MANAGERS (recuento/firmas) · PACKETS (estructura/conteos/bases) · BUILD & LAYOUT · arranque GameServer.

### Áreas PARTIAL
CONFIGURATION_SYSTEM (claves PVP/Feature RCV) · PACKET docs (opcodes/enums RCV) · MANAGERS_INDEX (descripciones heredadas) · COMMONS/NETWORK/ARCHITECTURE_OVERVIEW/PROJECT_STRUCTURE/README/LOGINSERVER_ARCHITECTURE (parcialmente auditadas).

### UNKNOWN relevantes
hot-reload configs · env override · autoCommit global · backup retención · claves Interface/PVP/Feature.ini · catálogo opcodes/enums · jerarquía entidades profunda · mecanismo E/S red · multi-cliente · roles 4 managers games/.

---