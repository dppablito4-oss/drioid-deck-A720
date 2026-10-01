# DroidDeck A720 — Compatibility Test 01

Fecha de la prueba: 2026-09-30 (America/Bogota)  
Repositorio: `dppablito4-oss/drioid-deck-A720`  
Dispositivo ADB: `A7UH025A28000522` (HONOR ELI-NX9, Snapdragon 7 Gen 3 / SM7550, Adreno 720, Android 16 / MagicOS 10)  
Aplicación: DroidDeck `0.2.0 (9)`, `com.droiddeck.launcher`  
Resultado general: **PARTIAL**

---

## Resumen ejecutivo

El objetivo de esta prueba fue sustituir la **pareja gráfica completa** de DroidDeck por drivers Turnip específicamente construidos para Adreno 710/720/722 (release `Banners-Turnip v26.3.0-20260930-r5`), sin alterar ninguna otra capa (FEX, afinidad CPU, resolución, `TU_DEBUG` manual, Gamescope ni Proton).

Los resultados se dividen claramente entre el lado Android (compositor) y el lado Linux (runtime invitado):

1. **Lado Android (Display Driver) — PASS COMPLETO:**  
   El nuevo Turnip Android (`Turnip-v26.3.0-20260930-r5-710-720-Test.zip`) fue cargado exitosamente por el compositor Wayland de DroidDeck. Identificó explícitamente `Turnip Adreno (TM) 720`, reportó Vulkan `1.4.363`, negoció formatos dma-buf con modificadores de compresión Qualcomm (`qcom_compressed`), habilitó motores de frame generation (`LSFG Native` y `Win-FG Native`) y presentó exitosamente los primeros frames reales (`2 frames on screen`). Se resolvió por completo la incertidumbre del baseline anterior, donde el driver del compositor aparecía vacío `()`.

2. **Lado Linux (Runtime Driver) — FAIL / EVALUACIÓN DEL DRIVER BLOQUEADA:**  
   El driver Linux (`Turnip-v26.3.0-20260930-r5-710-720-Test-Linux.zip`) fue descargado, importado y seleccionado correctamente en la interfaz (`Mesa_Turnip_v26.3.0-20260930-r5-710-720-Test-Linux`, generando su manifest `icd.json`). Asimismo, DroidDeck asignó automáticamente `TU_DEBUG=sysmem` y exportó `BL_VK_DRIVER`.  
   Sin embargo, dentro del entorno proot, Gamescope continuó cargando el Turnip stock del runtime: **Mesa 26.2.2**. El proceso repitió idénticamente el fallo del baseline: `device (chip_id = 43020000, gpu_id = 6720) is unsupported (VK_ERROR_INCOMPATIBLE_DRIVER)` y Gamescope terminó con estado 1 sin alcanzar Steam.

   **Causa raíz diagnosticada:** `SessionFiles.kt` intenta copiar `bannerlator-session` desde los assets del APK hacia `linuxfs/usr/local/bin/bannerlator-session` en cada arranque para inyectar la lógica de redirección `VK_DRIVER_FILES="$BL_VK_DRIVER"`. Debido a que el APK compilado localmente en Baseline #1 no incluyó el directorio `assets/linuxfs/`, `SessionFiles` arrojó `FileNotFoundException: linuxfs/usr/local/bin/bannerlator-session`. Como consecuencia, el runtime conservó el script `bannerlator-session` stock original de `r9`, el cual no exporta `VK_DRIVER_FILES`, forzando a Gamescope a consultar únicamente el driver residual Mesa 26.2.2.

Siguiendo estrictamente las reglas del test, **la prueba se detuvo en el Primer Checkpoint** sin aplicar modificaciones manuales a FEX, CPU, Gamescope ni Proton.

---

## Tabla principal: Baseline #2 vs A720 Compatibility Test 01

