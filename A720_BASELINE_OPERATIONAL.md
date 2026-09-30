# DroidDeck A720 — Baseline operativo #2

Fecha de la prueba: 2026-09-30 (America/Bogota)  
Repositorio: `dppablito4-oss/drioid-deck-A720`  
Dispositivo ADB: `A7UH025A28000522`  
Aplicación: DroidDeck `0.2.0 (9)`, `com.droiddeck.launcher`  
Resultado general: **PARTIAL**

## Resumen ejecutivo

El único prerrequisito cambiado fue:

```text
settings_enable_monitor_phantom_procs
null → false
```

Después del cambio, el preflight pasó, DroidDeck descargó e instaló su runtime `r9`, inició el compositor Wayland, PulseAudio, DirectAudio, `proot` y Gamescope. La prueba avanzó mucho más que la baseline #1, pero Steam y Big Picture no llegaron a ejecutarse.

El bloqueo confirmado ocurre en el Turnip incluido en el runtime Linux. Mesa `26.2.2` identifica `chip_id = 43020000`, `gpu_id = 6720`, pero lo rechaza como dispositivo no soportado con `VK_ERROR_INCOMPATIBLE_DRIVER`. Gamescope no encuentra ningún dispositivo Vulkan físico y termina con estado 1. El mismo fallo se reprodujo en dos intentos consecutivos.

No se modificaron código fuente, Turnip, FEX, afinidad CPU, resolución, `TU_DEBUG`, sysmem, GPU, KGSL ni controles térmicos. No se ejecutó `pm clear`, no se borraron datos, no se actualizaron fuentes y no se hicieron commits ni push.

## Comparación baseline #1 vs baseline #2

| Capa / condición | Baseline #1 | Baseline #2 |
|---|---|---|
| Phantom process monitor | `null`; bloqueaba el preflight | `false`; preflight aprobado |
| Runtime Linux | No instalado | **PASS**: `r9`, listo |
| Proton/componente asociado | No alcanzado | **PASS**: evento `proton.ready` |
| Compositor Wayland | No alcanzado | **PARTIAL**: inicia y acepta clientes, pero no encuentra dispositivo físico y presenta 0 frames |
| Turnip Android seleccionado | `Auto → turnip25.1.0`, sin sesión | `Auto → turnip25.1.0`; selección confirmada, carga funcional no demostrada |
| `proot` | No alcanzado | **PASS**: inicia |
| Turnip Linux | No alcanzado | **FAIL**: Mesa 26.2.2 rechaza Adreno 720 / GPU ID 6720 |
| Gamescope | No alcanzado | **FAIL**: 3.16.29 inicia, no crea backend Vulkan y sale con estado 1 |
| Steam | No alcanzado | **FAIL / no iniciado**: el evento `steam.starting` precede al fallo de Gamescope, sin proceso Steam observado |
| Big Picture | No alcanzado | **NO COMPROBADO**: nunca fue visible |
| FEX | No alcanzado | **NO COMPROBADO**: ningún proceso FEX llegó a ejecutarse |

## Prerrequisito cambiado

Valor registrado antes del cambio:

```text
adb shell settings get global settings_enable_monitor_phantom_procs
null
```

Único cambio aplicado:

```text
adb shell settings put global settings_enable_monitor_phantom_procs false
```

Verificación inmediata y al cierre:

```text
adb shell settings get global settings_enable_monitor_phantom_procs
false
```

Se forzó el cierre de DroidDeck, se limpió solamente `logcat` y se abrió de nuevo. No apareció otra vez el bloqueo “Steam cannot start yet / Child-process limit is unset”. `device.txt` confirma `Phantom proc monitor disabled (good)`.

## Hardware

