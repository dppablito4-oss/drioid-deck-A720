# DroidDeck A720 Baseline

Fecha de la prueba: 2026-09-30 (America/Bogota)  
Repositorio: `dppablito4-oss/drioid-deck-A720`  
Rama/estado inicial: `main...origin/main`; `release_apk/` ya estaba sin seguimiento  
Dispositivo ADB: `A7UH025A28000522`  
Resultado general: **PARTIAL**

## Alcance y método

Se compiló e instaló la versión actual sin modificar código, drivers, FEX, sysfs ni ajustes del sistema. No se borraron datos con `pm clear`, no se hicieron commits y no se actualizó desde upstream.

El APK se construyó mediante `gradlew assembleRelease` con Java 17, Android Platform 34, Build Tools 35.0.0, NDK 27.3.13750724 y CMake 3.22.1. El script reproducible `tools/build_local.sh` no pudo usarse porque Docker no está instalado en el PC; la compilación Gradle usó los binarios y assets ya incluidos en el repositorio.

Al pulsar **Play Steam**, DroidDeck detuvo correctamente el inicio en su preflight porque el ajuste global de Android `settings_enable_monitor_phantom_procs` está sin definir (`null`). La aplicación mostró:

> Steam cannot start yet — Child-process limit is unset

La baseline no cambió ese ajuste. Como resultado, no se descargó el runtime y no llegaron a iniciarse el compositor de sesión, Turnip de DroidDeck, proot, Gamescope, Steam ni FEX. Las capas no alcanzadas se marcan como **NO COMPROBADO**, no como fallos de esas capas.

La evidencia pesada se guardó fuera del repositorio en:

`C:\Users\Grafiplot\AppData\Local\Temp\droiddeck-a720-baseline-20260930-174224`

## Hardware detectado

| Campo | Valor observado |
|---|---|
| Fabricante | HONOR |
| Modelo | ELI-NX9 |
| Device | HNELIX |
| Product | ELI-NX9 |
| Hardware | qcom |
| Board platform | crow |
| SoC | SM7550 |
| Fabricante SoC | QTI |
| ABI | arm64-v8a, armeabi-v7a, armeabi |
| RAM física reportada | 11,602,956 kB (11.60 GB decimal; 11.07 GiB) |

## Android / MagicOS

| Campo | Valor observado |
|---|---|
| Android | 16 |
| API | 36 |
| Build | ELI-N39 10.0.0.160(C636E5R2P1) |
| Incremental | 10.0.0.160C636E5R2P1 |
| MagicOS | 10, según el build y la información objetivo |
| Kernel declarado | 5.15 |

## CPU

`/proc/cpuinfo` expone 8 procesadores (`0-7`) y sysfs confirma `present=0-7`, `online=0-7`. Sin embargo, `nproc` devolvió 6, probablemente por el cpuset visible para `adb shell`; no se interpreta como seis CPU físicas.

La clasificación de cluster es una **estimación basada exclusivamente en los techos y capacidades observados**, no una suposición por el nombre comercial del SoC.

| Core | Max freq | Capacidad sysfs | Cluster estimado |
|---|---:|---:|---|
| 0 | 1,804,800 kHz | 388 | lento/eficiencia |
| 1 | 1,804,800 kHz | 388 | lento/eficiencia |
| 2 | 1,804,800 kHz | 388 | lento/eficiencia |
| 3 | 1,804,800 kHz | 388 | lento/eficiencia |
| 4 | 2,400,000 kHz | 929 | rendimiento |
| 5 | 2,400,000 kHz | 929 | rendimiento |
| 6 | 2,400,000 kHz | 929 | rendimiento |
| 7 | 2,630,400 kHz | 1024 | máximo/prime |

Los cores 0-3 reportan CPU part `0xd46`; los cores 4-7, `0xd4d`. No se aplicó afinidad.

## GPU

| Campo | Valor observado |
|---|---|
| GPU detectada | Adreno 720 |
| Modelo KGSL | `Adreno720` |
| Chip ID KGSL | No expuesto: `gpu_chipid` no existe |
| SoC | SM7550 |
| Driver Vulkan del sistema disponible | Sí: `/vendor/lib64/hw/vulkan.adreno.so` |
| Nodo KGSL disponible | Sí: `/dev/kgsl-3d0`, major/minor `474,0` |
| Clase KGSL | Existe, pero MagicOS niega listar el directorio completo |

`cmd gpu vkjson` confirmó:

- deviceName: `Adreno (TM) 720`;
- driverName: `Qualcomm Technologies Inc. Adreno Vulkan Driver`;
- driver build: `853b1fd7f5 / If60f19d074`;
- fecha del driver: `03/13/26`;
- compiler: `E031.41.03.64`;
- conformance: Vulkan 1.3.0.1.