| Capa / Criterio | Baseline #2 | A720 Compatibility Test 01 |
|---|---|---|
| **Turnip Android** | No demostrado (`the compositor's driver ()`, `present: no physical devices`, solo formatos lineales) | **PASS**: `Mesa Turnip v26.3.0-20260930-r5-710-720-Test` cargado con éxito. Identifica `Turnip Adreno (TM) 720`, Vulkan 1.4.363, soporte completo `qcom_compressed`. |
| **Turnip Linux** | FAIL `gpu_id 6720 unsupported` (Mesa 26.2.2) | **FAIL**: Persiste `chip_id = 43020000, gpu_id = 6720 is unsupported` en Mesa 26.2.2. El nuevo driver Mesa 26.3.0 no llegó a ejecutarse en el invitado. |
| **Vulkan physical device** | FAIL | **PARTIAL**: PASS en compositor Android (`Turnip Adreno (TM) 720`), FAIL en Gamescope/Linux (`failed to find physical device`). |
| **Gamescope** | FAIL: 3.16.29 no crea backend Vulkan y sale con código 1 | **FAIL**: 3.16.29 inicia, no encuentra dispositivo físico Vulkan en el invitado y sale con estado 1 (`SDL_Vulkan_CreateSurface failed`). |
| **Frames presentados** | 0 presentados (`0 on screen`) | **2 frames presentados** (`2 frames on screen (0.2 fps)`, `present 3.20/6.13 ms`). |
| **Steam** | No iniciado (`guest.exited` código 1) | **No iniciado** (Gamescope aborta antes de ejecutar el cliente Steam). |
| **Big Picture** | No | **No** (no alcanzado). |

---

## Drivers exactos y comprobación SHA-256

Se utilizó exclusivamente la release oficial:
```text
Banners-Turnip v26.3.0-20260930-r5
```

### 1. Display Driver (Android / Bionic)
- **Archivo descargado:** `Turnip-v26.3.0-20260930-r5-710-720-Test.zip`
- **Tamaño:** 2,634,741 bytes
- **SHA-256 esperado:** `1058d306e61e245afe979122bc0e616cd6efa69ddeb7c49fb673b728331fc851`
- **SHA-256 verificado:** `1058d306e61e245afe979122bc0e616cd6efa69ddeb7c49fb673b728331fc851` (**MATCH EXACTO**)
- **Método de descarga:** Gestor integrado de DroidDeck (`Refresh drivers` → `Download`).
- **ID interno de instalación:** `Mesa_Turnip_v26.3.0-20260930-r5-710-720-Test`
- **Ruta en dispositivo:** `/data/user/0/com.droiddeck.launcher/files/graphics_driver/Mesa_Turnip_v26.3.0-20260930-r5-710-720-Test/`
- **Biblioteca principal:** `libvulkan_freedreno.so` (AArch64, bionic)

### 2. Runtime Driver (Linux / GLIBC)
- **Archivo descargado:** `Turnip-v26.3.0-20260930-r5-710-720-Test-Linux.zip`
- **Tamaño:** 3,210,767 bytes
- **SHA-256 esperado:** `a8c6ffa9881a5fba845707a268af5e0352653ca9c54217804ce18f4e19b3d144`
- **SHA-256 verificado:** `a8c6ffa9881a5fba845707a268af5e0352653ca9c54217804ce18f4e19b3d144` (**MATCH EXACTO**)
- **Método de descarga:** Gestor integrado de DroidDeck (`Refresh drivers` → `Download`).
- **ID interno de instalación:** `Mesa_Turnip_v26.3.0-20260930-r5-710-720-Test-Linux`
- **Ruta en dispositivo:** `/data/user/0/com.droiddeck.launcher/files/linux_vulkan_drivers/Mesa_Turnip_v26.3.0-20260930-r5-710-720-Test-Linux/`
- **Manifest ICD:** `icd.json`
- **Biblioteca principal:** `libvulkan_freedreno.so` (AArch64, glibc 2.38+)

---

## Estado original conservado y rollback

Antes y durante la prueba se mantuvieron sin alterar:

```text
Display driver original:     Auto -> turnip25.1.0 (conservado en base de datos para rollback)
Runtime driver original:     Runtime default (conservado en base de datos para rollback)
FEX defaults:                Sin cambios
CPU affinity:                Sin cambios (default, todos los núcleos)
Steam client core override:  OFF
Game cores:                  Default
SustainedPerformanceMode:    Sin cambios
GPU clock pin:               OFF
Frame generation:            OFF
Frame limiter:               Sin cambios
Resolución:                  Automática (1598x720)
Zink defaults:               Lazy descriptors true, glthread true, no-gl-error true
Audio:                       AAudioSink 44.1 kHz, DirectAudio 48 kHz
Gamescope:                   3.16.29
Proton:                      Ready (r9)
Runtime:                     r9
```

