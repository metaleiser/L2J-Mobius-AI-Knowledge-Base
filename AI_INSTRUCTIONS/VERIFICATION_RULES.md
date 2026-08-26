# VERIFICATION_RULES — Reglas obligatorias de verificación

Aplican a cualquier afirmación escrita o modificada en esta KB. Última revisión: 2026-08-26 (KB v2.3).

## Taxonomía de estados (obligatoria)

| Estado | Significado |
|---|---|
| **VERIFIED** | Confirmado mediante código, cliente, ejecución reproducible o evidencia suficiente (formato preferente `ruta/archivo.java:línea`, o recuento filesystem con fecha). |
| **DIRECTLY_SUPPORTED** | La afirmación se deriva directamente del código citado sin interpretación adicional (más débil que VERIFIED por línea exacta, más fuerte que orientación). *(añadido KB v2.2)* |
| **ORIENTATION_ONLY** | El documento orienta hacia dónde investigar, pero el hecho concreto aún requiere SOURCE. *(añadido KB v2.2)* |
| **SOURCE_REQUIRED** | Toda afirmación concreta de este dominio exige verificación en SOURCE antes de usarse. *(añadido KB v2.2)* |
| **OUTDATED** / **DEPRECATED** | Información anteriormente válida pero sustituida/desfasada; re-verificar antes de usar. |
| **CONTRADICTED** / **CONFLICT** | Existen fuentes o evidencias contradictorias; registrar ambas. |
| **OBSERVED** | Observado durante una prueba/uso del servidor o cliente, pero aún no completamente explicado. No elevar a VERIFIED sin confirmación. |
| **UNVERIFIED** | Afirmación encontrada en una fuente que aún no fue comprobada. Requiere inspección antes de usarse como cierta. |
| **ASSUMPTION** | Hipótesis de trabajo; nunca presentarse como hecho. |
| **UNKNOWN** | No existe información suficiente. |
| **UNKNOWN_CLIENT** | Dato que corresponde al CLIENTE (texto localizado, UI) y no puede verificarse porque los `.dat` están cifrados. Nunca presentar como hecho server-side. *(formalizado KB v2.2)* |

**Mapeo con estados históricos (no renombrar destructivamente):**
- `REQUIRES CODE VERIFICATION` → **UNVERIFIED**
- `OUTDATED` → **DEPRECATED** (o OUTDATED cuando solo esté desfasado y pueda revalidarse)
- `CONTRADICTED` → **CONFLICT**
- `VERIFIED`, `UNKNOWN` → se conservan
- `PARTIAL`, `PLANNED` → se conservan como históricos

## Metadata estándar recomendada (KB v2.2)

Para documentos nuevos e investigaciones futuras:

```
Status:        VERIFIED | DIRECTLY_SUPPORTED | ORIENTATION_ONLY | SOURCE_REQUIRED | PARTIAL | UNKNOWN | UNKNOWN_CLIENT | CONFLICT
Source:        rutas SOURCE/RUNTIME/XML/CLIENT usadas
Source commit: <hash> (p.ej. e2518ab…) cuando provenga del clone upstream
Verified date: AAAA-MM-DD
Scope:         qué cubre y qué queda fuera
Authority:     SOURCE > RUNTIME > XML > CLIENT > KB (indicar nivel usado)
```

## Entidades y procedencia (KB v2.0)

1. Toda afirmación debe indicar su **procedencia**: `SERVER_SOURCE`, `SERVER_RUNTIME`, `CLIENT` o `KNOWLEDGE_BASE`. No mezclar SOURCE/RUNTIME/CLIENT sin señalar de dónde viene cada dato.
2. **SOURCE ≠ RUNTIME**: si difieren, registrar CONFLICT/DEPRECATED y el contexto/versionado de cada lado. No corregir ninguno automáticamente.
3. **CLIENT ≠ SERVER**: afirmaciones del cliente se marcan `CLIENT` y viven separadas de las del servidor.

## Reglas de evidencia

4. **Evidencia concreta obligatoria para VERIFIED.** Toda afirmación VERIFIED debe relacionarse con evidencia específica (formato preferente `ruta/archivo.java:línea`, o recuento de filesystem con fecha).
5. **Evidencia insuficiente ⇒ UNVERIFIED / UNKNOWN / ASSUMPTION.** Nunca elevar a VERIFIED por comodidad.
6. **La documentación antigua no es evidencia.** Que algo estuviera escrito aquí no prueba que siga siendo cierto; re-verificar.
7. **Prohibido copiar estructuras de otras versiones de L2J/Mobius** (otras crónicas, forks, packs conocidos). Este build se documenta desde este build.
8. **No asumir igualdad de arquitectura entre versiones** (clases que existen en otros packs pueden no existir aquí: caso `NetworkThread`, `SendablePacket`).
9. **Conteos con fecha/commit.** Todo conteo variable (clases, archivos, tablas, handlers) indica cuándo/contra qué commit se midió y de qué entidad. Conteos vigentes (SOURCE `e2518ab`, 2026-08-24): clientpackets=280, serverpackets=389, managers=58 (52 con firma getInstance), configs core=16, customs=44, SQL=116, INI=73, `.properties`=0.
10. **Paths relativos a la entidad de origen** (`UPSTREAM/...`, `L2J_Mobius_CT_2.6_HighFive/...`, `Lineage2-TCT-273-client/...`). Prohibido rutas personales absolutas tipo `C:\Users\...`.
11. **Singleton solo si está demostrado.** Existir `getInstance()` en algunos managers no hace singleton a todos: 52 de 58 tienen firma; 6 no. No generalizar.
12. **Mecanismos de red solo con evidencia.** No afirmar NIO/AIO/select-loop/etc. sin leer el código que lo implementa.
13. **Prohibido inventar** tablas SQL, relaciones/FK (no existen FOREIGN KEY ni REFERENCES en este build — verificado), configuraciones, defaults, opcodes, handlers, managers, paquetes o métodos.
14. **Commit de referencia para datos upstream.** Cuando una afirmación provenga de análisis del clone upstream, conservar el commit usado.
15. **Build = Apache Ant** (no Gradle) para H5; no afirmar lo contrario sin evidencia.

---

### Plantilla mínima de evidencia recomendada

```
Afirmación: <texto>
Procedencia: SERVER_SOURCE | SERVER_RUNTIME | CLIENT | KB
Estado: VERIFIED | OBSERVED | UNVERIFIED | ASSUMPTION | UNKNOWN | DEPRECATED | CONFLICT
Evidencia: ruta/archivo.ext:línea — extracto breve | recuento filesystem YYYY-MM-DD
Baseline: e2518ab10872b28cd4c6860e102b493656ba8728 (SOURCE) / 26/05/2024 (RUNTIME) — según caso
```
