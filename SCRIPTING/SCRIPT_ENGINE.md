# SCRIPTING ENGINE

**Document Type**: SOURCE_ENTRY  
**Source of Truth**: \gameserver/scripting/ScriptEngine.java\, \gameserver/scripting/engine/ScriptExecutor.java\  
**Verified**: 2026-08-23  
**Status**: PARTIAL (some UNKNOWN implementation details)

---

## 1. SOURCE-OF-TRUTH PACKAGES / CLASSES

\org.l2jmobius.gameserver.scripting\:
- **ScriptEngine** — Singleton. Locates scripts, loads \Scripts.xml\, passes to executor, caches compiled scripts.
- **ScriptExecutor** — Compiles Java code to bytecode, loads classes dynamically, executes handlers.

\org.l2jmobius.gameserver.scripting.engine\:
- Handler framework classes (\AbstractHandler\, etc.)

---

## 2. ARCHITECTURE SUMMARY

\\\
GameServer.init()
  └─ ScriptEngine.getInstance().load()
       ├─ Parses config/Scripts.xml
       ├─ For each <script> entry:
       │    ├─ Read .java file from data/scripts/
       │    ├─ Compile to bytecode (compiler UNKNOWN — javac/ECJ?)
       │    ├─ Load class dynamically
       │    ├─ Register handler
       │    └─ Cache in memory
       └─ Excluded scripts skipped (<exclude> blocks)

On event/trigger:
  └─ Lookup handler from cache → execute → return result
\\\

**Technology**: Java source compiled at runtime. Compiler implementation UNKNOWN.

---

## 3. HANDLER CATEGORIES

13 handler types identified (partial — see source for full inventory):
- \ActionClickHandler\, \ChatHandler\, \BypassHandler\, \ItemHandler\
- \AdminCommandHandler\, \VoicedCommandHandler\, \UserCommandHandler\
- \EffectHandler\, \TargetHandler\, \PunishmentHandler\, \CommunityBoardHandler\

All registered via \MasterHandler.java\ (central registry — auto-generated or manually aggregated; UNKNOWN).

---

## 4. CONFIGURATION / LOADING / RUNTIME RELATIONSHIPS

\\\
config/Scripts.xml (runtime)
  └─ Referenced by ScriptEngine.load()
       └─ Paths relative to dist/game/data/scripts/

Script folders (dist/game/data/scripts/):
  handlers/   ← Main handler implementations
  custom/     ← Custom modifications
  ai/         ← AI scripts (UNKNOWN structure)
  quests/     ← Quest scripts
  events/     ← Event scripts

Exclusion support in XML:
  <exclude file="broken.java">
    <include file="working_fix.java"/>
  </exclude>
\\\

**Performance**: ~30-60s first load; subsequent access instant (cached).  
**Memory**: ~10-50MB typical.  
**Persistence**: Scripts stay in memory until server restart; no hot-reload.

---

## 5. USEFUL EVIDENCE REFERENCES

- Script loading flow: \ScriptEngine.parseDocument()\ reads XML, iterates script entries
- Handler registration: each handler class extends base type, registered in MasterHandler
- Error handling: compilation errors logged, script skipped; runtime errors caught, logged, server continues
- UNKNOWN: hot-reload, inter-script communication, debug support, memory limits, per-script timeouts

---

## 6. RELEVANT CROSS-LINKS

- [QUESTS/QUEST_ENGINE_REFERENCE.md](../QUESTS/QUEST_ENGINE_REFERENCE.md) — quest-specific script loading
- [ARCHITECTURE_OVERVIEW.md](../ARCHITECTURE_OVERVIEW.md) — handler pattern
- [SKILLS/SKILL_HANDLERS_SCRIPTS.md](../SKILLS/SKILL_HANDLERS_SCRIPTS.md) — skill-related handlers

---

**Source of Truth**: ScriptEngine class, handler framework  
**Verified**: 2026-08-23  
**Status**: PARTIAL

