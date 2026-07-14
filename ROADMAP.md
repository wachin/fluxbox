# ROADMAP

Fecha: 2026-07-14

Este archivo resume lo que ya se hizo en este repositorio y lo que queda pendiente para continuar el trabajo directamente desde aquí.

## Estado actual

[x] Se revisó el árbol upstream completo de Fluxbox.
[x] Se identificó una posible causa raíz del problema con diálogos `File -> Open...` en aplicaciones Qt/KDE bajo Fluxbox en X11.
[x] Se revisó el manejo de ventanas `transient` y `modal` en `src/Window.cc`, `src/FocusControl.cc` y `src/WinClient.cc`.
[x] Se confirmó que el problema observado encaja con un diálogo modal que sí se crea, pero no siempre se eleva ni toma el foco correctamente.
[x] Se aplicó un parche mínimo en `src/Window.cc`.
[x] Se creó `README.md` en este repositorio documentando el parche y cómo probarlo.
[x] Se añadió un directorio `debian/` tomado del paquete de MX Linux para facilitar trabajo posterior de empaquetado.
[x] Se confirmó que `autogen.sh` funciona en este árbol y genera `configure` correctamente.
[ ] Verificar con `./configure` si el árbol genera `Makefile` con objetivo `uninstall` listo para usar.
[ ] Compilar el proyecto parcheado.
[ ] Probar el binario parcheado en una sesión real de Fluxbox.
[ ] Construir un paquete `.deb` desde este repositorio con el `debian/` ya añadido.

## Parche aplicado

Archivo modificado:

- `src/Window.cc`

Cambios hechos:

[x] En `mapRequestEvent()`, tratar también a las ventanas `transient` como candidatas al mismo flujo de foco usado para ventanas nuevas.
[x] En `mapNotifyEvent()`, usar `raiseAndFocus()` en lugar de `focus()` cuando Fluxbox ya decidió que la ventana debe recibir foco.

Objetivo del parche:

- mejorar el comportamiento de diálogos modales/transitorios,
- especialmente en `Kate`, `Kdenlive`, `Ksnip`, `Dolphin` y aplicaciones Qt/KDE similares,
- evitando que el diálogo exista pero quede oculto, tapado o sin foco aparente.

## Trabajo inmediato recomendado

[ ] Ejecutar `./configure` en este repositorio.
[ ] Confirmar que se generan `Makefile` y reglas de `install`/`uninstall`.
[ ] Ejecutar `make -j"$(nproc)"`.
[ ] Si compila, probar primero con instalación local controlada.
[ ] Si la prueba local mejora el problema, construir después un `.deb`.

## Pruebas funcionales pendientes

[ ] Iniciar sesión en Fluxbox con el binario parcheado.
[ ] Abrir `kate` y probar `File -> Open...`.
[ ] Repetir con `kdenlive`.
[ ] Repetir con `ksnip`.
[ ] Repetir con `dolphin`.
[ ] Comprobar si el diálogo aparece inmediatamente, visible y con foco.
[ ] Verificar si sigue existiendo congelamiento aparente de más de varios segundos.
[ ] Comparar el comportamiento con y sin reglas `[transient]` en `~/.fluxbox/apps`.

## Empaquetado Debian / MX

[x] El repositorio ya tiene un directorio `debian/`.
[ ] Revisar que el `debian/` añadido sea coherente con este árbol fuente exacto.
[ ] Revisar `debian/patches/series` para decidir si el parche nuevo debe añadirse allí como patch Debian formal.
[ ] Decidir si conviene mantener el cambio directamente en `src/Window.cc` o moverlo a `debian/patches/`.
[ ] Ejecutar `dpkg-buildpackage -b -us -uc` cuando el árbol esté listo.
[ ] Instalar el `.deb` generado y probarlo en una sesión real.

## Mejoras técnicas futuras para Fluxbox en 2026

Estas no están hechas todavía. Son líneas de trabajo para una revisión más amplia del proyecto.

[ ] Revisar compatibilidad moderna con Qt5/Qt6 y aplicaciones KDE actuales.
[ ] Revisar foco, elevación y stacking de transients/modals en más rutas aparte de `Window.cc`.
[ ] Añadir pruebas automáticas específicas para `WM_TRANSIENT_FOR`, modales y diálogos.
[ ] Revisar comportamiento con portales (`xdg-desktop-portal`) y diálogos modernos.
[ ] Revisar integración con `systemd --user`, `dbus-update-activation-environment` y sesiones X11 actuales.
[ ] Revisar variables de entorno que pueden influir en Qt/KDE dentro de sesiones mínimas.
[ ] Auditar dependencias y macros antiguas de `autoconf`/`automake` que ya muestran advertencias.
[ ] Revisar `AC_LANG_CPLUSPLUS`, `AC_HEADER_STDC`, `AC_CHECK_LIBM`, `AC_TYPE_SIGNAL`, `AC_CONFIG_HEADER` y otras macros obsoletas.
[ ] Evaluar modernización del sistema de build.
[ ] Revisar soporte HiDPI, temas e iconos modernos.
[ ] Revisar comportamiento con varios monitores y setups XRandR actuales.
[ ] Revisar interacción con docks, systray, notificaciones y aplicaciones recientes.
[ ] Revisar errores históricos en `ChangeLog` relacionados con transients para detectar patrones no resueltos.
[ ] Revisar si existen bugs similares reportados recientemente aguas abajo en Debian, antiX o MX.

## Observaciones útiles para retomar luego

[x] El árbol original upstream no traía `debian/`; ese directorio fue añadido después.
[x] `README.md` ya documenta el parche aplicado y la lógica técnica detrás del cambio.
[x] El problema del usuario original se reproduce como una falsa congelación: la aplicación queda esperando el diálogo modal, pero la ventana puede no quedar visible o enfocada.
[ ] Confirmar con pruebas reales si este parche basta o si hay más rutas de foco implicadas.

## Siguiente paso sugerido al abrir Codex directamente aquí

[ ] Abrir Codex en `/home/wachin/Dev/fluxbox-dev/fluxbox`.
[ ] Ejecutar `./configure`.
[ ] Compilar.
[ ] Probar el parche en sesión Fluxbox real.
[ ] Si funciona, empaquetar en `.deb`.
[ ] Si no funciona, seguir trazando `MapRequest`, `MapNotify`, `focusRequestFromClient()` y `FocusControl`.