---

## TU_DEBUG y Sysmem

- **Valor observado en el proceso invitado (`proot`):**
  ```bash
  TU_DEBUG=sysmem
  ```
- **Origen:** **Automático por driver.**
  La lógica de DroidDeck 0.2.0 (`SessionService.kt: tuDebug(linuxDriverId)`) evaluó el ID `Mesa_Turnip_v26.3.0-20260930-r5-710-720-Test-Linux`, detectó el patrón `710-720` y configuró automáticamente `sysmem` en el entorno proot.
- **Preferencia manual:** Se mantuvo intacta (`Turnip sysmem: false` en `device.txt`). La automatización funcionó según lo previsto por el release sin requerir forzado manual.

---

## Checkpoint 1: Turnip Linux (FAIL)

En `session.log` de la sesión invitada:

```text
TU: error: ../mesa-26.2.2/src/freedreno/vulkan/tu_device.cc:1683: device (chip_id = 43020000, gpu_id = 6720) is unsupported (VK_ERROR_INCOMPATIBLE_DRIVER)
TU: error: ../mesa-26.2.2/src/freedreno/vulkan/tu_knl.cc:402: failed to query kernel driver version for device /dev/dri/renderD0 (VK_ERROR_INCOMPATIBLE_DRIVER)
[gamescope] [Error] vulkan: failed to find physical device
```

- **gpu_id detectado:** `6720` (`chip_id = 43020000`)
- **Driver activo reportado por Mesa:** Mesa `26.2.2` (driver built-in del runtime stock)
- **Vulkan version:** N/A (no se inicializó la instancia Vulkan compatible)
- **Dispositivo físico enumerado:** `0`
- **TU_DEBUG activo:** `sysmem`

**Diagnóstico:**  
El mensaje de bloqueo **NO desapareció**. La prueba se detuvo inmediatamente aquí, tal como exigía la regla de corte del Checkpoint 1.

---

## Checkpoint 2: Gamescope (FAIL)

- **Versión:** Gamescope `3.16.29` (compilado con gcc 16.1.1)
- **Backend:** Wayland (`--backend wayland --expose-wayland`)
- **PID:** `1675` (sesión 18:11:08) / `2313` (sesión 18:11:50)
- **Resolución:** `1598 × 720`
- **Refresh rate configurado:** `60 Hz`
- **Fallo registrado:**
  ```text
  [gamescope] [Error] vulkan: failed to find physical device
  ERROR: wayland: Display scaling requires the missing 'zxdg_output_manager_v1' protocol: disabling
  Failed to load plugin 'libdecor-gtk.so': failed to init
  SDL_Vulkan_CreateSurface failed: VK_KHR_wayland_surface extension is not enabled in the Vulkan instance.
  Failed to create backend.
  ```
- **Estado de salida:** `1` (salida limpia del invitado `guest.exited` con código `GUEST_EXIT`).

---

## Checkpoint 3: Display Turnip Android (PASS)

El compositor Android de DroidDeck (`BannerWayland`) registró en `wayland.log` y `logcat`:

```text
18:11:09.070  TurnipDriver: imported driver chosen by the user: Mesa_Turnip_v26.3.0-20260930-r5-710-720-Test
18:11:09.070  TurnipDriver: graphics driver Mesa_Turnip_v26.3.0-20260930-r5-710-720-Test (libvulkan_freedreno.so)
18:11:09.114  BannerWayland: [gpu] compositor renders on Turnip Adreno (TM) 720 with libvulkan_freedreno.so
18:11:09.115  BannerWayland: driver folder /data/user/0/com.droiddeck.launcher/files/graphics_driver/Mesa_Turnip_v26.3.0-20260930-r5-710-720-Test/
18:11:09.124  BannerWayland: [perf] layer buffers: the display's release fences are waited for on the GPU (VK_KHR_external_semaphore_fd)
18:11:09.127  BannerWayland: [framegen] engines ready on the compositor's device: LSFG Native available (supported (device Vulkan 1.4.363, fmt 37)), Win-FG Native available
18:11:10.345  BannerWayland: [dmabuf] formats: AR24 linear+qcom_compressed, XR24 linear+qcom_compressed, AB24 linear+qcom_compressed, XB24 linear+qcom_compressed
18:11:10.346  BannerWayland: [dmabuf] feedback ready: 8 format/modifier pairs, main device 0:0
```

