# Hum — Arquitectura (Mac v1)

> Objetivo técnico: datos correctos, explicados y baratos de obtener. Hum no puede gastar lo que critica.

---

## 1. Forma de la app

- **App de barra de menú** (sin ícono en el Dock por defecto). Al abrir la ventana principal aparece en el Dock y en ⌘+Tab; al cerrarla, vuelve a vivir solo en la barra.
- **SwiftUI** para toda la interfaz: `MenuBarExtra` (estilo ventana) para el popover y `Window` para la ventana principal.
- **macOS 14+**, Apple Silicon como prioridad (Intel funciona, pero no se optimiza).
- **Sin sandbox y fuera de la App Store:** inspeccionar y cerrar procesos de otras apps no es compatible con el sandbox.

## 2. Módulos

| Módulo | Qué hace | Depende de la UI |
|---|---|---|
| `HumCore` | Lectura de datos del sistema (procesos, CPU, memoria, batería, energía, disco) y acciones (cerrar, forzar cierre, expulsar). Swift puro. | No |
| `HumEngine` | Muestreo, cálculo de deltas, agrupación de procesos en apps, detección de problemas y frase de estado. | No |
| `HumContent` | Textos ES/EN y explicaciones de procesos, en archivos de contenido. | No |
| `HumApp` | Interfaz SwiftUI: popover, ventana, ícono de la barra, onboarding, ajustes. | Sí |

`HumEngine` y `HumContent` definen el **modelo y la lógica** que luego se replican en Windows; por eso no saben nada de SwiftUI.

## 3. De dónde sale cada dato

Solo APIs públicas, salvo que se indique. Las APIs privadas se aíslan en `HumCore` detrás de una interfaz, para cambiarlas sin tocar el resto.

| Dato | Fuente | Notas |
|---|---|---|
| CPU total | `host_processor_info` / `host_statistics` | Delta entre muestras. |
| Memoria y presión | `host_statistics64` (`vm_statistics64`) + `DispatchSource.makeMemoryPressureSource` | La presión (normal / advertencia / crítica) es lo que se muestra, no los "GB usados". |
| Swap | `sysctl vm.swapusage` | |
| Lista de procesos | `proc_listallpids` + `proc_pidinfo` | Nombre, ruta, usuario, PID padre. |
| CPU y memoria por proceso | `proc_pid_rusage` (tiempo de CPU, `phys_footprint`) | `phys_footprint` es lo que muestra el Monitor de Actividad como "Memoria". |
| Energía por proceso | Contadores de energía de `proc_pid_rusage` | **Aproximada.** El "Impacto energético" exacto del Monitor de Actividad usa APIs privadas. Se valida en el spike. |
| Agrupar procesos en apps | `NSWorkspace.runningApplications` + ruta del ejecutable dentro del `.app` | Chrome Helper vive dentro de `Google Chrome.app` → se agrupa con Chrome. |
| App que no responde | `CGSEventIsAppUnresponsive` (**API privada**, la que usa macOS) | Sin permisos e instantánea. Puede romperse con una actualización de macOS → respaldo opcional con Accesibilidad (ver §8). |
| Cerrar / forzar cierre | `NSRunningApplication.terminate()` / `forceTerminate()`; `kill` para procesos sin interfaz | Procesos de otro usuario o del sistema → candado + motivo. |
| Batería: %, cargando, tiempo restante | `IOPSCopyPowerSourcesInfo` | Si no hay fuente de batería → Mac sin batería. |
| Batería: vatios, ciclos, capacidad, temperatura | IORegistry, `AppleSmartBattery` (solo lectura) | Vatios = voltaje × amperaje. |
| Qué impide dormir | `IOPMCopyAssertionsByProcess` | Equivalente a `pmset -g assertions`. |
| Límite de carga / Carga optimizada | Por confirmar | **Riesgo.** Si no hay forma pública de leerlo, solo se muestra el enlace a Ajustes del Sistema. |
| Discos | `FileManager.mountedVolumeURLs` + valores de recurso | Nombre, capacidad, disponible, extraíble. |
| Expulsar | `NSWorkspace.unmountAndEjectDevice` (o DiskArbitration) | |
| App que bloquea el disco | `proc_pidinfo` con `PROC_PIDLISTFDS` + ruta de cada archivo abierto | Equivalente a `lsof` sin lanzar procesos. Sin Acceso total al disco puede no ver todo → se dice. |

