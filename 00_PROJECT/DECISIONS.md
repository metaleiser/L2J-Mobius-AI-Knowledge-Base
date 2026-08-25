# DECISIONS

**Última actualización**: 2026-08-25 (KB v2.0)

Registro de decisiones de proyecto. Cronológico (descendente). Cada entrada indica fecha, decisión, contexto y estado.

---

## D-003 — 2026-08-25 · Establecer arquitectura de 4 entidades del workspace

- **Decisión**: Definir formalmente 4 entidades con roles distintos:
  - `SERVER_SOURCE` = `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive` (código, Git GitLab, baseline `e2518ab`)
  - `SERVER_RUNTIME` = `L2J_Mobius_CT_2.6_HighFive` (servidor desplegado/observable, build 26/05/2024, sin Git)
  - `CLIENT` = `Lineage2-TCT-273-client`
  - `KNOWLEDGE_BASE` = `AI_KNOWLEDGE_BASE`
- **Contexto**: La auditoría SOURCE↔SERVER demostró que el runtime y el source no son idénticos ni representan el mismo commit.
- **Consecuencia**: Cuando SOURCE y RUNTIME difieran → NO corregir automáticamente ninguno; **registrar la discrepancia** con su contexto/versionado. Nunca mezclar SOURCE/RUNTIME/CLIENT sin indicar procedencia.
- **Estado**: ACTIVA (aprobada por el usuario).

---

## D-002 — 2026-08-25 · KB v2.0 = actualización, no reconstrucción

- **Decisión**: La KB evoluciona a v2.0 **preservando** el conocimiento histórico. No se reconstruye desde cero, no se elimina investigación anterior, no se reemplazan documentos por resúmenes.
- **Contexto**: Se aprobó la actualización KB v2.0 tras la auditoría.
- **Estado**: ACTIVA.

---

## D-001 — 2026-08-24 · KB v1.0 sobre baseline `e2518ab`

- **Decisión**: Documentar la KB sobre el commit `e2518ab…` del repo oficial, con baseline upstream y sistema AUDIT/VERSIONING.
- **Estado**: SUPERSEDIDA por D-002/D-003 (mantenida como histórico).