SurfaceFlinger informó además OpenGL ES 3.2, driver Qualcomm `0676.76.3`.

## Display

| Campo | Valor observado |
|---|---|
| Resolución física | 1200 × 2664 |
| Resolución Android | 1200 × 2664; rotación horizontal de app 2664 × 1200 |
| Densidad física | 520 dpi |
| Densidad override | 384 dpi |
| Modos disponibles | 60.000004, 90.0 y 120.00001 Hz |
| Antes de DroidDeck | 120.00001 Hz (`8,333,333 ns`) |
| Launcher DroidDeck/preflight | 60.000004 Hz (`16,666,666 ns`) |
| Steam | NO COMPROBADO |
| Después de cerrar DroidDeck | 120.00001 Hz |

MagicOS registró inicialmente votos de 120 Hz, pero SurfaceFlinger identificó la capa de `MainActivity` como `ExplicitDefault, desiredRefreshRate 60.00`; el modo activo terminó siendo 60 Hz. Esto demuestra 60 Hz en el launcher/preflight, no el comportamiento de `SessionActivity`, que no llegó a abrirse.

## DroidDeck build

| Campo | Valor |
|---|---|
| Package | `com.droiddeck.launcher` |
| versionName | 0.2.0 |
| versionCode | 9 |
| minSdk | 26 |
| targetSdk | 28 |
| APK instalado | `/data/app/.../com.droiddeck.launcher-.../base.apk` |
| APK local | `app/build/outputs/apk/release/app-release.apk` |
| Tamaño | 25,902,644 bytes |
| SHA-256 | `D8F192AC8CDEF1A596B94B15D274ABF56C5743AFFA0A18C4275501D10FCC0F08` |
| Coincidencia instalado/local | Sí, hash idéntico |

La firma v2/v3 se verificó con la testkey AOSP configurada por el proyecto.

## Turnip Android

Configuración visible: **Display driver: Auto - picked by GPU**.

El código actual asignaría un GPU 7xx al asset incluido `turnip25.1.0`. Sin embargo, la sesión no llegó a crear el compositor ni a registrar `TurnipDriver: graphics driver ...`; por tanto:

| Campo | Resultado |
|---|---|
| nombre configurado | Auto → `turnip25.1.0` según la lógica actual |
| versión configurada | 25.1.0 según el identificador del asset |
| ruta prevista | directorio privado `files/graphics_driver/turnip25.1.0/` |
| selección | automática por `gpu_model=Adreno720` |
| cargado/validado en esta ejecución | **NO** |

El launcher Compose/HWUI sí cargó el driver OpenGL ES Qualcomm del sistema (`/vendor/lib64/egl/libGLESv2_adreno.so`, versión 0676.76.3). Esto no debe confundirse con el Turnip bionic del compositor de sesión.

## Turnip Linux

Configuración visible: **Runtime driver: Runtime default**.

El runtime no está instalado y no existe un ICD Linux que pueda inspeccionarse en esta instalación. Por tanto:

| Campo | Resultado |
|---|---|
| nombre | Runtime default |
| versión | NO DISPONIBLE |
| ruta | NO DISPONIBLE |
| selección | default actual, sin import manual |
| cargado/validado | **NO** |

### TU_DEBUG

- No existe evidencia de `Download/droiddeck-tu-debug`.
- La instalación fue limpia y el default de `tuSysmem` en esta versión es `false`.
- El driver Linux seleccionado está vacío (`Runtime default`), por lo que la regla automática para imports con nombre `710-720` tampoco aplica.
- Resultado configurado: **TU_DEBUG no se establecería**.
- Resultado en proceso: no hubo proceso de sesión donde verificarlo.

## FEX

La UI mostró **FEX defaults**. En esta configuración:

| Campo | Resultado |
|---|---|
| FEX preset | FEX defaults |
| FEX variables | ninguna variable adicional configurada |
| CPU affinity | sin override; todos los cores por defecto |
| FEX ejecutado | NO |

No hay procesos FEX ni runtime instalado. La configuración se registra, pero su comportamiento no pudo evaluarse.

## Steam startup

