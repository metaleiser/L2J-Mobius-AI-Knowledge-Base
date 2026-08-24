# SKILL LOADING

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Skills — carga de datos  
**Source of Truth**: `data/xml/SkillData.java`, `util/DocumentSkill.java`  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## 1. CLASE CARGADORA: `SkillData`

- **Path**: `java/org/l2jmobius/gameserver/data/xml/SkillData.java`
- **Package**: `org.l2jmobius.gameserver.data.xml`
- **Declaración**: `public class SkillData` (singleton estilo eager, accedido con `SkillData.getInstance()`).

### Estado interno (verificado)
```java
private final Map<Integer, Skill> _skillsByHash = new ConcurrentHashMap<>();
private final Map<Integer, Integer> _maxSkillLevels = new ConcurrentHashMap<>();
private final Set<Integer> _enchantable = ConcurrentHashMap.newKeySet();
private final List<File> _skillFiles = new ArrayList<>();
```
- La clave de `_skillsByHash` es un hash id+nivel(+enchant) — el cálculo exacto del hash vive en `Skill`/loader → detalle `REQUIRES CODE VERIFICATION`.

## 2. FLUJO DE CARGA (verificado)

```
Constructor SkillData:
   processDirectory("data/stats/skills", _skillFiles)
   processDirectory("data/stats/skills/custom", _skillFiles)
   load()

processDirectory(dirName, list):
   dir = new File(ServerConfig.DATAPACK_ROOT, dirName)
   añade todos los *.xml (case-insensitive) a la lista

load():
   para cada file → loadSkills(file) → agrega al mapa
   luego pasa por los files para calcular _maxSkillLevels / _enchantable

loadSkills(File):
   if null → LOGGER.warning("Skill file not found."); 
   DocumentSkill doc = new DocumentSkill(file);
   doc.parse() → List<Skill>
```

## 3. PARSER: `util/DocumentSkill.java`

- **Path**: `java/org/l2jmobius/gameserver/util/DocumentSkill.java`
- **Import verificado** en SkillData L40.
- Convierte el XML en instancias `Skill` (incluye `<table>` por nivel y rutas enchant1..7).
- Detalle interno completo del parseo de sub-tags (`<for>`, cond, effect anidados) → `REQUIRES CODE VERIFICATION` si se requiere inventario exhaustivo.

## 4. CUÁNDO OCURRE LA CARGA

- En el arranque del **GameServer**, durante la fase de loaders XML (ver `GAMESERVER_ARCHITECTURE.md` — fase data).
- Los **effect/target handlers script** se cargan después vía ScriptEngine (ver `SKILL_HANDLERS_SCRIPTS.md`).

## 5. RECUPERACIÓN EN RUNTIME

```java
public Skill getSkill(int skillId, int level)   // L223
```
- Usado masivamente (p.ej. `Player.addSkill(SkillData.getInstance().getSkill(4270, level))`).

## 6. RELOAD

```java
public void reload()   // L188
```
- Recarga skills **y también los Skill Trees** (comentario explícito en el código).
- Permite modificar XML sin reiniciar (vía admin/reload correspondiente).

## 7. MANEJO DE ERRORES

- Archivo nulo → warning `"Skill file not found."`.
- Errores de parseo XML → excepción propagada/logueada por el reader base (detalle REQUIRES).

## 8. TRAZABILIDAD

| Elemento | Path |
|----------|------|
| Loader | `data/xml/SkillData.java` |
| Parser | `util/DocumentSkill.java` |
| XMLs | `dist/game/data/stats/skills/*.xml` + `custom/` |
| XSD | `dist/game/data/xsd/skills.xsd` |
| Config raíz datapack | `ServerConfig.DATAPACK_ROOT` |

---
**Status**: VERIFIED  
**Verified**: 2026-08-23