| Campo | Valor observado |
|---|---|
| Fabricante / modelo | HONOR ELI-NX9 |
| Device / product | HNELIX / ELI-NX9 |
| Board / hardware | crow / qcom |
| SoC | QTI SM7550, Snapdragon 7 Gen 3 |
| GPU KGSL | `Adreno720` |
| Chip ID KGSL | No expuesto por DroidDeck (`unknown`) |
| Android | 16, API 36 |
| MagicOS/build | ELI-N39 `10.0.0.160(C636E5R2P1)` |
| Parche de seguridad | 2026-08-01 |
| Kernel | `5.15.180-android13-8-00018-g9b2308ac0ad6-ab14563692` |
| CPU | 8 cores |
| Techos CPU | 0–3: 1.80 GHz; 4–6: 2.40 GHz; 7: 2.63 GHz |
| RAM total | 11,602,956 kB (11.60 GB decimal; 11.07 GiB) |

Los datos de `device.txt` coinciden con las mediciones ADB de la baseline #1.

## Runtime instalado

El flujo propio de DroidDeck mostró `Downloading the Linux runtime · 165 of 790 MB` y luego `Unpacking the Linux runtime`. No se instaló ningún runtime externo.

Según `events.jsonl`:

- `runtime.installing` a las 17:54:05.481;
- `runtime.ready` a las 17:55:09.171: aproximadamente 63.7 s para descarga más desempaquetado;
- `proton.installing` inmediatamente después;
- `proton.ready` a las 17:55:36.693: aproximadamente 27.5 s adicionales;
- desde el inicio del runtime hasta el compositor: aproximadamente 91.3 s.

El tamaño anunciado fue **790 MB**. La descarga visible avanzó rápidamente; el resto del intervalo correspondió principalmente al desempaquetado. No hubo error de descarga o instalación.

Espacio en `/data`:

| Momento | Usado | Disponible |
|---|---:|---:|
| Antes | 154,462,164 kB | 77,201,452 kB |
| Después | 160,121,036 kB | 71,542,580 kB |
| Diferencia | +5,658,872 kB | −5,658,872 kB |

La ocupación final añadida por el flujo completo fue aproximadamente **5.66 GB decimal / 5.27 GiB**. `device.txt` confirma runtime `r9`, `Runtime ready true` y ruta `/data/user/0/com.droiddeck.launcher/files/linuxfs`.

## Turnip Android realmente cargado

Estado: **NO CONFIRMADO / no funcionalmente demostrado**.

`device.txt` confirma la selección:

```text
Display driver (chosen) Auto -> turnip25.1.0
name / version        turnip25.1.0
imported available    none
```

Sin embargo, esto solo prueba qué driver seleccionó la configuración. La evidencia de ejecución no permite afirmar que Turnip Android se cargó correctamente:

- el compositor registra `present: no physical devices`;
- el campo de driver queda vacío: `the compositor's driver ()`;
- solo anuncia formatos dma-buf lineales `AR24`, `XR24`, `AB24`, `XB24`;
- no encuentra un layout `qcom_compressed` importable;
- el dispositivo principal dma-buf queda como `0:0` y no hay display DRM `/dev/dri`;
- `logcat` no contiene una línea `TurnipDriver` ni una ruta de biblioteca Turnip que confirme la carga;
- al abrir la aplicación, el proceso sí registra el GLES propietario de Qualcomm (`/vendor/lib64/egl/libGLESv2_adreno.so`, versión `0676.76.3`), pero eso no prueba Turnip para el compositor.

Por tanto, **`Auto → turnip25.1.0` está seleccionado, pero Turnip Android no puede marcarse PASS**. No se cambió el driver para investigar alternativas.

## Turnip Linux realmente cargado

Estado: **FAIL confirmado**.

El proceso invitado exportó:

```text
VK_ICD_FILENAMES=/data/user/0/com.droiddeck.launcher/files/linuxfs/usr/share/vulkan/icd.d/freedreno_icd.json
```

`device.txt` lo identifica como el Turnip incluido en el runtime. El log revela Mesa `26.2.2` y confirma que el ICD fue alcanzado, pero no pudo crear un dispositivo compatible:

```text
TU: error: ../mesa-26.2.2/src/freedreno/vulkan/tu_device.cc:1683:
device (chip_id = 43020000, gpu_id = 6720) is unsupported
(VK_ERROR_INCOMPATIBLE_DRIVER)

TU: error: ../mesa-26.2.2/src/freedreno/vulkan/tu_knl.cc:402:
failed to query kernel driver version for device /dev/dri/renderD0
(VK_ERROR_INCOMPATIBLE_DRIVER)
```

