# VERIFICATION_RULES — Reglas obligatorias de verificación

Aplican a cualquier afirmación escrita o modificada en esta KB. Última revisión: 2026-08-24.

1. **Evidencia concreta obligatoria para VERIFIED.** Toda afirmación VERIFIED debe poder relacionarse con evidencia específica del código o filesystem (formato preferente `ruta/archivo.java:línea`, o recuento de filesystem con fecha).

2. **Evidencia insuficiente ⇒ UNKNOWN / REQUIRES CODE VERIFICATION.** Nunca elevar a VERIFIED por comodidad.

3. **La documentación antigua no es evidencia.** Que algo estuviera escrito aquí no prueba que siga siendo cierto; re-verificar contra código.

4. **Prohibido copiar estructuras de otras versiones de L2J/Mobius** (otras crónicas, forks, packs conocidos). Este build se documenta desde este build.

5. **No asumir igualdad de arquitectura entre versiones** (p. ej. clases que existen en otros packs pueden no existir aquí: caso `NetworkThread`, `SendablePacket`).

6. **Conteos con fecha/commit.** Todo conteo variable (clases, archivos, tablas, handlers) indica cuándo/contra qué commit se midió. Conteos vigentes actuales: clientpackets=280, serverpackets=389, managers=58 (52 con firma getInstance), configs core=16, customs=44, SQL=116, INI=73, `.properties`=0 (baseline `e2518ab`, verificado 2026-08-24).

7. **Paths relativos al proyecto.** Prohibido rutas personales absolutas tipo `C:\Users\...`. Raíz de referencia = raíz del servidor (`java/`, `dist/`). La ubicación física del workspace se documenta solo en [AI_README.md](AI_README.md).

8. **Singleton solo si está demostrado.** Existir `getInstance()` en algunos managers no hace singleton a todos: 52 de 58 tienen firma; 6 no. No generalizar.

9. **Mecanismos de red solo con evidencia.** No afirmar NIO/AIO/select-loop/etc. sin leer el código que lo implementa.

10. **Prohibido inventar** tablas SQL, relaciones/FK (no existen FOREIGN KEY ni REFERENCES en este build — verificado), configuraciones, defaults, opcodes, handlers, managers, paquetes o métodos.

11. **Cliente ≠ Servidor.** Afirmaciones sobre el cliente (`L2 H5`) se marcan CLIENT y viven separadas de las del servidor hasta que exista fase CLIENT↔SERVER.

12. **Commit de referencia para datos upstream.** Cuando una afirmación provenga de análisis del clone upstream, conservar el commit usado.

---

### Plantilla mínima de evidencia recomendada

```
Afirmación: <texto>
Estado: VERIFIED | PARTIAL | UNKNOWN | REQUIRES CODE VERIFICATION
Evidencia: ruta/archivo.ext:línea — extracto breve | recuento filesystem YYYY-MM-DD
Baseline: e2518ab10872b28cd4c6860e102b493656ba8728 (si aplica)
```