# HNS + LVL — Arquitectura (resumen)

Proyecto: HNS + LVL (Clan Busters Legacy)
Autor: Bstr # Thynuviel (este repositorio)

## Objetivo

Mantener la experiencia clásica de HNS + LVL, modernizando el código, centralizando configuración y migrando el progreso a "frags".

## Estructura principal

- addons/amxmodx/scripting/hns_expmod.sma
  - Núcleo monolítico: init, menús, HUD, registración de forwards/ham, muchas funciones de gameplay.
- addons/amxmodx/scripting/hns_expmod_laser.sma
  - Lógica de despliegue, beam y contadores de láser.
- addons/amxmodx/scripting/hns_expmod_clases.sma
  - Definición de clases/rangos (debe ser convertida a rangos sólo descriptivos).
- addons/amxmodx/scripting/level.sma (nuevo)
  - Carga `levels.ini` y lógica central de `level_check(id)` (migrado desde `check`).
- addons/amxmodx/scripting/include/expmod.inc
  - Forwards y nativos compartidos.
- addons/amxmodx/configs/\*.ini
  - Configs centralizadas: `levels.ini`, `laser.ini`, `premium.ini`, `party.ini`, `carnage.ini`, `happyhour.ini`, `admins.ini`.
- DB: addons/amxmodx/data/sqlite3/cuentas.sq3
  - Contiene tablas `datos`, `estadisticas`, etc. Actualmente `xp_normal` se usa como compatibilidad para frags.

## Sistemas y comportamiento

1. Registro / Persistencia

- `client_putinserver`, `Cargar`, `Guardar` manejan carga y guardado via SQLx.
- `Guardar` actualiza columnas: actualmente escribe `xp_normal` con `p_frags[][FRAGS_TOTAL]` para compatibilidad.
- Comando `/ag_migratefrags` añade columna `frags_total` (ALTER TABLE) y ejecuta un UPDATE-copy.

2. Niveles / Progreso

- Progreso único basado en frags (`p_frags[][FRAGS_TOTAL]`).
- `frags_required_for_level(index, level)` usa parámetros de `levels.ini` para calcular frags requeridos.
- `level_check(id)` (nuevo) centraliza la subida de nivel y efectos (sonido, efecto visual, Guardar, forward `g_fwLevel`).

3. Vida

- Vida por nivel: implementado como `100 + (nivel - 1)` en spawn.
- Las clases ya no modifican vida (las clases siguen existiendo como etiquetas/rangos).

4. Lasers

- Lógica en `hns_expmod_laser.sma`; límite por clase y bonificación por nivel (3 base para CT mínimo + floor(level/50)).
- Admins: comandos `ag_setlasers`, `ag_showlasers` creados; se requiere añadir comando para borrar lasers mal colocados (próximo).

5. Party

- `p_party_info` mantiene combos; las kills se comparten y ahora suman frags; party logic distribuido en `hns_expmod.sma`.

6. Premium

- Premium solo otorga multiplicadores de progreso (`p_mult`), no ventajas de vida/daño/armas.

7. Carnage

- Modo especial con contadores `FRAGS_CARNAGE` y canje por puntos.
- Reglas especiales (awp, deagle, etc.) mantenidas; admin puede forzar carnage (se implementará menú/command central).

## Problemas detectados (prioritarios)

- Persistencia: uso de `formatex` para SQL puede provocar errores con nombres con comillas; falta prepared queries.
- DB: `xp_normal` se usa como alias de `frags_total` (temporal). Requiere migración segura y backup.
- Inconsistencias: `p_exp` aún declarado y usado en pocas líneas; limpiar tras validación completa.
- Seguridad/validación: comandos admin no validan targets conectados o rangos; faltan logs de acciones.
- Performance: funciones pesadas en `PreThink` o run-tasks frecuentes pueden afectar en servidores con muchos jugadores.

## Malas prácticas encontradas

- Código monolítico y duplicado.
- SQL concatenado sin prepared statements.
- Validaciones insuficientes en comandos administrativos.
- Mensajes y textos dispersos (difícil de localizar para traducción/branding).

## Recomendaciones inmediatas

1. Migrar `check()` a módulo `level.sma` (completado).
2. Implementar wrapper SQL en `sql.sma` con queries parametrizadas y migración controlada.
3. Crear módulo `player.sma` para spawn/vida y centralizar la salud por nivel.
4. Refactorizar `hns_expmod_laser.sma` para leer `laser.ini` y exponer forward admin para eliminar láseres.
5. Auditar y reemplazar `p_exp` (mantener alias temporal hasta que `frags_total` esté en DB).
6. Centralizar textos en `texts.ini` para reemplazar referencias de autor y rebranding.

## Siguientes pasos propuestos (iterativos)

- Implementar `sql.sma` con prepared queries y migración DB en staging.
- Extraer `player.sma` y mover lógica de spawn/vida.
- Finalizar integración de `lasers.sma` y añadir comando admin para borrar láseres generados por jugador.
- Refactorizar HUD/menus a `hud.sma` y revisar todos los textos para eliminar referencias al autor original.
- Test en servidor de staging (10-32 jugadores) y corregir bugs de sincronía.

## Notas finales

Este documento es la base para la refactorización incremental. Procederé con las tareas acordadas por pasos pequeños, creando commits/patches legibles. Si apruebas, continúo con la implementación de `sql.sma` (prepared queries) y la migración segura de la BD en staging.