El nodo Android `/dev/kgsl-3d0` fue enlazado como `/dev/dri/renderD0` dentro del runtime, tal como muestra el comando `proot`. El fallo se repitió sin variación material en las sesiones de las 17:55 y 17:59.

## TU_DEBUG

`TU_DEBUG` no aparece en el entorno completo registrado del proceso `proot`. `device.txt` confirma `Turnip sysmem false`. La baseline se ejecutó con sysmem desactivado, como se pidió.

## Gamescope

Estado: **FAIL**.

Gamescope `3.16.29` inicia y llega a conectarse varias veces al compositor Wayland, pero el Turnip Linux no ofrece un dispositivo Vulkan físico:

```text
[gamescope] vulkan: failed to find physical device
SDL_Vulkan_CreateSurface failed:
VK_KHR_wayland_surface extension is not enabled in the Vulkan instance.
Failed to create backend.
```

El invitado termina con estado 1 aproximadamente 1.2 s después de iniciar en el primer intento y 0.75 s después en el segundo. No hubo crash Java/Android, ANR ni tombstone; fue una salida controlada de la sesión invitada.

También aparece repetidamente un problema secundario:

```text
/usr/local/lib/libfakeinput.so from /etc/ld.so.preload cannot be preloaded
```

No se atribuye el bloqueo principal a `libfakeinput.so`, porque Gamescope continúa hasta fallar explícitamente en Vulkan.

## Steam

Estado: **NO INICIADO**.

`events.jsonl` registra `steam.starting`, pero inmediatamente después registra `guest.exited`, `session.stopping` y `session.failed` con código `GUEST_EXIT` y estado 1. No se observó proceso `steam`, `steamwebhelper` ni Steam ARM64 vivo. El mensaje visible final fue “Steam stopped unexpectedly”.

## Big Picture

Big Picture visible: **no**  
Pantalla negra de Steam: **no comprobable; Steam no inició**  
Frames del compositor: **0 presentados**  
Artefactos gráficos: **no comprobable**  
Parpadeos: **no comprobable**  
Touch en Big Picture: **no comprobable**  
Audio de Steam: **no comprobable**  
Freezes: **no observados; la sesión falla de inmediato**  
Crash: **salida de invitado con estado 1; sin crash Android**

`wayland.log` mantuvo el compositor vivo durante varios minutos, pero cada bloque de 10 s registró `0 on screen`, `0 presented` y ningún buffer de Gamescope. Es un resultado de cero frames, no una sesión Big Picture estable con pantalla negra.

## Resolución

La resolución física del panel es 1200 × 2664; la aplicación quedó en horizontal. DroidDeck eligió automáticamente una salida de sesión de **1598 × 720**. Las resoluciones personalizadas para Steam y Desktop permanecieron desactivadas. No se cambió manualmente la resolución.

## Refresh rate

| Superficie / intento | Evidencia | Resultado |
|---|---|---|
| Android Home | `dumpsys display`, modo activo 2 | Panel físico a 120.00001 Hz; render policy puede ser 60 Hz |
| DroidDeck MainActivity | `VariableRefreshRateHandler` | 60.00 Hz |
| SessionActivity, primer intento | `device.txt`, `BL_REFRESH=60` | 60.00 Hz |
| SessionActivity, segundo intento | `device.txt`, `BL_REFRESH=120` | Solicitud/configuración 120.00 Hz |
| Superficie de SessionActivity | `VariableRefreshRateHandler` durante la captura | 60.00 Hz deseados por MagicOS en las líneas capturadas |
| Steam Big Picture | No inició | No comprobado |

La aplicación registró `display 60 → 120 Hz (mode 2)` y el panel cambió al modo físico 120 Hz. En el segundo intento el entorno también llevó `BL_REFRESH=120`. Aun así, no hubo frames de Gamescope y no se pudo demostrar una renderización Big Picture real a 120 fps/Hz. Los aproximadamente 600 ticks por 10 s del compositor corresponden a su bucle cercano a 60 Hz, no a frames presentados.