| Capa | Estado | Evidencia |
|---|---|---|
| Launcher | PASS | Arranque en frío correcto, UI visible y estable; sin crash/ANR |
| Preflight de Android | FAIL/BLOCKED | `settings_enable_monitor_phantom_procs=null`; “Steam cannot start yet” |
| Compositor | NO COMPROBADO | `SessionActivity`/compositor no se iniciaron |
| Turnip Android | NO COMPROBADO | No aparece carga de `TurnipDriver` en el log |
| Runtime | FAIL/NO INSTALADO | Setup informó “Not installed · about 3 GB” |
| Turnip Linux | NO COMPROBADO | Runtime ausente |
| Gamescope | NO COMPROBADO | Sin proceso ni log |
| Steam ARM64 | NO COMPROBADO | Sin proceso ni log |
| Big Picture | NO COMPROBADO | No alcanzado |
| Touch | PARTIAL | Navegación táctil del launcher y ajustes funcionó; sesión no alcanzada |
| Audio | NO COMPROBADO | Sesión no alcanzada |

No se observó pantalla negra de sesión: la sesión nunca se creó.

## Memoria

| Momento | MemAvailable | PSS DroidDeck | RSS DroidDeck |
|---|---:|---:|---:|
| Antes de DroidDeck | 4,953,596 kB | — | — |
| Preflight activo | 4,920,640 kB | 127,699 kB | 281,664 kB |
| Tras varios minutos en UI | 4,892,884 kB | 114,975 kB | 270,092 kB |
| Steam/Big Picture | NO COMPROBADO | NO COMPROBADO | NO COMPROBADO |

Swap total: 12,582,908 kB. No se atribuye la variación completa de `MemAvailable` a DroidDeck porque había otras aplicaciones y servicios activos.

## CPU durante Steam

**NO COMPROBADO.** No existieron procesos `steamwebhelper`, FEX, Gamescope o proot.

En el primer snapshot del preflight, DroidDeck apareció con 71.4% de un core mientras la pantalla comprobaba el ajuste cada dos segundos. Tras varios minutos quedó en 0.0% en el snapshot puntual. Esto no representa consumo de Steam ni un benchmark sostenido.

## GPU / KGSL

MagicOS permitió leer `gpubusy`:

- preflight: `669921 1002251` en la lectura puntual;
- formato interpretado como contadores busy/total del kernel, no como porcentaje estable de una carga Steam.

El listado de `/sys/class/kgsl/kgsl-3d0/` fue denegado, por lo que no se inventan `gpuclk`, power level ni métricas térmicas KGSL. `dumpsys thermalservice` sí expuso sensores GPU.

## Thermal

| Momento | Estado | Batería | GPU0/GPU1 actual | Skin actual | Cooling devices |
|---|---:|---:|---:|---:|---|
| Antes | 1 | 36.0 °C | 39.6 / 40.8 °C | 36.098 °C | todos 0 |
| Preflight activo | 1 | 36.0 °C | 45.2 / 46.8 °C | 36.544 °C | todos 0 |
| Tras varios minutos de UI | 1 | 36.0 °C | 39.6 / 40.0 °C | 35.976 °C | todos 0 |

Los valores “Cached temperatures” eran más altos y no se mezclan con las lecturas actuales del HAL. No hubo cooling device activo ni throttling observable. No se obtuvo comportamiento térmico bajo Steam porque Steam no arrancó.

## Refresh rate

| Estado | Hz medidos |
|---|---:|
| Home antes de DroidDeck | 120.00001 |
| Launcher/preflight DroidDeck | 60.000004 |
| Steam | NO COMPROBADO |
| Home después de cerrar DroidDeck | 120.00001 |

El panel soporta realmente 60/90/120 Hz. La afirmación “Steam funciona a 120 Hz” no puede confirmarse ni refutarse con esta ejecución.

## Logs propios de DroidDeck

No se encontraron:

- `Download/DroidDeck/`;
- `Download/Wayland-logs/`;
- `device.txt`;
- `app.log`;
- `session.log`;
- `wayland.log`;
- `network.txt`;
- `events.jsonl`;
- `audio.log`.

La ausencia es coherente con el código: `SessionPaths.beginOrCurrent()` crea el directorio cuando comienza una sesión. El preflight se detuvo antes.

Sí se conservaron fuera del repositorio:

- `logcat-full.txt` (captura completa, ~5.8 MB);
- `steam-blocker.png`;
- `after-play.xml` y dumps XML de configuración;
- `installed-base.apk` para comprobar el hash.

## device.txt

No existe porque no comenzó ninguna sesión. Por ello no se atribuyen a `device.txt` datos obtenidos por otros medios. Los equivalentes ADB observados fueron:

- Model: ELI-NX9;
- Device/product: HNELIX / ELI-NX9;
- Board/hardware: crow / qcom;
- SoC: SM7550;
- Android: 16, API 36;
- Kernel: 5.15 declarado;
- ABIs: arm64-v8a, armeabi-v7a, armeabi;
- Cores: 8 online en sysfs;
- RAM: 11,602,956 kB;
- KGSL model: Adreno720;
- KGSL chip id: no expuesto;
- System Vulkan ICD: presente;
- Session output/refresh: no creados;
- Display/Linux runtime drivers: configurados como Auto y Runtime default, no cargados.