## 4. Muestreo y presupuesto de consumo

Un solo reloj central, con tolerancia para que macOS agrupe los despertares.

| Situación | Frecuencia |
|---|---|
| Ventana principal abierta | 1–2 s |
| Popover abierto | 2 s |
| Todo cerrado (solo ícono de la barra) | 10 s, solo métricas globales |
| Con batería baja o modo Bajo consumo | 30 s |

- Con todo cerrado **no se recorre la lista completa de procesos** en cada muestra; solo lo necesario para la onda y para detectar alertas.
- El ranking de energía se acumula en segundo plano con muestreo lento.

**Presupuesto (se mide en la Fase 3):**
- CPU en reposo < ~1 %.
- Memoria < ~60 MB.
- Tamaño de la app < ~15 MB (incluida la fuente).

## 5. Datos guardados

- Todo local, en `~/Library/Application Support/Hum/`.
- **Energía desde la última carga:** Hum guarda cuándo se desconectó el cargador y acumula la energía por app desde ese momento. Limitación: si Hum no estaba abierta al desconectar, el ranking empieza cuando se abre (y se dice en la UI).
- Preferencias en `UserDefaults`.
- **Cero red:** Hum no se conecta a internet. Sin telemetría, sin analíticas.

## 6. Permisos

| Permiso | ¿Se pide en v1? | Para qué |
|---|---|---|
| Ninguno | — | Todo lo básico funciona sin permisos. |
| Acceso total al disco | Opcional, en contexto | Ver más archivos abiertos al buscar qué bloquea un disco. |
| Accesibilidad | No | Solo como respaldo opcional si la API privada deja de funcionar. |
| Notificaciones | No (v3) | |

## 7. Distribución sin costo

- **Firma ad-hoc** (`codesign --sign -`), gratuita. Sin notarización.
- **GitHub Actions** (runner de macOS): compila, firma ad-hoc, crea el `.dmg` y publica el release con su **checksum SHA-256**.
- **Builds reproducibles:** cualquiera puede compilar desde el código y comparar.
- **Primera apertura:** el README y la web explican, con capturas, el paso por Ajustes del Sistema → Privacidad y seguridad → "Abrir igualmente".
- **Actualizaciones en v1:** manuales desde GitHub Releases. *(Decisión pendiente: un aviso de nueva versión implicaría conectarse a internet; se evalúa después de v1 y sería opcional).*
- *Opcional:* tap propio de Homebrew.

## 8. Riesgos y spikes (antes de la Fase 2)

Pruebas técnicas cortas para confirmar que se puede antes de construir:

1. **App que no responde.** Confirmar que `CGSEventIsAppUnresponsive` funciona bien en macOS 26 (coincide con lo que marca el Monitor de Actividad). Respaldo: mensajes de Accesibilidad con tiempo límite, solo con permiso del usuario. Descartado: registros del sistema (macOS solo registra la falta de respuesta después de un forzar cierre).
2. **Energía por proceso.** Comparar los contadores de `proc_pid_rusage` con el Monitor de Actividad y definir cómo se presenta ("alto / medio / bajo" en vez de un número falso de preciso).
3. **Límite de carga.** Buscar una forma pública de leer su estado en macOS 26.
4. **Qué bloquea el disco** sin Acceso total al disco: qué tanto se ve.
5. **Consumo de Hum** con el muestreo propuesto.

## 9. Equivalencias para Windows (Fase 7)

Mismo modelo y lógica, otras fuentes:

| Dato | Windows |
|---|---|
| CPU, memoria, procesos | `NtQuerySystemInformation` / PDH, `GetProcessMemoryInfo` |
| Agrupar procesos | Árbol de procesos + ventanas; servicios dentro de `svchost` vía Service Control Manager |
| App que no responde | `IsHungAppWindow` (pública) |
| Batería | `GetSystemPowerStatus` + WMI / Battery IOCTL |
| Qué impide dormir | Power Requests (equivalente a `powercfg /requests`) |
| Qué bloquea un disco | **Restart Manager API** (pública, hecha para esto) |
| Energía por proceso | Estimación propia (como en Mac) |

---

## Decisiones abiertas

- Aviso de actualizaciones (después de v1).