- **Nombre del driver:** `Turnip Adreno (TM) 720` (anteriormente `()`)
- **Versión de Vulkan:** `1.4.363`
- **Biblioteca:** `libvulkan_freedreno.so`
- **Ruta:** `/data/user/0/com.droiddeck.launcher/files/graphics_driver/Mesa_Turnip_v26.3.0-20260930-r5-710-720-Test/`
- **Dispositivo físico Vulkan:** **SÍ**
- **Formatos dma-buf negociados:** AR24, XR24, AB24, XB24 con modificador `qcom_compressed` y linear.
- **Sincronización GPU:** Semáforos `VK_KHR_external_semaphore_fd` operativos.

---

## Checkpoint 4: Primer Frame (PASS)

`wayland.log` confirma presentación activa en pantalla:

```text
18:11:19.131  stats  last 10 s: 2 frames on screen (0.2 fps) | 0 GPU frames from games | 0 window redraws | 0 windows open
18:11:19.131  perf   last 10 s: 599 ticks, 2 scenes, 2 on screen (copy 2, zero-copy 0, layer copy 0) | render_scene 17.56/27.36 ms | base 0 black kept, 0 presented | acquire 0.09/0.14 ms | present 3.20/6.13 ms (2) | fence wait 7.94/15.63 ms (2, 0 GPU release waits)
```

- **Buffers recibidos:** Sí (2 escenas compuestas)
- **Frames presentados:** **2 frames**
- **Tiempo de presentación:** 3.20 ms / 6.13 ms
- **Dispositivo físico compositor:** `Turnip Adreno (TM) 720`

---

## Checkpoints 5 y 6: Steam y Big Picture (NO ALCANZADOS)

- **Steam ARM64:** No llegó a ejecutarse. El evento `steam.starting` en `events.jsonl` fue inmediatamente sucedido por `guest.exited` (PID Gamescope muerto en ~1.1 segundos). No existió proceso `steam` ni `steamwebhelper`.
- **Big Picture visible:** No.
- **Pantalla negra:** No aplica (la sesión aborta y regresa a la UI Android).
- **Artefactos / Flickering:** No observables.
- **Touch / Audio de Steam:** No alcanzados.

---

## Telemetría de hardware

### 1. CPU
- **Topología:** 8 cores (0–3: 1.80 GHz Cortex-A520; 4–6: 2.40 GHz Cortex-A715; 7: 2.63 GHz Cortex-A715).
- **Procesos activos durante el intento:**
  - `com.droiddeck.launcher` (PID 887)
  - `libpulseaudio.so` (PID 1628)
  - `libdirectaudiorelay.so` (PID 1656)
  - `proot` (PID 1663)
  - `gamescope` (PID 1675)
- **Procesos al cierre:** Únicamente `com.droiddeck.launcher` vivo. No quedaron procesos zombis en background.

### 2. Memoria (RAM)
- **MemTotal:** 11,602,956 kB (11.07 GiB)
- **MemAvailable:** 4,670,972 kB (~4.45 GiB)
- **SwapTotal:** 12,582,908 kB (~12.00 GiB)
- **Consumo DroidDeck (PID 887 tras la sesión):**
  - **TOTAL PSS:** 146,447 kB
  - **TOTAL RSS:** 228,556 kB
  - **TOTAL SWAP PSS:** 97,434 kB

### 3. GPU
- **Modelo:** `Adreno720`
- **Muestreo `gpubusy` tras la sesión:**
  ```text
  102040 1006196 (~10.1% de ocupación en reposo/compositor)
  ```
- **GPU Clock / Devfreq:** Nodos de frecuencia denegados por SELinux de MagicOS 10. No se modificaron ni forzaron frecuencias.

