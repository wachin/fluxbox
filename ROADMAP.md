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
[x] Se verificó con `./configure` que el árbol genera `Makefile` y que existe objetivo `uninstall`.
[x] Se compiló el proyecto parcheado con `make -j4`.
[ ] Probar el binario parcheado en una sesión real de Fluxbox.
[x] Construir un paquete `.deb` desde este repositorio con el `debian/` ya añadido.

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

[x] Ejecutar `./configure` en este repositorio.
[x] Confirmar que se generan `Makefile` y reglas de `install`/`uninstall`.
[x] Ejecutar `make -j"$(nproc)"`.
[x] Construir el paquete `.deb` con `dpkg-buildpackage -b -us -uc`.
[ ] Instalar el `.deb` generado y probarlo en una sesión real de Fluxbox.

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
[x] Ese directorio `debian/` fue añadido manualmente desde el archivo de empaquetado MX Linux:
[x] https://mxrepo.com/mx/repo/pool/main/f/fluxbox/fluxbox_1.3.7+git20220731-0mx23+1.debian.tar.xz
[x] Revisar que el `debian/` añadido sea coherente con este árbol fuente exacto a nivel básico:
[x] `dpkg-buildpackage` alcanza `dpkg-source --before-build` y aplica correctamente `debian/patches/series`.
[x] El árbol vuelve a quedar limpio con `dpkg-source --after-build .`.
[ ] Revisar `debian/patches/series` para decidir si el parche nuevo debe añadirse allí como patch Debian formal.
[ ] Decidir si conviene mantener el cambio directamente en `src/Window.cc` o moverlo a `debian/patches/`.
[x] Ejecutar `dpkg-buildpackage -b -us -uc` con todas las build-deps instaladas.
[x] Resolver build-deps faltantes detectadas en esta máquina.
[x] Verificar con `dpkg-checkbuilddeps` que ya no faltan dependencias de construcción.
[x] Generar `../fluxbox_1.3.7+git20220731-0mx23+1_amd64.deb`.
[x] Generar `../fluxbox-dbgsym_1.3.7+git20220731-0mx23+1_amd64.deb`.
[ ] Instalar el `.deb` generado y probarlo en una sesión real.

## Resultado de verificación en esta máquina

[x] `./configure` terminó bien.
[x] El objetivo `uninstall` existe (`make -n uninstall`).
[x] `make -j4` terminó bien y generó el binario `./fluxbox`.
[x] `dpkg-buildpackage -b -us -uc` arranca correctamente con el `debian/` importado.
[x] `dpkg-buildpackage -b -us -uc` terminó correctamente y generó el paquete binario.

Notas observadas durante `./configure`:

- El árbol llegó a compilar inicialmente sin `imlib2` ni `xpm` porque `configure` no encontró esas librerías opcionales.
- Para el paquete Debian/MX, `debian/control` exige las dependencias de construcción completas.
- En esta máquina ya están instaladas y verificadas; `dpkg-checkbuilddeps` no reporta faltantes.

Comando de instalación de build-deps usado/recomendado según `debian/control`:

```bash
sudo apt install autoconf automake autotools-dev bzip2 debhelper \
    dh-autoreconf libfribidi-dev libgtk2.0-dev libimlib2-dev libtool \
    libx11-dev libxext-dev libxft-dev libxinerama-dev libxpm-dev \
    libxrandr-dev libxt-dev
```

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
[x] Ejecutar `./configure`.
[x] Compilar.
[ ] Probar el parche en sesión Fluxbox real.
[x] Instalar build-deps y empaquetar en `.deb`.
[ ] Si no funciona, seguir trazando `MapRequest`, `MapNotify`, `focusRequestFromClient()` y `FocusControl`.

## Next safe step

Siguiente paso seguro:

  1. Instalar el paquete generado con `sudo dpkg -i ../fluxbox_1.3.7+git20220731-0mx23+1_amd64.deb`
  2. Cerrar sesión y volver a entrar en Fluxbox.
  3. Probar `Kate`, `Kdenlive`, `Ksnip` y `Dolphin` con `File -> Open...`.
  4. Si hiciera falta revertir, reinstalar el paquete de Fluxbox de los repositorios de MX o reconstruir/reinstalar desde este árbol.
