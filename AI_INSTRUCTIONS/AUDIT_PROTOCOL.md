# AUDIT_PROTOCOL — Protocolo de auditoría de la KB

Última revisión: 2026-08-24. Objetivo: auditar periódicamente la KB contra código local, clone upstream y baseline conocida, sin modificar nada hasta tener autorización.

## FASE A — Identificación

1. Localizar KB: `AI_KNOWLEDGE_BASE/` (raíz del workspace).
2. Localizar servidor: carpeta anidada `L2J_Mobius_CT_2.6_HighFive` (ver mapa en [AI_README.md](AI_README.md)).
3. Localizar cliente si existe: `L2 H5/`.
4. Localizar clone upstream: `UPSTREAM/L2J_Mobius`.
5. Obtener commit actual del clone:
   - `git -C UPSTREAM/L2J_Mobius rev-parse HEAD`
   - `git -C UPSTREAM/L2J_Mobius status --porcelain` (debe estar limpio)
6. Comparar con `UPSTREAM_BASELINE_COMMIT` ([../VERSIONING/UPSTREAM_BASELINE.md](../VERSIONING/UPSTREAM_BASELINE.md)). Si difiere → hay cambios upstream; continuar en FASE C.

## FASE B — Inventario (filesystem manda)

Contar recursivamente en el snapshot del servidor y anotar fecha:

- `*.java`, `*.xml`, `*.sql`, `*.ini`, `*.properties`
- total de archivos
- documentos Markdown de la KB (67 técnicos + docs de sistema)

Cifras de referencia del baseline: java=3194, xml=2907, sql=116, ini=73, properties=0, total=27657. Cualquier desviación = cambio estructural.

## FASE C — Cambio upstream (solo lectura)

En el clone:

- `git fetch origin master`
- `git rev-parse HEAD` vs baseline
- `git log --oneline HEAD..origin/master` (commits nuevos)
- `git diff --name-status HEAD origin/master` (agregados/eliminados/modificados)
- `git diff --stat HEAD origin/master`

Clasificar archivos modificados por área (config/database/network/packets/managers/skills/quests/scripts/build).

⚠️ Prohibido ejecutar `pull/merge/rebase/reset/checkout` destructivos automáticamente.

## FASE D — Impact Analysis

Mapear cambios → documentos potencialmente afectados (usar las rutas citadas por cada doc). Producir lista: documento | afirmación en riesgo | evidencia necesaria. **No modificar todavía.**

## FASE E — Verificación puntual

Para cada ítem: localizar evidencia en código → determinar estado (VERIFIED/PARTIAL/UNKNOWN/RCV/OUTDATED/CONTRADICTED) → redactar corrección propuesta con motivo y evidencia ([VERIFICATION_RULES.md](VERIFICATION_RULES.md)).

## FASE F — Corrección

**Solo tras autorización explícita del usuario.** Aplicar únicamente lo demostrado; conservar UNKNOWN/RCV cuando falte evidencia.

## FASE G — Validación post-cambio

Checklist obligatorio:

- [ ] Recuento `.md` (técnicos = 67 + sistema)
- [ ] Enlaces internos sin rotos
- [ ] Sin huérfanos no justificados
- [ ] Conteos citados correctos y fechados
- [ ] Sin referencias obsoletas (rutas personales, clases inexistentes)
- [ ] UNKNOWN/RCV conservados
- [ ] Estado de versión actualizado ([../VERSIONING/KB_VERSION.md](../VERSIONING/KB_VERSION.md))
- [ ] Integridad servidor (inventario intacto)
- [ ] Integridad cliente
- [ ] Integridad clone (`status --porcelain` vacío, HEAD esperado)

Cerrar registrando en [../VERSIONING/AUDIT_HISTORY.md](../VERSIONING/AUDIT_HISTORY.md) y, si procede, CHANGELOG.