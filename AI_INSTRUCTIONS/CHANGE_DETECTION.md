# CHANGE_DETECTION — Detección de cambios upstream

Última revisión: 2026-08-24. Clone de referencia: `UPSTREAM/L2J_Mobius` (rama `master`, sparse-checkout del proyecto).

## Comandos permitidos (SOLO LECTURA)

Ejecutar siempre con `-C` apuntando al clone:

```bash
git -C UPSTREAM/L2J_Mobius fetch origin master
git -C UPSTREAM/L2J_Mobius rev-parse HEAD
git -C UPSTREAM/L2J_Mobius rev-parse origin/master
git -C UPSTREAM/L2J_Mobius log --oneline HEAD..origin/master
git -C UPSTREAM/L2J_Mobius diff --stat HEAD origin/master
git -C UPSTREAM/L2J_Mobius diff --name-status HEAD origin/master
```

Interpretación:

- Si `HEAD == baseline` y no aparecen commits nuevos → sin cambios; registrar auditoría "sin novedad".
- Si hay commits nuevos → listar, clasificar archivos por área y pasar a Impact Analysis ([UPDATE_PROTOCOL.md](UPDATE_PROTOCOL.md)).
- `--name-status`: A=añadido, M=modificado, D=eliminado. `--stat`: magnitud del cambio.

## Operaciones PROHIBIDAS sin autorización explícita

- `git pull` · `git merge` · `git rebase`
- `git reset` (cualquier modo)
- `git checkout` destructivo o cambio de rama
- Cualquier escritura dentro de `UPSTREAM/` o del snapshot del servidor

El clone existe para **comparar**, no para sincronizar automáticamente.

## Semántica del baseline

`UPSTREAM_BASELINE_COMMIT` = commit contra el cual la KB actual está validada.

Valor vigente:
```
UPSTREAM_BASELINE_COMMIT = e2518ab10872b28cd4c6860e102b493656ba8728
BASELINE_DATE            = 2026-08-22 03:06:03 +0300
```

Flujo de rotación del baseline:

1. Auditoría N compara HEAD vs baseline N.
2. Tras aprobar/documentar una actualización se fija `NEW_UPSTREAM_BASELINE_COMMIT`.
3. El valor anterior se conserva en [../VERSIONING/AUDIT_HISTORY.md](../VERSIONING/AUDIT_HISTORY.md) y CHANGELOG.
4. **Nunca sobrescribir silenciosamente** el baseline: toda rotación queda registrada con fecha, commit anterior y nuevo.