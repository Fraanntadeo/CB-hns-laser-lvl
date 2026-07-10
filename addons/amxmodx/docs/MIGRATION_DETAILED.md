MIGRATION DETALLADA: EXP -> FRAGS (estado actual)

Hallazgos detectados (lista precisa):

1) `addons/amxmodx/scripting/hns_expmod.sma`:
- Líneas con SELECT que usan `xp_normal` (ya parcheadas para preferir `frags_total`):
  - Mensajes top3: "SELECT nombre, nivel, rango, xp_normal FROM datos ORDER BY xp_normal DESC LIMIT 3" -> reemplazado por COALESCE(frags_total, xp_normal).
  - Top15: "SELECT nombre, nivel, rango, xp_normal FROM datos ORDER BY xp_normal DESC LIMIT 15" -> reemplazado por COALESCE(frags_total, xp_normal).
- SELECTS de `xp_normal, xp_level` en la loteria (2 ocurrencias) -> reemplazados por `COALESCE(frags_total, xp_normal), xp_level` y añadida sincronización `sql_update_frags(...)` tras los UPDATEs.
- Varios `UPDATE datos SET xp_normal=...` siguen presentes (compatibilidad). Se propuso mantenerlos pero sincronizar `frags_total`.
- Declaración `new p_exp[33][EXP]` (posible código muerto) y asignaciones de 0 en finales del archivo — pocas referencias (3 ocurrencias). Recomendado: marcar `p_exp` como deprecated y remover tras pruebas.

2) `addons/amxmodx/scripting/sql.sma`:
- Contiene `sql_migrate_frags()` (preparada) que hace `ALTER TABLE` y copia datos desde `xp_normal` si `frags_total` es NULL/0.
- Añadida función pública `escape_sql_string` y `sql_update_frags`/`sql_thread_query_safe` usada para sincronizar `frags_total`.

3) Mensajes y textos en `hns_expmod.sma`:
- Reemplacé etiquetas visibles "Exp" por "Frags" en top3 y top15. Quedan otras cadenas que contienen "exp" en comentarios y textos (p.ej. help de lotería, menús, logros). Necesario reemplazar todas las cadenas visibles.

4) Seguridad y SQL:
- Parcheado: escapado de inputs en registro, login, Guardar() y desbaneos; uso de wrapper `sql_thread_query_safe`.

Acciones aplicadas (resumen):
- Parcheo de consultas SELECT para priorizar `frags_total` usando `COALESCE(frags_total, xp_normal)`.
- Parcheo de mensajes visibles Top3/Top15: 'Exp' -> 'Frags'.
- Parcheo de lotería: SELECT usa COALESCE y se invoca `sql_update_frags` tras actualizar `xp_normal`.
- Añadido `escape_sql_string` público en `sql.sma` y uso del mismo en puntos críticos.
- Rebranding parcial en varios archivos y actualización de `premiums.ini` (ejemplo de autor).

Siguientes pasos recomendados (priorizados):
1) Reemplazar todas las cadenas visibles y mensajes (HUD/menus/archivos HELP) que mencionan "Exp" por "Frags" (completo). Esto incluye logros, loteria, menus, HUD.
2) Auditar y modificar todas las `UPDATE datos SET xp_normal=...` para también actualizar `frags_total` (o migrar la lógica para usar `frags_total` directamente). Ya añadí sincronización en loteria y Guardar(), pero hay más ocurrencias.
3) Eliminar/depurar `p_exp` y archivo `include/expmod.inc` (marcar deprecated y luego remover tras pruebas).
4) Implementar `auth.sma` y migración de contraseñas (hash), y mover textos a configs.

Notas operativas:
- Hice cambios no disruptivos (preservando `xp_normal` para compatibilidad), pero recomiendo planificar un `sql_migrate_frags()` en staging y backup antes de mover los UPDATE principales.
- Backup recomendado: copia manual de `addons/amxmodx/data/sqlite3/cuentas.sq3`.

Estado: Parches iniciales aplicados; quedan cambios masivos de strings, renombrados y refactor grandes para completar la migración.
