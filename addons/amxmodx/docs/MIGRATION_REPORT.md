Resumen de migración: EXP → FRAGS

Ocurrencias detectadas (búsqueda inicial):

- addons/amxmodx/scripting/hns_expmod.sma
  - `EXP_NORMAL`, `EXP_LEVEL` enums (línea ~125)
  - `p_exp` array declarado (línea ~781)
  - Múltiples consultas SQL que usan `xp_normal` en `datos` (p.ej. SELECT ORDER BY xp_normal) (~líneas 1397, 4229)
  - Mensajes/strings que mencionan "Exp" en menús y help
  - Funciones que aún usan `xp_normal` para rankings y notificaciones

- addons/amxmodx/scripting/sql.sma
  - Comentario y método `sql_migrate_frags` que copia desde `xp_normal` a `frags_total`

- addons/amxmodx/scripting/include/expmod.inc
  - Archivo de include legado (`expmod`)

- configs/premiums.ini
  - Ejemplo con autor original (reemplazado)

- configs/plugins-expmod.ini
  - Lista de plugins relacionados con expmod (no modificado)

Acciones recomendadas siguientes (fases):

1. Rebranding (completado parcialmente): reemplazar todas menciones al autor y cabeceras en archivos fuente.
2. Migración de datos y código:
   - Añadir columna `frags_total` si no existe (helper `sql_migrate_frags` ya presente).
   - Cambiar todas las consultas que usan `xp_normal` para que prioricen `frags_total` (COALESCE(frags_total, xp_normal)).
   - Renombrar variables internas (`p_exp` → `p_frags` o similar) y enums `EXP` → `FRAGS` (en fases para no romper runtime).
3. Seguridad:
   - Reforzar almacenamiento de contraseñas (hash) — implementar migración on-first-login.
   - Asegurar todas las consultas con `escape_sql_string` o `SQL_PrepareQuery`.
4. Refactor:
   - Extraer `auth.sma`, `hud.sma`, `lasers.sma`, `party.sma`, `premium.sma`, `carnage.sma`.
   - Mantener compatibilidad con ReHLDS y AMX Mod X 1.10.

Tareas identificadas para parches:

- Actualizar SELECT/ORDER BY que usan `xp_normal` en `hns_expmod.sma`.
- Reemplazar mensajes de HUD que mencionan "Exp".
- Renombrar includes o adaptarlos si `include/expmod.inc` está obsoleto.
- Auditar plugins listados en `plugins-expmod.ini`.

Notas:

- Ya apliqué parches de seguridad para consultas críticas y escapado en `hns_expmod.sma` y `sql.sma`.
- Antes de correr `sql_migrate_frags`, crear backup de `addons/amxmodx/data/sqlite3/cuentas.sq3`.

Próximo bloque de trabajo: enumerar líneas exactas donde aparece `xp_normal` y generar patches para convertir SELECTs a `COALESCE(frags_total, xp_normal)` e iniciar renombrado de variables en una fase controlada.
