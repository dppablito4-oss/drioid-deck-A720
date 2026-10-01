# DroidDeck A720 — Prueba A/B Zero-Copy

Fecha: 2026-09-30 (America/Bogota)  
Dispositivo: Honor 200 / SM7550 / Adreno 720  
Aplicación: DroidDeck 0.2.0 (9)  
Resultado: **PARTIAL — A medida; B bloqueada antes de modificar el estado**

## Resumen

Se midió durante más de cinco minutos el estado A, **Traditional Copy**, sobre una sesión Steam real y estable. En su tramo final estable, el compositor presentó **44.15 FPS** de media mientras recibía **98.2 GPU frames/s** y ejecutaba **60.34 ticks/s**. `wayland.log` confirmó `copy > 0` y `zero-copy 0` durante toda la ventana.

No se ejecutó B porque DroidDeck 0.2.0 contiene el soporte nativo `banner_ahb_v1` / `nativeSetZeroCopy(boolean)`, pero esta APK no expone el control vivo necesario para invocarlo. El drawer en ejecución solo mostró Performance HUD, Stretch y Frame generation. La única llamada accesible desde el código de la aplicación activa Zero-Copy conjuntamente con HDR, lo cual cambiaría dos variables y haría inválido el A/B solicitado.

La APK instalada tampoco es depurable (`run-as: package not debuggable`) y no había un mecanismo JDWP/Frida disponible para invocar el método nativo sin modificar la aplicación. No se añadió un hook, no se recompiló la APK y no se utilizó HDR como atajo.

Zero-Copy nunca se activó, por lo que el estado final continúa en **Traditional Copy** y no fue necesario rollback.

## Variables mantenidas

Durante esta tarea no se modificaron:

- Turnip Android ni Linux;
- FEX;
- afinidad CPU;
- resolución;
- Gamescope;
- frame limiter;
- GPU clocks;
- frame generation;
- HDR;
- código fuente ni APK.

Estado observado de la sesión existente:

| Variable | Valor |
|---|---|
| Display driver | `Mesa_Turnip_v26.3.0-20260930-r7-710-720-Test` |
| Runtime driver | `Mesa_Turnip_v26.3.0-20260930-r7-710-720-Test-Linux` |
| Vulkan | 1.4.363 |
| `TU_DEBUG` efectivo | `sysmem` |
| FEX | defaults |
| Client core override | OFF |
| Game cores | default / todos |
| Resolución de sesión | 1598 × 720 automática |
| Refresh de sesión | 60 Hz |
| Gamescope | 3.16.29 |
| Escena | Pantalla de inicio de sesión Steam con spinner QR |

Estos valores ya estaban activos al comenzar esta tarea y no fueron alterados por la prueba Zero-Copy.

## Soporte Zero-Copy encontrado

El compositor sí arrancó la infraestructura:

```text
zero-copy: banner_ahb_v1 version 2 advertised
zero-copy: off at launch
```

El driver Android funcionó con Adreno 720 y anunció dma-buf lineal y comprimido:

```text
compositor renders on Turnip Adreno (TM) 720 with libvulkan_freedreno.so
AR24 linear+qcom_compressed
XR24 linear+qcom_compressed
AB24 linear+qcom_compressed
XB24 linear+qcom_compressed
```

El proyecto contiene:

```text
WaylandCompositor.nativeSetZeroCopy(boolean)
banner_ahb_v1.mode
contadores copy / zero-copy / layer copy
```

Sin embargo, la UI de esta versión no conecta ese método a un switch independiente. La llamada Kotlin disponible está dentro de la activación HDR:

```text
if (HDR está activo) {
    WaylandCompositor.nativeSetZeroCopy(true)
}
```

Usarla habría cambiado simultáneamente HDR y Zero-Copy, contra la condición de una sola variable.

## Prueba A — Traditional Copy

Ventana cronometrada:

```text
Inicio: 21:26:43
Fin:    21:32:08
Duración observada: 5 min 25 s
```

La ventana incluye el calentamiento inicial de la superficie reanudada. Para describir el rendimiento sostenido se informa también el tramo final de 12 bloques consecutivos de 10 s.

### Resultado completo, incluido calentamiento

| Métrica | Resultado |
|---|---:|
| Bloques stats analizados | 31 |
| FPS presentados, media | 34.00 |
| FPS mínimo observado | 0.2 |
| FPS máximo observado | 45.2 |
| GPU frames, media | 72.77/s |
| Compositor ticks | 60.33/s |
| Escenas | 33.81/s |
| `render_scene` promedio | 5.41 ms |
| `render_scene` máximo observado | 14.22 ms |
| `fence wait` promedio | 3.36 ms |
| `fence wait` máximo observado | 13.38 ms |
| `present` promedio | 1.25 ms |
| `present` máximo observado | 10.79 ms |
| Frames por copia | 10,820 |
| Frames Zero-Copy | **0** |