## Errores encontrados

| Severidad | Archivo/log | Mensaje relevante | Posible capa responsable |
|---|---|---|---|
| Crítica para la prueba | UI + `settings get global` | Child-process limit unset; valor `null` | Prerrequisito Android/MagicOS; el preflight de DroidDeck actúa correctamente |
| Alta para cobertura | UI Setup | Runtime “Not installed · about 3 GB” | Estado de instalación, no fallo probado del instalador |
| Media | `logcat-full.txt` | `GamesAware: ... is not a game` y `ITouchService: ... not game app` | Clasificación/lista interna de MagicOS |
| Media | `logcat-full.txt` | `ANDR-PERF-UTIL ... Failed to update ... kgsl-3d0/min_pwrlevel` | Framework de rendimiento Qualcomm/HONOR; no prueba un fallo de DroidDeck |
| Media | `logcat-full.txt` + dumpsys | Capa de MainActivity pide 60 Hz; el sistema vuelve de 120 a 60 | Launcher/SurfaceFlinger/MagicOS; la sesión no fue medida |
| Baja | `logcat-full.txt` | Una ocurrencia `ShaderCache::load value buffer allocate failed` | HWUI/MagicOS; sin defecto visual observado |
| Baja | `logcat-full.txt` | Servicio `IUniPerfAidlInterface` denegado por SELinux/no declarado | Integración vendor de MagicOS; el launcher siguió estable |

No hubo `FATAL EXCEPTION`, ANR ni muerte inesperada del proceso durante la prueba.

## Cuellos de botella observados

### CONFIRMADOS

1. El preflight impide cualquier sesión mientras el monitor de procesos fantasma está sin definir.
2. El runtime Linux no está instalado.
3. El launcher/preflight funciona a 60 Hz aunque Home estaba y vuelve a 120 Hz.
4. No hay datos de sesión propios porque ninguna sesión comenzó.

### PROBABLES

1. La integración de juego/rendimiento de MagicOS es inconsistente: algunos servicios reciben el paquete, otros lo consideran “not a game”, y los intentos vendor sobre `min_pwrlevel` fallan.
2. La política de refresco de MainActivity prevalece a 60 Hz después de votos transitorios de 120 Hz. No se extrapola a SessionActivity.

### NO COMPROBADOS

1. Rendimiento o compatibilidad de Turnip bionic.
2. Turnip glibc del runtime y `TU_DEBUG` efectivo.
3. Wayland, dma-buf, KGSL poll, Gamescope, Steam ARM64 y Big Picture.
4. Audio, touch dentro de Steam, FPS/HUD y frame pacing.
5. CPU/RAM/GPU y throttling bajo carga real.
6. FEX, juegos, Wine y afinidad de CPU.

## Próximos experimentos recomendados

Antes de cualquier optimización debe hacerse una **segunda captura de baseline operativa** con un único cambio de prerrequisito documentado: desactivar “Restrict child processes” (o establecer `settings_enable_monitor_phantom_procs=false`), instalar el runtime mediante el flujo normal y repetir exactamente esta medición. Ese cambio debe quedar registrado porque esta baseline conserva `null`.

Una vez alcanzado Big Picture, realizar pruebas A/B de una sola variable y restaurar el estado entre ellas:

1. Drivers específicos para A720 frente al driver baseline.
2. `TU_DEBUG=sysmem` OFF/ON.
3. KGSL poll fix OFF/ON.
4. SustainedPerformanceMode ON/OFF.
5. Frame pacing y selección real de 60/90/120 Hz.
6. Resolución baseline 720p frente a alternativas controladas.
7. FEX defaults frente a presets de rendimiento.
8. Afinidad por clusters, usando los techos/capacidades medidos y no una topología asumida.

En cada A/B se deben registrar tiempo de arranque, FPS de Big Picture, frame times, PSS/RSS, `MemAvailable`, top de CPU, `gpubusy`, Hz activo y térmicas actuales del HAL.

## Conclusión

La plataforma queda identificada inequívocamente como Honor 200 / SM7550 / Adreno 720. El APK 0.2.0 compilado desde este repositorio está instalado correctamente y el launcher es estable, pero la baseline de Steam queda **PARTIAL**: el preflight bloquea la sesión por el límite de procesos fantasma sin definir y el runtime no está instalado. No existe evidencia válida todavía para juzgar Turnip, Gamescope, Steam, FEX, audio o rendimiento de Big Picture.