### 4. Thermal (HAL Actual)
Lecturas directas de `dumpsys thermalservice`:
- **Batería:** 35.0 °C
- **GPU0:** 38.0 °C
- **GPU1:** 38.8 °C
- **Skin:** 35.42 °C
- **Thermal Status:** 0 (`NONE`, sin estrangulamiento térmico)
- **Cooling devices GPU/cpufreq:** 0 (inactivos)

### 5. Refresh Rate
- **Panel físico:** 120.00 Hz
- **SessionActivity:** 60.00 Hz deseados por la política de renderizado de MagicOS.
- **Gamescope solicitado:** 60 Hz (`BL_REFRESH=60`).

---

## Análisis de fallos

### Errores que desaparecieron respecto a Baseline #2
1. **Compositor sin dispositivo físico:** Resuelto. Ahora detecta `Turnip Adreno (TM) 720`.
2. **Nombre de driver vacío en compositor:** Resuelto (`Turnip Adreno (TM) 720`).
3. **Falta de compresión dma-buf:** Resuelto. Ahora negocia `AR24 linear+qcom_compressed` y 8 modificadores Qualcomm.
4. **Cero frames presentados:** Resuelto. Se presentaron 2 frames reales en pantalla.

### Errores que persistieron
1. **Rechazo de Vulkan en el invitado Linux:** Mesa 26.2.2 rechaza `chip_id = 43020000, gpu_id = 6720` con `VK_ERROR_INCOMPATIBLE_DRIVER`.
2. **Gamescope Backend Failure:** Gamescope aborta al no encontrar dispositivo físico Vulkan en el invitado.
3. **Falta de biblioteca en preload:** `/usr/local/lib/libfakeinput.so` continúa ausente en `/etc/ld.so.preload`.

### Error nuevo crítico identificado
- **Fallo en la puesta en escena de scripts (`SessionFiles`):**  
  ```text
  SessionFiles: could not stage usr/local/bin/bannerlator-session
  SessionFiles: java.io.FileNotFoundException: linuxfs/usr/local/bin/bannerlator-session
  SessionFiles: usr/local/bin/bannerlator-session NOT staged
  ```
  La ausencia de `assets/linuxfs` en el APK compilado impidió que el runtime adoptara el script que lee `BL_VK_DRIVER` y exporta `VK_DRIVER_FILES`.

---

## Información de registro y evidencias (Logs)

Todos los artefactos de la prueba fueron resguardados en el PC sin sobreescribir las baselines previas:

```text
C:\Users\Grafiplot\AppData\Local\Temp\droiddeck-a720-compat-test01-20260930-181108\
├── device.txt       (Configuración y hardware detectado en la sesión)
├── app.log          (Logs de DroidDeck y servicios de sesión)
├── session.log      (Logs de Gamescope, Mesa y bannerlator-session)
├── wayland.log      (Logs del compositor Wayland y presentación de frames)
├── events.jsonl     (Eventos de ciclo de vida de la sesión)
├── network.txt      (Configuración de red y DNS del runtime)
├── audio.log        (Salida de PulseAudio y AAudioSink)
├── crash.log        (Tombstones/dumps del sistema)
└── logcat.txt       (Logcat completo del sistema durante el experimento)
```

---

## Conclusión

El experimento se clasifica como **PARTIAL**.

Respondiendo a la pregunta central de la prueba:
> *¿La pareja Turnip específica A710/A720/A722 permite que el Honor 200 supere el bloqueo Vulkan y alcance Gamescope/Steam?*

1. **En la capa de presentación Android:** **SÍ.** Se confirma categóricamente que el driver específico `Banners-Turnip v26.3.0-20260930-r5-710-720-Test` es plenamente compatible con la Adreno 720 del Snapdragon 7 Gen 3 en Android 16 / MagicOS 10. Desbloquea Vulkan 1.4.363, soporte dma-buf comprimido (`qcom_compressed`) y presentación gráfica real hacia el display.
2. **En la capa de ejecución Linux:** **NO ALCANZADO.** No se pudo verificar la compatibilidad del binario Linux A720 porque la ejecución continuó atada a Mesa 26.2.2 debido a que el script de sesión no pudo redirigir el cargador Vulkan hacia el manifest importado.

Tal como especificaban las reglas del experimento, no se realizaron optimizaciones posteriores ni modificaciones manuales, finalizando la prueba con este diagnóstico.
