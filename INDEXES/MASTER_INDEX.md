# MASTER INDEX

**Purpose**: Routing-focused index for all AI queries about L2J Mobius CT 2.6 HighFive.  
**Last Updated**: 2026-08-26 (Sprint 0.6B)  
**Status**: VERIFIED

> ✅ = existing document · ⧗ = planned (not yet created) · Links relative to \AI_KNOWLEDGE_BASE/\

---

## 1. SYSTEM & MAINTENANCE

| Route | Documents |
|-------|-----------|
| AI entry, rules, protocols | [AI_INSTRUCTIONS/AI_README.md](../AI_INSTRUCTIONS/AI_README.md), [VERIFICATION_RULES.md](../AI_INSTRUCTIONS/VERIFICATION_RULES.md), [AUDIT_PROTOCOL.md](../AI_INSTRUCTIONS/AUDIT_PROTOCOL.md), [UPDATE_PROTOCOL.md](../AI_INSTRUCTIONS/UPDATE_PROTOCOL.md), [CHANGE_DETECTION.md](../AI_INSTRUCTIONS/CHANGE_DETECTION.md) |
| KB versioning & changelog | [VERSIONING/KB_VERSION.md](../VERSIONING/KB_VERSION.md), [UPSTREAM_BASELINE.md](../VERSIONING/UPSTREAM_BASELINE.md), [CHANGELOG.md](../VERSIONING/CHANGELOG.md), [AUDIT_HISTORY.md](../VERSIONING/AUDIT_HISTORY.md) |
| Gaps & coverage | [GAPS.md](../GAPS.md) — check COVERED/PARTIAL/MISSING before asserting |
| Source vs Runtime | [SOURCE_VS_RUNTIME.md](../SOURCE_VS_RUNTIME.md) ✅ |

---

## 2. PROJECT & WORKSPACE

| Route | Documents |
|-------|-----------|
| Project overview (4 entities) | [00_PROJECT/PROJECT_CONTEXT.md](../00_PROJECT/PROJECT_CONTEXT.md) ✅ |
| Reference sources (repos, wiki, client) | [00_PROJECT/REFERENCE_SOURCES.md](../00_PROJECT/REFERENCE_SOURCES.md) ✅ |
| Decisions & roadmap | [00_PROJECT/DECISIONS.md](../00_PROJECT/DECISIONS.md) ✅, [00_PROJECT/ROADMAP.md](../00_PROJECT/ROADMAP.md) ✅ |
| Future ideas | [00_PROJECT/IDEAS.md](../00_PROJECT/IDEAS.md) ✅ |
| Build & deployment | [BUILD_AND_DEPLOYMENT.md](../BUILD_AND_DEPLOYMENT.md) ✅ — SOURCE_ENTRY (Ant, not Gradle) |
| KB overview | [README.md](../README.md), [PROJECT_OVERVIEW.md](../PROJECT_OVERVIEW.md), [PROJECT_STRUCTURE.md](../PROJECT_STRUCTURE.md) |

---

## 3. ARCHITECTURE

| Route | Documents |
|-------|-----------|
| High-level system architecture | [ARCHITECTURE_OVERVIEW.md](../ARCHITECTURE_OVERVIEW.md) ✅ |
| GameServer detail | [GAMESERVER_ARCHITECTURE.md](../GAMESERVER_ARCHITECTURE.md) ✅ |
| LoginServer detail | [LOGINSERVER_ARCHITECTURE.md](../LOGINSERVER_ARCHITECTURE.md) ✅ |
| Commons library | [COMMONS_ARCHITECTURE.md](../COMMONS_ARCHITECTURE.md) ✅ |

---

## 4. DOMAIN DOCUMENTS

