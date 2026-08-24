# UPDATE_PROTOCOL — Actualización de la KB tras cambios de Mobius

Última revisión: 2026-08-24.

## Flujo obligatorio

```
UPSTREAM UPDATE
   ↓
FETCH            (solo lectura, en el clone)
   ↓
IDENTIFY NEW COMMIT
   ↓
COMPARE COMMITS
   ↓
INVENTORY CHANGES     (name-status / stat)
   ↓
IMPACT ANALYSIS       (documentos potencialmente obsoletos)
   ↓
KB AUDIT              (verificación puntual con evidencia)
   ↓
PROPOSED CHANGES      (qué cambió · qué docs afecta · evidencia · recomendación)
   ↓
USER APPROVAL         ← obligatorio
   ↓
DOCUMENTATION UPDATE  (solo lo demostrado)
   ↓
VALIDATION            (checklist FASE G de AUDIT_PROTOCOL)
   ↓
NEW KB VERSION        (según política de KB_VERSION.md)
   ↓
CHANGELOG             (entrada concisa + AUDIT_HISTORY)
```

## Reglas duras

1. La IA **NO actualiza la KB automáticamente** porque exista un commit nuevo upstream.
2. Antes de tocar nada debe presentar: qué cambió, qué documentos podrían quedar obsoletos, qué evidencia encontró, qué recomienda modificar.
3. `git pull/merge/rebase/reset/checkout` destructivos: prohibidos sin autorización explícita.
4. Tras aprobar y documentar una actualización se establece `NEW_UPSTREAM_BASELINE_COMMIT` ([CHANGE_DETECTION.md](CHANGE_DETECTION.md)); nunca sobrescribir silenciosamente el baseline anterior.
5. Cada actualización aplicada cierra con: validación completa + entrada en [../VERSIONING/CHANGELOG.md](../VERSIONING/CHANGELOG.md) + registro en [../VERSIONING/AUDIT_HISTORY.md](../VERSIONING/AUDIT_HISTORY.md) + versión nueva solo si la política lo exige.

## Formato del informe previo (UPSTREAM UPDATE REPORT)

```
Current local commit: <hash>
New upstream commit: <hash>
Commits nuevos: <lista corta>
Archivos modificados: <por área>
Sistemas afectados: <áreas KB>
Documentos KB afectados: <lista con afirmación en riesgo>
Documentos nuevos potencialmente necesarios: <solo si superan el criterio anti-basura>
Riesgo: ALTO | MEDIO | BAJO
Recomendación: <acción propuesta>
ACCIÓN REQUERIDA: APROBAR / RECHAZAR
```