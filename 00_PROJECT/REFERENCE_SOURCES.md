# REFERENCE_SOURCES

**Última actualización**: 2026-08-25 (KB v2.0)

Documento de **registro** de fuentes: solo indica *fuente, propósito, autoridad y cuándo utilizarla*.
**NO contiene** ni copia el contenido de ninguna fuente. Una fuente se consulta cuando es útil; su contenido **no** se convierte automáticamente en verdad — debe pasar por análisis/verificación antes de entrar a la KB.

---

## Fuentes registradas

| Fuente | Propósito | Autoridad | Cuándo utilizarla |
|---|---|---|---|
| **Repo Git de la KB** — `https://github.com/metaleiser-vscode/L2J-Mobius-AI-Knowledge-Base.git` | Versionar la documentación del proyecto | Historial de la KB | Al modificar/consultar esta documentación |
| **Repo oficial de Mobius (upstream)** — `https://gitlab.com/MobiusDevelopment/L2J_Mobius.git` | Código fuente oficial de Mobius (todas las crónicas) | **Server Source** — autoridad de código/arquitectura | Implementar o estudiar código real |
| **Clone local upstream** — `UPSTREAM/L2J_Mobius` | Copia local del repo oficial (sparse: solo H5) | Igual que el repo oficial, HEAD local `e2518ab…` | Comparar, verificar código sin conexión |
| **SERVER_RUNTIME local** — `L2J_Mobius_CT_2.6_HighFive` | Servidor H5 desplegado que usamos para jugar/probar | Autoridad de **estado desplegado/observable** (build 26/05/2024) | Ver qué hay realmente ejecutándose |
| **Cliente H5** — `Lineage2-TCT-273-client` | Cliente real (v413, `system/l2.exe`) | Autoridad para **info solo-cliente** | Textos, diálogos, recursos, comportamiento cliente |
| **Wiki / documentación oficial de Mobius** | Documentación general y guías | Referencia secundaria | Contexto general, nunca como verdad única |
| **README / notas del proyecto** | Notas del pack (requisitos, links) | Referencia | Requisitos de entorno, versiones |

---

## Jerarquía de uso

```
CÓDIGO (SOURCE) > RUNTIME (estado observable) > CLIENT (solo-cliente) > KB > WIKI/REFERENCIA > SUPUESTO
```

Reglas:
- La KB **documenta** conocimiento verificado; no reemplaza la evidencia original.
- Las fuentes externas (wiki, foros, packs de terceros) son **investigación**, no verdad automática.
- No se copian wikis completas ni contenido externo dentro de la KB.

---

## Notas de verificación (KB v2.0)

- Build del source H5 = **Apache Ant** (`build.xml`), NO Gradle.
- Baseline source verificado: `e2518ab10872b28cd4c6860e102b493656ba8728` (GitLab, rama `master`).
- Runtime: build **26/05/2024**, sin Git, JARs propios.
- Cliente: `Lineage2-TCT-273-client`, protocolo/versión indicada como "Lineage2 Ver413" en `system/l2.ini`.
