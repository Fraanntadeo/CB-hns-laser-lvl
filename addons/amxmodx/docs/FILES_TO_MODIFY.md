Lista de archivos y cambios propuestos

1) addons/amxmodx/scripting/hns_expmod.sma
   - Rebranding: cabecera, `register_plugin` ya modificado.
   - Seguridad: escapado de queries (aplicado en puntos críticos).
   - Migración EXP->FRAGS: reemplazar SELECTs por COALESCE(frags_total, xp_normal) (parcialmente aplicado).
   - Reemplazar strings UI: "Exp" -> "Frags" (parcialmente aplicado en top3/top15).
   - Revisar y actualizar todos los `UPDATE datos SET xp_normal=...` para sincronizar `frags_total`.
   - Extraer funciones: auth (registro/login), hud, lasers, party, premium, carnage.
   - Eliminar/mark deprecated: `p_exp` y `include/expmod.inc`.

2) addons/amxmodx/scripting/sql.sma
   - Añadir/hardening: `escape_sql_string`, `sql_thread_query_safe`, `sql_update_frags` (ya aplicados).
   - Añadir backups y procedimientos de migración seguros (`sql_migrate_frags`).

3) addons/amxmodx/scripting/hns_expmod_laser.sma
   - Rebranding cabecera (aplicado).
   - Revisar dependencias a `expmod` y consolidar en `lasers.sma`.

4) addons/amxmodx/scripting/hns_expmod_clases.sma
   - Rebranding cabecera (aplicado).
   - Revisar uso de clases/rangos y migrar parámetros a `rangos.ini`.

5) addons/amxmodx/scripting/level.sma
   - Cabecera aplicada.
   - Confirmar fórmula configurable y exponer APIs.

6) configs/*.ini
   - Consolidar valores: `happyhour.ini`, `levels.ini`, `laser.ini`, `premium.ini`, `party.ini`, `carnage.ini`, `admins.ini`, `rangos.ini`, `detonadora.ini`.
   - Mover textos hardcode y multiplicadores.

7) Otros/legacy
   - `parachute.sma`: no tocar (tercero-party) salvo rebranding si se desea.
   - `include/expmod.inc`: revisar y eliminar si es legacy.

Notas:
- Prioridad alta: sincronizar `frags_total` en todos los puntos donde actualmente se actualiza `xp_normal`.
- Prioridad alta: completar reemplazo de mensajes de UI de "Exp" a "Frags" para consistencia.

Siguiente acción: empezar a preparar parches por archivo (empezando por `hns_expmod.sma` restante), aplicando cambios no disruptivos y manteniendo compatibilidad hasta que la migración de DB esté lista.
