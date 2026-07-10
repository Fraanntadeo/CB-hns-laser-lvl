Propuesta de arquitectura modular para HNS + LVL (Clan Busters Legacy)

Objetivo: conservar la experiencia clásica mientras modernizamos la base de código, mejoramos seguridad y facilitamos mantenimiento.

Módulos propuestos (alto nivel):

1) `core.sma` (shim)
   - Carga y registra forwards, comandos y menús.
   - Interfaz mínima que delega a los módulos (auth, level, hud, lasers, party, carnage, premium, sql, admin_tools).

2) `auth.sma`
   - Registro/login/session management.
   - Validaciones, límites de intentos, cambio de email/pass.
   - Manejo de hashing de contraseñas (migración on-first-login).
   - Funciones públicas: `auth_register()`, `auth_login()`, `auth_change_password()`, `auth_find_by_password_unsafe()` (legacy, deshabilitable).

3) `level.sma` (existe)
   - Carga `levels.ini`.
   - Fórmula configurable para frags->nivel.
   - Nuevo: API pública `frags_required_for_level()`, `player_level_to_hp()`.

4) `hud.sma`
   - Render HUD limpio según especificación (nivel, rango, frags, frags necesarios, vida, lasers, multiplicador, happy hour).
   - Actualización periódica por `set_task`.

5) `lasers.sma` (mover desde hns_expmod_laser.sma)
   - Lógica de colocación y conteo por nivel.
   - Comandos admin: `ag_clearlasers`, `ag_clearlasers_player`.

6) `party.sma`
   - Gestión de parties, reparto de frags compartidos y aplicación de multiplicadores.

7) `premium.sma`
   - Multiplicadores y vencimientos.
   - Hook para Happy Hour.

8) `carnage.sma`
   - Modo Carnage: selección, inicio/cancel admin, puntos y compra de habilidades.

9) `sql.sma` (existe)
   - Helpers: escape, thread wrapper, migrate frags, sql_update_frags.
   - Centralizar todas las consultas SQL relevantes.

10) `admin_tools.sma`
    - Comandos y menús para administradores (forzar/cancelar carnage, limpiar láseres, logs).

Fases de migración (sprint plan):
- Fase 0: Backups y entorno staging.
- Fase 1: Rebranding y seguridad mínima (hecho parcialmente).
- Fase 2: Centralizar SQL helpers y sanear todas las consultas (parches aplicados parcialmente).
- Fase 3: Migración EXP->FRAGS (SELECTs, UPDATEs, DB ALTER y copia).
- Fase 4: Extraer módulos `auth.sma`, `hud.sma`, `lasers.sma`.
- Fase 5: QA, pruebas en staging, y despliegue gradual.

Lista de archivos a migrar/crear (prioridad alta):
- `addons/amxmodx/scripting/hns_expmod.sma` (refactor en core + dividir)
- `addons/amxmodx/scripting/auth.sma` (nuevo)
- `addons/amxmodx/scripting/hud.sma` (nuevo)
- `addons/amxmodx/scripting/lasers.sma` (mover)
- `addons/amxmodx/scripting/party.sma` (nuevo)
- `addons/amxmodx/scripting/premium.sma` (nuevo)
- `addons/amxmodx/scripting/carnage.sma` (nuevo)
- `addons/amxmodx/scripting/sql.sma` (endurecer y centralizar)

Tareas de seguimiento inmediatas (para ejecutar ahora):
- Completar reemplazo de todas las cadenas "exp" por "frags" (UI y logs).
- Marcar `p_exp` como deprecated (comentario) y reemplazar usos por `p_frags` donde aplique.
- Implementar `auth.sma` y migración de contraseñas.

Tiempo estimado por fase (rough):
- Auditoría completa y parches de seguridad: 2-4 horas
- Migración SELECT/UPDATE y pruebas locales: 2-3 horas
- Implementación de `auth.sma` y migración de contraseñas: 3-5 horas
- Modularización y refactor completo: 8-16 horas

Fin del documento.
