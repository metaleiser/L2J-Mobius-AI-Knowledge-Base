# AI_BOOTSTRAP — Mandatory first-read for any AI agent

**Project**: L2J Mobius CT 2.6 HighFive  
**KB Version**: 2.4  
**Audience**: AI agents operating on this workspace  
**Status**: VERIFIED

This document is the mandatory entry point for any AI session. Read it first, then follow the routing table below.

---

## 1. Workspace entities

| Entity | Path | Authority |
|---|---|---|
| **SERVER_SOURCE** | `UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive` | Code implementation and architecture |
| **SERVER_RUNTIME** | `L2J_Mobius_CT_2.6_HighFive` | Currently deployed observable state |
| **CLIENT** | `Lineage2-TCT-273-client` | Client-only resources and behavior |
| **KNOWLEDGE_BASE** | `AI_KNOWLEDGE_BASE` | Previously investigated knowledge |

**Baseline**: `SERVER_SOURCE` is at upstream commit `e2518ab10872b28cd4c6860e102b493656ba8728`.  
**Runtime build**: `26/05/2024`. `SERVER_SOURCE` and `SERVER_RUNTIME` are not identical.

The KB never replaces source code. If the KB contradicts `SERVER_SOURCE`, `SERVER_SOURCE` wins.

---

## 2. Evidence taxonomy

Canonical states from `AI_INSTRUCTIONS/VERIFICATION_RULES.md`:

| State | Meaning |
|---|---|
| **VERIFIED** | Confirmed against source, runtime or client with concrete evidence |
| **DIRECTLY_SUPPORTED** | Derives directly from cited code without extra interpretation |
| **ORIENTATION_ONLY** | Points to where to investigate; fact still requires verification |
| **SOURCE_REQUIRED** | Concrete claims must be verified in `SERVER_SOURCE` before use |
| **PARTIAL** | Some aspects verified, others not |
| **UNKNOWN** | Insufficient information |
| **ASSUMPTION** | Working hypothesis, not a fact |
| **CONFLICT** | Contradictory evidence exists |
| **DEPRECATED** | Superseded information |
| **UNKNOWN_CLIENT** | Client-side data that cannot be verified due to encryption |

---

## 3. Mandatory next-reads

After this document, read:

1. `GAPS.md` — coverage map (COVERED / PARTIAL / MISSING / SOURCE_REQUIRED).
2. `VERSIONING/KB_VERSION.md` — current version, baseline, file counts.

---

## 4. Domain routing

Load only the documents relevant to the current task.

| Task domain | Primary KB documents |
|---|---|
| Skills, buffs, effects, targeting | `SKILLS/SKILL_SEMANTIC_REFERENCE.md` |
| Scheme buffer NPC or Community Board schemes | `BUFFS/SCHEME_BUFFER_ANALYSIS.md`, `BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md`, `BUFFS/SCHEME_SYSTEM_COMPARISON.md` |
| Scheme validation design | `SKILLS/SCHEME_VALIDATOR_DESIGN.md`, `SKILLS/SCHEME_VALIDATION_LIFECYCLE.md` |
| Quests | `QUESTS/QUEST_ENGINE_REFERENCE.md` |
| Combat, damage, death | `COMBAT/COMBAT_ARCHITECTURE.md` |
| Player, items, inventory, NPCs, world | `SYSTEMS/{PLAYER,ITEM,INVENTORY,NPC,WORLD}_SYSTEM.md` |
| Packets, networking | `PACKETS/PACKET_ARCHITECTURE.md`, `NETWORK/NETWORK_ARCHITECTURE.md` |
| Database | `DATABASE/DATABASE_ARCHITECTURE.md` |
| Configuration | `CONFIGURATION/CONFIGURATION_SYSTEM.md` |
| Threading | `THREADING/THREADING_ARCHITECTURE.md` |
| Scripting | `SCRIPTING/SCRIPT_ENGINE.md` |
| Project planning, roadmap, decisions | `00_PROJECT/ROADMAP.md`, `00_PROJECT/DECISIONS.md` |
| Full document listing and metadata | `AI_INSTRUCTIONS/AI_MANIFEST.md` |

---

## 5. Source inspection rules

Inspect `SERVER_SOURCE` only when:

- The KB marks the information `SOURCE_REQUIRED`.
- The KB information is `PARTIAL`, `UNKNOWN`, `CONFLICT` or stale.
- The task explicitly requires source verification.
- The source may have changed since the KB verification baseline.

When source inspection is required, use targeted file reads and exact line ranges from the KB whenever possible. Do not load entire Java files if the KB already identifies the relevant methods and ranges.

---

## 6. Modification boundaries

| Allowed | Forbidden |
|---|---|
| Files inside `AI_KNOWLEDGE_BASE/` | `SERVER_SOURCE`, `SERVER_RUNTIME`, `CLIENT` |
| KB version, changelog, audit history | Upstream `.git` directory |
| New KB documents and cross-links | SQL schema, Java/XML/SQL/INI/config in server trees |

Do not modify server code without explicit authorization.

---

## 7. Session reset policy

Start a new session when:

- The previous task is complete.
- The domain changes substantially.
- The session accumulated excessive context.
- A new micro-sprint begins.
- The conversation contains stale assumptions.

Continue a session only for a direct continuation of the same investigation or implementation.