## FPS Steam UI

No medible: Big Picture no inició. No hay valores válidos para reposo, scroll de biblioteca, Settings ni transiciones. El HUD estaba activado, pero no recibió contenido Steam.

## CPU

No se modificó afinidad; la configuración efectiva dejó todos los cores disponibles.

Los procesos transitorios confirmados fueron `proot`/sesión y Gamescope. En el primer intento se registró Gamescope PID 26825; en el segundo, `proot` PID 28871 y Gamescope PID 28878. El fallo ocurrió en menos de dos segundos, de modo que no fue posible obtener una muestra estable de `top` ni distribución por core para Steam.

Al cierre solo seguía `com.droiddeck.launcher` PID 25821; no quedaban `proot`, Gamescope, Steam, `steamwebhelper` ni FEX. El campo de CPU mostrado por `ps` para DroidDeck en esa lectura fue aproximadamente 5, pero no representa una carga sostenida de sesión.

## RAM

| Momento | MemAvailable | Swap usada |
|---|---:|---:|
| Antes | 4,925,212 kB | 2,457,600 kB |
| Después del fallo | 4,791,540 kB | 2,924,032 kB |

Medición final de DroidDeck:

```text
TOTAL PSS:       73,487 kB
TOTAL RSS:      240,908 kB
TOTAL SWAP PSS:   7,366 kB
```

No existe una medición “durante Big Picture” porque esa fase no se alcanzó. La diferencia global de memoria/swap no se atribuye exclusivamente a DroidDeck: corresponde al estado completo del sistema entre ambas lecturas.

## GPU

`gpubusy` fue legible. Tras el fallo, cinco muestras separadas por aproximadamente 400 ms devolvieron `0 0`, consistente con ausencia de carga gráfica. No se obtuvo una ventana útil de carga porque Gamescope murió casi inmediatamente.

MagicOS negó acceso de lectura a:

```text
/sys/class/kgsl/kgsl-3d0/gpuclk
/sys/class/kgsl/kgsl-3d0/devfreq/cur_freq
/sys/class/kgsl/kgsl-3d0/default_pwrlevel
/sys/class/kgsl/kgsl-3d0/num_pwrlevels
```

Por ello no hay frecuencia ni power level verificables. No se escribió ningún nodo KGSL. `logcat` contiene mensajes del servicio de rendimiento de HONOR intentando cambiar `min_pwrlevel`, pero son acciones del sistema, no de esta prueba, y no demuestran que el cambio tuviera éxito.

## Thermal

Lecturas actuales del HAL, no los valores cacheados:

| Momento | Batería | GPU0 | GPU1 | Skin | Estado | Cooling GPU/devfreq |
|---|---:|---:|---:|---:|---:|---:|
| Inicio | 35.0 °C | 38.4 °C | 39.2 °C | 35.208 °C | 1 | 0 / 0 |
| Cierre, después de ambos fallos | 37.0 °C | 40.4 °C | 40.8 °C | 36.737 °C | 1 | 0 / 0 |

Los valores cacheados llegaron a mostrar GPU0 48.0 °C, GPU1 49.2 °C y skin 37.116 °C, pero no se mezclan con las lecturas “Current temperatures from HAL”. No hubo throttling activo en los cooling devices de GPU o devfreq.

No se pudieron tomar cortes válidos a +2, +5 y +10 min de Big Picture porque Steam no inició y la conexión USB/ADB se interrumpió durante el desempaquetado. Inventar esos puntos produciría una comparación inválida.

## Touch

La interfaz Android de DroidDeck respondió y permitió iniciar el flujo. El touch dentro de Steam/Big Picture no se pudo comprobar. El entorno mantuvo `Touch mode auto`; no se modificó.

## Audio

Estado de infraestructura: **PASS parcial**.

PulseAudio 13.0 inició correctamente, creó `AAudioSink` estéreo 44.1 kHz y `DirectAudioMic` mono 48 kHz. El relay DirectAudio abrió el micrófono, creó el socket y detectó un lector. Esto demuestra que el plumbing de audio arrancó.