El mínimo de 0.2 FPS corresponde a una pausa breve de la animación durante el calentamiento, no a un crash. Por eso la media completa no representa por sí sola el régimen sostenido.

### Tramo final estable

| Métrica | Resultado |
|---|---:|
| Duración | 120 s |
| FPS presentados, media | **44.15** |
| GPU frames, media | **98.2/s** |
| Compositor ticks | **60.34/s** |
| `render_scene` promedio / máximo | **5.40 / 14.22 ms** |
| `fence wait` promedio / máximo | **4.14 / 13.38 ms** |
| `present` promedio / máximo | **0.73 / 4.59 ms** |
| Zero-Copy | **0 frames** |

Los últimos cuatro bloques fueron 43.2, 45.2, 44.1 y 45.2 FPS. Esto reproduce el comportamiento aproximado de ~40–45 FPS indicado antes de la prueba.

### GPU busy

Se obtuvieron 55 muestras válidas de `gpubusy` cada cinco segundos:

| Métrica | Resultado |
|---|---:|
| Media | 39.21 % |
| Mínimo | 21.73 % |
| Máximo | 66.05 % |

No se escribió ningún nodo KGSL.

### Thermal

Lectura previa cercana al comienzo:

| Sensor | Valor |
|---|---:|
| GPU0 | 38.4 °C |
| GPU1 | 38.8 °C |
| Batería | 35.0 °C |
| Skin | 34.957 °C |
| Thermal status | 0 |

Al finalizar A:

| Sensor | Valor |
|---|---:|
| GPU0 | 46.8 °C |
| GPU1 | 48.4 °C |
| Batería | 37.0 °C |
| Skin | 37.532 °C |
| Thermal status | 1 |
| Cooling GPU | 0 |
| Cooling KGSL devfreq | 0 |

No hubo cooling activo de GPU/devfreq.

### SurfaceFlinger missed frames

Los contadores globales antes y después permanecieron:

```text
Total missed frame count: 216
HWC missed frame count:   192
GPU missed frame count:    44
```

Delta durante A: **0 / 0 / 0**. Son contadores globales de SurfaceFlinger, no exclusivos de DroidDeck.

### Estabilidad visual

- Crash: no.
- Gamescope, Steam y `steamwebhelper`: vivos al finalizar A.
- Corrupción de color visible en la captura: no.
- Artefactos visibles en la captura: no.
- Flickering: no cuantificable mediante una captura estática; no aparece un error equivalente en los logs.
- Pool drops: 0 en los bloques analizados.

## Prueba B — Zero-Copy AHB

Estado: **NO EJECUTADA / BLOQUEADA**.

No existe en la UI instalada un switch independiente que llame `nativeSetZeroCopy(true)`. Tampoco existe una preferencia o archivo de control independiente en el código de esta versión. Las alternativas descartadas fueron:

1. Activar HDR: también cambia formato, color y ruta de presentación; invalida el A/B.
2. Añadir un botón, intent o hook: sería una implementación nueva y una modificación de código.
3. Instrumentar la APK: la compilación instalada no es depurable.
4. Instalar tooling de inyección: no estaba disponible y ampliaría materialmente el alcance de la prueba.

No apareció ninguna línea:

```text
zero-copy switched on
zero-copy > 0
```

porque el modo nunca fue activado.

## Comparación A vs B

| Pregunta | Resultado |
|---|---|
| FPS finales en A | 44.15 FPS sostenidos |
| FPS finales en B | No medido |
| ¿Sube de ~40 hacia 60 FPS? | **No se puede concluir** |
| ¿Baja `render_scene`? | No se puede concluir |
| ¿Baja `fence wait`? | No se puede concluir |
| ¿Disminuyen missed frames? | No se puede concluir; A tuvo delta 0 |
| ¿Aparece `zero-copy > 0`? | No; A registró 0 y B no se activó |
| ¿Artefactos/flickering/crash en B? | No comprobado |

## Estado final y rollback

Zero-Copy continúa **OFF**. No se cambió ninguna preferencia y no fue necesario rollback. La sesión Steam permaneció viva.

Para completar B de forma válida, DroidDeck necesita exponer el control vivo ya implementado —una acción UI o mecanismo de prueba que invoque exclusivamente `WaylandCompositor.nativeSetZeroCopy(true/false)`— sin habilitar HDR ni reiniciar con otra configuración. Esta tarea no autoriza añadirlo, por lo que se detuvo antes de contaminar el experimento.

## Evidencia

Capturas y muestras de esta prueba quedaron fuera del repositorio en:

```text
C:\Users\Grafiplot\AppData\Local\Temp\droiddeck-a720-zero-copy-ab-20260930-212643
```

El log de sesión fuente fue:

```text
/storage/emulated/0/Download/DroidDeck/session-20260930-190111/wayland.log
```