| Domain | Key Documents |
|--------|---------------|
| **NETWORK/** | [NETWORK_ARCHITECTURE.md](../NETWORK/NETWORK_ARCHITECTURE.md) ✅ — TCP/encryption/packet layers, Netty NIO |
| **DATABASE/** | [DATABASE_ARCHITECTURE.md](../DATABASE/DATABASE_ARCHITECTURE.md) ✅ — HikariCP, schema, XML loading |
| **CONFIGURATION/** | [CONFIGURATION_SYSTEM.md](../CONFIGURATION/CONFIGURATION_SYSTEM.md) ✅ — SOURCE_ENTRY, 16 core + 44 custom configs |
| **THREADING/** | [THREADING_ARCHITECTURE.md](../THREADING/THREADING_ARCHITECTURE.md) ✅ — SOURCE_ENTRY, 3 pools |
| **SCRIPTING/** | [SCRIPT_ENGINE.md](../SCRIPTING/SCRIPT_ENGINE.md) ✅ — SOURCE_ENTRY, Java runtime compilation |
| **SKILLS/** (14 docs) | [SKILL_DATA_MODEL.md](../SKILLS/SKILL_DATA_MODEL.md), [EFFECT_SYSTEM.md](../SKILLS/EFFECT_SYSTEM.md), [SKILL_SEMANTIC_REFERENCE.md](../SKILLS/SKILL_SEMANTIC_REFERENCE.md), [SCHEME_VALIDATION.md](../SKILLS/SCHEME_VALIDATION.md) ✅, + others |
 | **GAMEPLAY/** (10 docs) | [LEVELING_AND_PROGRESSION.md](../GAMEPLAY/LEVELING_AND_PROGRESSION.md), [PARTY_PVE.md](../GAMEPLAY/PARTY_PVE.md), [PVE_CONTENT_MODEL.md](../GAMEPLAY/PVE_CONTENT_MODEL.md), [INSTANCE_SYSTEM.md](../GAMEPLAY/INSTANCE_SYSTEM.md), [RAID_BOSS_SYSTEM.md](../GAMEPLAY/RAID_BOSS_SYSTEM.md), [HUNTING_ZONES.md](../GAMEPLAY/HUNTING_ZONES.md), [TELEPORT_SYSTEM.md](../GAMEPLAY/TELEPORT_SYSTEM.md), [PVE_REWARDS_AND_LOOT.md](../GAMEPLAY/PVE_REWARDS_AND_LOOT.md), [ZONE_SYSTEM.md](../GAMEPLAY/ZONE_SYSTEM.md), [GAMEPLAY_RELATION_GRAPH.md](../GAMEPLAY/GAMEPLAY_RELATION_GRAPH.md) ✅ |
| **BUFFS/** (3 docs) | [SCHEME_BUFFER_ANALYSIS.md](../BUFFS/SCHEME_BUFFER_ANALYSIS.md), [COMMUNITY_BOARD_SCHEME_ANALYSIS.md](../BUFFS/COMMUNITY_BOARD_SCHEME_ANALYSIS.md), [SCHEME_SYSTEM_COMPARISON.md](../BUFFS/SCHEME_SYSTEM_COMPARISON.md) ✅ |
| **QUESTS/** (15 docs) | Start at [QUEST_ENGINE_REFERENCE.md](../QUESTS/QUEST_ENGINE_REFERENCE.md) ✅ |
| **COMBAT/** (7 docs) | Combat system documentation ✅ |
| **CLIENT_RESEARCH/** (4 docs) | [CLIENT_STRUCTURE.md](../CLIENT_RESEARCH/CLIENT_STRUCTURE.md), [CLIENT_ENCRYPTION.md](../CLIENT_RESEARCH/CLIENT_ENCRYPTION.md), [QUEST_PILOT_Q00001.md](../CLIENT_RESEARCH/QUEST_PILOT_Q00001.md), [CLIENT_SERVER_MAPPING.md](../CLIENT_RESEARCH/CLIENT_SERVER_MAPPING.md) ✅ |
| **PACKETS/** (8 docs) | Start at [PACKET_ARCHITECTURE.md](../PACKETS/PACKET_ARCHITECTURE.md) ✅ |
| **WORLD/** | World system documentation ✅ |
| **SYSTEMS/** | Systems documentation ✅ |
| **INDEXES/** | [MASTER_INDEX.md](../INDEXES/MASTER_INDEX.md) (this doc), [MANAGERS_INDEX.md](../INDEXES/MANAGERS_INDEX.md), [ENTITY_TYPES_INDEX.md](../INDEXES/ENTITY_TYPES_INDEX.md) ✅ |
| **REFERENCE/** | [ABNORMAL_TYPE_CATALOG.md](../REFERENCE/ABNORMAL_TYPE_CATALOG.md) ✅ — 337-value catalog |

---

## 5. CROSS-DOMAIN ROUTING

| If you need... | Go to... |
 | Leveling & progression | [GAMEPLAY/LEVELING_AND_PROGRESSION.md](../GAMEPLAY/LEVELING_AND_PROGRESSION.md) |
| Party PvE | [GAMEPLAY/PARTY_PVE.md](../GAMEPLAY/PARTY_PVE.md) |
| PvE content model | [GAMEPLAY/PVE_CONTENT_MODEL.md](../GAMEPLAY/PVE_CONTENT_MODEL.md) |
| Instance system | [GAMEPLAY/INSTANCE_SYSTEM.md](../GAMEPLAY/INSTANCE_SYSTEM.md) |
| Raid boss system | [GAMEPLAY/RAID_BOSS_SYSTEM.md](../GAMEPLAY/RAID_BOSS_SYSTEM.md) |
| Hunting zones | [GAMEPLAY/HUNTING_ZONES.md](../GAMEPLAY/HUNTING_ZONES.md) |
| Teleport / travel | [GAMEPLAY/TELEPORT_SYSTEM.md](../GAMEPLAY/TELEPORT_SYSTEM.md) |
| PvE rewards & loot | [GAMEPLAY/PVE_REWARDS_AND_LOOT.md](../GAMEPLAY/PVE_REWARDS_AND_LOOT.md) |
| Zone gameplay | [GAMEPLAY/ZONE_SYSTEM.md](../GAMEPLAY/ZONE_SYSTEM.md) |
| PvE relation graph | [GAMEPLAY/GAMEPLAY_RELATION_GRAPH.md](../GAMEPLAY/GAMEPLAY_RELATION_GRAPH.md) |
| Source vs Runtime truth | [SOURCE_VS_RUNTIME.md](../SOURCE_VS_RUNTIME.md) |
|----------------|----------|
| Source vs Runtime truth | [SOURCE_VS_RUNTIME.md](../SOURCE_VS_RUNTIME.md) |
| Gaps / missing coverage | [GAPS.md](../GAPS.md) |
| Stacking rules for skills | [SKILLS/SKILL_SEMANTIC_REFERENCE.md](../SKILLS/SKILL_SEMANTIC_REFERENCE.md) §5 |
| AbnormalType values | [REFERENCE/ABNORMAL_TYPE_CATALOG.md](../REFERENCE/ABNORMAL_TYPE_CATALOG.md) |
| Quest engine & party credit | [QUESTS/QUEST_ENGINE_REFERENCE.md](../QUESTS/QUEST_ENGINE_REFERENCE.md) + [QUESTS/QUEST_PARTY_CREDIT.md](../QUESTS/QUEST_PARTY_CREDIT.md) |
| Scheme validation design | [SKILLS/SCHEME_VALIDATION.md](../SKILLS/SCHEME_VALIDATION.md) |
| Manager singleton inventory | [INDEXES/MANAGERS_INDEX.md](../INDEXES/MANAGERS_INDEX.md) |
| Entity type hierarchy | [INDEXES/ENTITY_TYPES_INDEX.md](../INDEXES/ENTITY_TYPES_INDEX.md) |
| Client file structure | [CLIENT_RESEARCH/CLIENT_STRUCTURE.md](../CLIENT_RESEARCH/CLIENT_STRUCTURE.md) |

---

## 6. VERSION & STATUS

**Knowledge Base Version**: Sprint 0.6B  
**Markdown Documents**: Measured from filesystem (active KB + ARCHIVE separate)  
**Status**: Active development — Sprint 0.6B (Compressions & Reorganizations, SOURCE_ENTRY conversions)

---

## 7. GETTING STARTED FOR NEW AI TASKS

1. [README.md](../README.md) — KB usage rules and conventions.
2. This index ([INDEXES/MASTER_INDEX.md](../INDEXES/MASTER_INDEX.md)) — locate the domain.
3. [GAPS.md](../GAPS.md) — check coverage before asserting any facts.
4. Domain document identified from sections above.
5. SOURCE_REQUIRED / MISSING → verify directly in SERVER_SOURCE (\UPSTREAM/L2J_Mobius/L2J_Mobius_CT_2.6_HighFive/\).
6. Build system is Apache Ant (not Gradle) — see [BUILD_AND_DEPLOYMENT.md](../BUILD_AND_DEPLOYMENT.md).

---

**Purpose**: Navigation guide for all AI queries  
**Status**: VERIFIED


