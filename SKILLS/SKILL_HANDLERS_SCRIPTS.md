# SKILL HANDLERS (SCRIPTS)

**Proyecto**: L2J Mobius CT 2.6 HighFive  
**Capa**: Skills — handlers implementados como scripts  
**Source of Truth**: `handler/{EffectHandler,TargetHandler}.java`, `dist/game/data/scripts/handlers/{EffectMasterHandler.java, skill/effects/, skill/targets/}`  
**Verified**: 2026-08-23  
**Status**: VERIFIED

---

## 1. HECHO CLAVE

En este fork **no existe `SkillHandler`**, y las implementaciones de efectos/targeting **no están compiladas en el core**: son **scripts Java** cargados por el ScriptEngine desde el datapack.

## 2. REGISTROS DEL CORE

### `handler/EffectHandler.java`
```java
public class EffectHandler implements IHandler<Class<? extends AbstractEffect>, String>
{
    private final Map<String, Class<? extends AbstractEffect>> _handlers;
    public void registerHandler(Class<? extends AbstractEffect> handler)
    public Class<? extends AbstractEffect> getHandler(String name)
    public int size()
}
```
- Clave = nombre del efecto tal como aparece en el XML (`<effect name="PhysicalDamage">`).
- Valor = clase que extiende `AbstractEffect` (definida en un script).

### `handler/TargetHandler.java`
- Patrón equivalente para targeting (clase handler por TargetType).

## 3. SCRIPTS QUE ALIMENTAN LOS REGISTROS

### Efectos — 165 archivos
- Ubicación: `dist/game/data/scripts/handlers/skill/effects/*.java`
- Ejemplos verificados: `AddHate.java`, `ApplySkillEffects.java`, `AttackTrait.java`, `Backstab.java`, `Betray.java`, …

### Targets — 34 archivos
- Ubicación: `dist/game/data/scripts/handlers/skill/targets/*.java`

### Registro maestro
- `dist/game/data/scripts/handlers/EffectMasterHandler.java`:
```java
EffectHandler.getInstance().registerHandler((Class<? extends AbstractEffect>) c);
LOGGER.info("EffectMasterHandler: Loaded " + EffectHandler.getInstance().size() + " effect handlers.");
```
- Existe también `MasterHandler.java` (registra el resto de handlers del sistema).

## 4. FLUJO DE RESOLUCIÓN (XML → ejecución)

```
XML: <effect name="PhysicalDamage" power="#power" .../>
   ↓ DocumentSkill parsea y guarda el nombre+params
Runtime: EffectHandler.getInstance().getHandler("PhysicalDamage")
   ↓ instancia la clase script (extends AbstractEffect)
Skill.getEffects(scope) devuelve instancias listas
   ↓ applyEffects/onStart/onTick (ver EFFECT_SYSTEM.md)
```

Mismo patrón con `<targetType>` → `TargetHandler.getHandler(...)`.

## 5. CARGA Y RECARGA

- Los scripts se cargan con el **ScriptEngine** durante el arranque (fase scripts — ver SCRIPTING/SCRIPT_ENGINE.md).
- Al recargar scripts se repuebla `EffectHandler`; combinado con `SkillData.reload()` permite modificar comportamiento sin reiniciar (detalle de orden REQUIRES CODE VERIFICATION).

## 6. IMPLICACIONES PARA EDICIÓN

- Cambiar el comportamiento de un efecto concreto → editar su script en `dist/game/data/scripts/handlers/skill/effects/`.
- Añadir un nuevo efecto → crear script + registrar nombre en `EffectMasterHandler` + usarlo en XML.
- El core Java solo cambia si se altera el motor (`AbstractEffect`, `BuffInfo`, `EffectList`, `Skill`).

## 7. TRAZABILIDAD

| Elemento | Path |
|----------|------|
| Registro efectos | `handler/EffectHandler.java` |
| Registro targets | `handler/TargetHandler.java` |
| Master efectos | `dist/game/data/scripts/handlers/EffectMasterHandler.java` |
| Implementaciones | `dist/game/data/scripts/handlers/skill/effects/` (165), `.../skill/targets/` (34) |
| Interfaz base | `handler/IHandler.java` |

---
**Status**: VERIFIED  
**Verified**: 2026-08-23