Audio audible de Steam: **NO COMPROBADO**, porque Steam nunca produjo contenido. El error de prioridad en tiempo real de PulseAudio (`setrlimit(RLIMIT_RTPRIO)`) no impidió el inicio del daemon.

## Errores

Errores determinantes:

1. Turnip/Mesa Linux `26.2.2` declara no soportado `chip_id 43020000 / gpu_id 6720`.
2. No puede consultar la versión del kernel driver mediante `/dev/dri/renderD0`.
3. Gamescope no encuentra dispositivo Vulkan físico.
4. Gamescope no puede crear el backend y el invitado sale con estado 1.

Errores o limitaciones secundarios:

- `libfakeinput.so` falta para `/etc/ld.so.preload`;
- compositor Android con `present: no physical devices` y nombre de driver vacío;
- no hay layout dma-buf `qcom_compressed` importable; se mantienen buffers lineales;
- no hay dispositivo DRM de display para clientes;
- acceso denegado a frecuencia y power level de KGSL;
- desconexión USB/ADB durante el desempaquetado, sin detener la instalación en el teléfono.

## Logs encontrados

Se recopilaron fuera del control de versiones en:

```text
C:\Users\Grafiplot\AppData\Local\Temp\droiddeck-a720-operational-20260930-175315
```

Sesiones:

```text
Download/DroidDeck/session-20260930-175405
Download/DroidDeck/session-20260930-175900
```

Se conservaron `device.txt`, `app.log`, `session.log`, `wayland.log`, `network.txt`, `events.jsonl`, `audio.log`, `crash.log` y marcadores de finalización cuando existían. La segunda sesión comparte el compositor ya iniciado y por eso la evidencia Wayland continuó en el `wayland.log` de la primera carpeta. También se conservó el `logcat` general hasta la desconexión USB.

## Cuellos de botella confirmados

1. **Compatibilidad Vulkan Linux con Adreno 720:** el Turnip de Mesa 26.2.2 incluido en runtime `r9` rechaza explícitamente el GPU ID 6720. Es el primer bloqueo reproducible y suficiente para impedir Gamescope y Steam.
2. **Creación de backend Gamescope:** sin dispositivo físico Vulkan, Gamescope termina inmediatamente.
3. **Ruta de presentación Android no validada:** aunque `turnip25.1.0` está seleccionado, el compositor informa driver vacío y ningún dispositivo físico; no se demostró una ruta Turnip Android funcional.

## Cuellos de botella probables

- La ausencia de soporte específico para esta representación de Adreno 720 en ambos lados de la ruta gráfica puede requerir una variante de Turnip compatible, pero esta baseline no autoriza ni evalúa ese cambio.
- La falta de `libfakeinput.so` podría afectar controles cuando se supere Vulkan, aunque no explica el fallo gráfico actual.
- La ruta dma-buf limitada a layouts lineales podría afectar rendimiento o compatibilidad cuando existan frames, pero aún no puede cuantificarse.

## Cosas todavía no comprobadas

- Inicio real de Steam ARM64 y `steamwebhelper`.
- Big Picture visible y estable.
- FPS de la interfaz Steam.
- Renderizado real de Steam a 60, 90 o 120 Hz.
- Touch, menú lateral, biblioteca, Settings y scroll dentro de Steam.
- Audio audible de Steam.
- Uso sostenido de CPU por proceso y por core.
- Memoria durante Big Picture.
- Uso y frecuencia GPU bajo carga estable.
- Thermal a +2, +5 y +10 min de una sesión estable.
- FEX en ejecución.
- Comportamiento de juegos.

## Conclusión

La baseline #2 cumple su objetivo diagnóstico: elimina únicamente el bloqueo de procesos y revela el siguiente fallo real de la cadena. El runtime se instala correctamente, pero el Turnip Linux incluido no acepta la Adreno 720 del Honor 200; por ello Gamescope sale antes de Steam y Big Picture.

No se aplicó ninguna optimización ni workaround. La siguiente prueba debe tratar este fallo como una variable A/B aislada, manteniendo iguales el resto de ajustes.
