# G-code de configuración y validación (Marlin)

Este documento recopila los comandos G-code relevantes para ajustes, restauración y validación de firmware en impresoras 3D con Marlin. Está diseñado para uso privado, con enfoque en trazabilidad, mantenimiento y control técnico. Todos los comandos pueden enviarse por USB desde terminales como Pronterface, OctoPrint, Repetier o desde el menú LCD si está habilitado.

---

## 🧠 Configuración de EEPROM

La EEPROM (Electrically Erasable Programmable Read-Only Memory) es una sección de memoria no volátil que permite guardar configuraciones personalizadas en la impresora, incluso después de apagarla o reiniciarla. En Marlin, esta funcionalidad es esencial para evitar recompilar el firmware cada vez que se ajusta un parámetro como PID, offsets, malla de nivelación, etc.

---

### 🔧 Macros que controlan el comportamiento de EEPROM

```c++
#define EEPROM_SETTINGS        // Habilita el uso de EEPROM con M500/M501
//#define DISABLE_M503         // Desactiva el comando M503 para ahorrar ~2.7 KB de flash
//#define EEPROM_CHITCHAT      // Muestra mensajes detallados al usar EEPROM
#define EEPROM_BOOT_SILENT     // Suprime mensajes de M503 al arrancar, solo muestra errores
#define EEPROM_AUTO_INIT       // Inicializa EEPROM automáticamente si hay errores
#define EEPROM_INIT_NOW        // Inicializa EEPROM en el primer arranque tras recompilar
```

| Macro | Descripción | Recomendación |
|-------|-------------|----------------|
| `EEPROM_SETTINGS` | Activa el uso persistente de configuraciones | ✅ Siempre habilitada |
| `DISABLE_M503` | Elimina el comando `M503` para ahorrar espacio | ❌ Mantener desactivada para trazabilidad |
| `EEPROM_CHITCHAT` | Muestra confirmaciones al usar comandos EEPROM | ⚖️ Útil en pruebas, opcional en producción |
| `EEPROM_BOOT_SILENT` | Suprime salida de `M503` al arrancar | ✅ Recomendado para arranque limpio |
| `EEPROM_AUTO_INIT` | Repara EEPROM automáticamente si hay errores | ✅ Evita fallos por corrupción o cambios de versión |
| `EEPROM_INIT_NOW` | Inicializa EEPROM en el primer arranque tras recompilar | ⚖️ Útil si haces builds limpios y quieres evitar `M502 + M500` manuales |

---

### 🧪 Comandos G-code para gestión de EEPROM

| Comando | Descripción |
|--------|-------------|
| `M500` | Guarda los ajustes actuales en EEPROM |
| `M501` | Carga los ajustes guardados desde EEPROM |
| `M502` | Restaura los valores por defecto del firmware (requiere `M500` para persistirlos) |
| `M503` | Muestra los ajustes actuales guardados en EEPROM |

---

### 🧩 Buenas prácticas para EEPROM

- Ejecutar `M503` tras cada arranque para verificar configuración activa
- Usar `M500` tras cada ajuste validado para persistencia
- Documentar qué ajustes se guardan por versión binaria
- Validar si `EEPROM_AUTO_INIT` se activa tras cambios de configuración

---

## 🔥 Configuración y validación térmica de hotend y cama caliente

Esta sección documenta todo lo relacionado con el sistema térmico de la impresora 3D: sensores, calentadores, protección ante fallos, control PID, y comandos de validación. Su correcta configuración garantiza seguridad, estabilidad de temperatura y calidad de impresión.

---

### 🔧 Macros de protección térmica y control PID

```c++
#define THERMAL_PROTECTION_HOTENDS     // Protege el hotend ante fallos de calentamiento o lectura
#define THERMAL_PROTECTION_BED         // Protege la cama caliente ante fallos térmicos
#define WATCH_TEMP_PERIOD 40           // Tiempo de observación (segundos)
#define WATCH_TEMP_INCREASE 2          // Incremento mínimo esperado (°C)

#define PIDTEMP                        // Activa control PID para el hotend
#define PIDTEMPBED                     // Activa control PID para la cama caliente
#define PID_FUNCTIONAL_RANGE 10        // Rango de error permitido antes de desactivar PID
```

| Macro | Descripción | Recomendación |
|-------|-------------|----------------|
| `THERMAL_PROTECTION_HOTENDS` | Detecta fallos de calentamiento o lectura en el hotend | ✅ Siempre habilitada |
| `THERMAL_PROTECTION_BED` | Detecta fallos térmicos en la cama | ✅ Siempre habilitada |
| `PIDTEMP` | Activa control PID para el hotend | ✅ Recomendado para estabilidad |
| `PIDTEMPBED` | Activa control PID para la cama | ⚖️ Útil si la cama tiene variaciones térmicas |
| `WATCH_TEMP_*` | Define criterios de vigilancia térmica | ✅ Ajustar según hardware |

---

### 🔧 Macros de sensores, límites y extrusión segura

```c++
#define TEMP_SENSOR_0 1        // Sensor del hotend (ej. 1 = EPCOS 100K)
#define TEMP_SENSOR_BED 1      // Sensor de la cama caliente
#define HEATER_0_MAXTEMP 275   // Límite de temperatura para el hotend
#define BED_MAXTEMP 120        // Límite de temperatura para la cama
#define PREVENT_COLD_EXTRUSION // Impide extrusión si el hotend está frío
#define EXTRUDE_MINTEMP 170    // Temperatura mínima para permitir extrusión
```

| Macro | Descripción | Recomendación |
|-------|-------------|----------------|
| `TEMP_SENSOR_*` | Define el tipo de termistor usado | ✅ Ajustar según hardware |
| `*_MAXTEMP` | Límite de seguridad para cada componente | ✅ Validar según tolerancia del material |
| `PREVENT_COLD_EXTRUSION` | Bloquea extrusión si el hotend está frío | ✅ Activar para evitar obstrucciones |
| `EXTRUDE_MINTEMP` | Temperatura mínima para permitir extrusión | ✅ Ajustar según material (PLA, PETG, etc.) |

---

### 🧪 Comandos G-code para validación térmica

| Comando | Descripción |
|--------|-------------|
| `M105` | Consulta temperatura actual del hotend y cama |
| `M104 S0` | Apaga el hotend |
| `M140 S0` | Apaga la cama caliente |
| `M109 S200` | Espera a que el hotend alcance 200 °C |
| `M190 S60` | Espera a que la cama alcance 60 °C |

> Los comandos `M109` y `M190` son bloqueantes: no permiten continuar hasta que se alcance la temperatura deseada. Son útiles para validar sensores y tiempos de calentamiento.

---

### 🧪 Comandos G-code para autotune y ajuste PID

| Comando | Descripción |
|--------|-------------|
| `M303 E0 S200 C8` | Autotune PID para el hotend a 200 °C con 8 ciclos |
| `M303 E-1 S60 C8` | Autotune PID para la cama a 60 °C con 8 ciclos |
| `M301 P I D` | Establece valores PID para el hotend |
| `M304 P I D` | Establece valores PID para la cama caliente |

> Tras ejecutar `M303`, se recomienda usar `M500` para guardar los nuevos valores en EEPROM.

---

### 🧩 Buenas prácticas de validación térmica

- Ejecutar `M105` antes y después de cada prueba para verificar lectura de sensores
- Usar `M109` y `M190` para confirmar que se alcanzan las temperaturas objetivo
- Ejecutar `M303` tras cambios de hardware o firmware
- Verificar estabilidad de temperatura durante impresión prolongada
- Validar que los límites definidos en `*_MAXTEMP` no se excedan durante pruebas
- Confirmar que `PREVENT_COLD_EXTRUSION` bloquea correctamente la extrusión si el hotend está frío
- Usar `M503` para revisar valores PID activos
- Documentar valores PID por versión binaria y tipo de hotend/cama

---

## 🧭 Configuración del proceso de homing y velocidades de posicionamiento

El homing es el proceso mediante el cual la impresora mueve cada eje hasta activar sus respectivos endstops, estableciendo así la posición cero (origen) de coordenadas. Es fundamental para garantizar que los movimientos posteriores se realicen dentro de los límites físicos de la máquina.

---

### 🔧 Macros relacionadas con homing y velocidades

```c++
#define HOMING_FEEDRATE_MM_M { (50*60), (50*60), (4*60) }  // Velocidades en mm/min para X, Y, Z
#define EDITABLE_HOMING_FEEDRATE                           // Permite editar las velocidades vía LCD o M210
//#define Z_SAFE_HOMING                                    // Requiere probe; evita homing fuera de la cama
```

| Macro | Descripción | Recomendación |
|-------|-------------|----------------|
| `HOMING_FEEDRATE_MM_M` | Define la velocidad de homing para cada eje | ✅ Ajustar según mecánica y seguridad |
| `EDITABLE_HOMING_FEEDRATE` | Permite modificar velocidades desde LCD o G-code | ⚖️ Útil en calibración, desactivar en producción |
| `Z_SAFE_HOMING` | Mueve el cabezal a una posición segura antes de hacer homing en Z | ❌ Desactivar si no usas Z-Probe |

---

### 🧪 Comandos G-code para homing y ajuste de velocidad

| Comando | Descripción |
|--------|-------------|
| `G28`   | Realiza homing en todos los ejes (X, Y, Z) |
| `M210`  | Edita velocidades de homing si `EDITABLE_HOMING_FEEDRATE` está habilitado |

> Si los drivers entran en reposo, puede ser necesario ejecutar `G28` nuevamente antes de otros movimientos.

---

### 🧩 Buenas prácticas de validación de homing

- Verificar que cada eje se detenga correctamente en su endstop
- Ajustar `HOMING_FEEDRATE_MM_M` para evitar golpes o movimientos bruscos
- Desactivar `Z_SAFE_HOMING` si no se usa sensor Z-Probe
- Documentar velocidades por binario y tipo de mecánica (cartesiana, CoreXY, etc.)

---

## 📐 Configuración de nivelación de cama manual con tramming

En tu entorno, la nivelación de cama se realiza manualmente ajustando los tornillos en puntos específicos, sin usar sensores Z-Probe ni malla automática. Marlin permite guiar este proceso mediante tramming, que posiciona el cabezal en zonas clave de la cama para facilitar el ajuste físico.

---

### 🔧 Macros relacionadas con nivelación manual

```c++
#define BED_TRAMMING                   // Activa el menú de nivelación por puntos
//#define BED_TRAMMING_USE_PROBE       // Desactivado: no se usa sensor Z-Probe
#define LEVEL_BED_CENTER               // Opcional: incluye punto central en el tramming
```

| Macro | Descripción | Recomendación |
|-------|-------------|----------------|
| `BED_TRAMMING` | Habilita nivelación manual por puntos desde el LCD | ✅ Activar |
| `BED_TRAMMING_USE_PROBE` | Usa sensor para medir cada punto | ❌ Desactivar |
| `LEVEL_BED_CENTER` | Agrega punto central al flujo de tramming | ⚖️ Útil si quieres validar centro de la cama |

### 🧪 Archivos G-code de validación por material

Para validar la calidad de nivelación tras el ajuste manual, se generaron dos archivos G-code que imprimen una cuadrícula de líneas horizontales y verticales en toda la cama. Esto permite verificar adherencia, altura uniforme y consistencia en cada zona.

```gcode
; nivelacion-pla.gcode
M104 S200      ; Calentar hotend a 200 °C
M140 S60       ; Calentar cama a 60 °C
M190 S60       ; Esperar cama
M109 S200      ; Esperar hotend
G28            ; Homing completo
G1 Z0.2 F300   ; Altura inicial
G1 X0 Y0 F9000 ; Inicio en esquina
; Cuadrícula de líneas horizontales y verticales
; (generada desde PrusaSlicer sin retracciones ni desplazamientos Z)
```

```gcode
; nivelacion-petg.gcode
M104 S240      ; Calentar hotend a 240 °C
M140 S80       ; Calentar cama a 80 °C
M190 S80       ; Esperar cama
M109 S240      ; Esperar hotend
G28            ; Homing completo
G1 Z0.2 F300   ; Altura inicial
G1 X0 Y0 F9000 ; Inicio en esquina
; Cuadrícula de líneas horizontales y verticales
; (generada desde PrusaSlicer sin retracciones ni desplazamientos Z)
```

> Ambos archivos fueron generados con altura de capa de 0.2 mm, sin retracciones, y sin desplazamientos Z entre líneas. Se recomienda ejecutarlos tras cada ajuste de tornillos para validar adherencia y uniformidad.

### 🧪 Comandos G-code útiles en nivelación manual

| Comando | Descripción |
|--------|-------------|
| `G28`   | Realiza homing completo (X, Y, Z) antes de iniciar tramming |
| `M503`  | Verifica si los límites y offsets están correctamente definidos |
| `M500`  | Guarda ajustes si se modifican offsets o límites desde el LCD |

> Aunque no usas `G29`, el comando `G28` sigue siendo esencial para posicionar el cabezal antes de ajustar tornillos. El flujo de tramming se realiza desde el menú LCD.

---

### 🧩 Buenas prácticas de validación en tramming manual

- Ejecutar `G28` antes de iniciar el proceso de tramming
- Validar que el cabezal se posicione correctamente en cada punto de ajuste
- Documentar desviaciones por tornillo y registrar correcciones aplicadas
- Usar `LEVEL_BED_CENTER` si quieres validar el centro como referencia adicional

---

---

## 🧵 Configuración del estacionamiento de la boquilla

```c++
#define NOZZLE_PARK_FEATURE
#define NOZZLE_PARK_POINT { (X_MIN_POS + 10), (Y_MAX_POS - 10), 20 }
#define NOZZLE_PARK_MOVE 0
#define NOZZLE_PARK_Z_RAISE_MIN 2
#define NOZZLE_PARK_XY_FEEDRATE 100
#define NOZZLE_PARK_Z_FEEDRATE 5
```

| Comando | Descripción |
|--------|-------------|
| `G27 P0` | Estaciona la boquilla si Z está por debajo del punto |
| `G27 P1` | Siempre eleva Z hasta la altura de parqueo |
| `G27 P2` | Eleva Z por la cantidad definida sin exceder `Z_MAX_POS` |
| `M600`   | Inicia cambio de filamento (si está habilitado) |
| `M25` / `M24` | Pausa y retoma impresión |

### Flujo sugerido para cambio de filamento

```gcode
M25         ; Pausar impresión
G27 P1      ; Estacionar boquilla
M600        ; Cambiar filamento
G1 E10 F300 ; Extruir prueba
M24         ; Retomar impresión
```

---

## 🔌 Implementación del apagado automático de fuente

### Requisitos físicos

- Fuente ATX o similar con control por señal lógica (pin verde PS_ON)
- Alternativamente, módulo de relé o MOSFET para cortar alimentación principal
- Conexión segura entre pin libre de la mainboard y circuito de control (ej. PA1 → transistor → PS_ON)

> ⚠️ Nunca conectar directamente a 220 V. Usar aislamiento mediante relé, optoacoplador o transistor.

### Configuración en Marlin

```c++
#define PS_ON_PIN PA1
#define POWER_SUPPLY 1
#define PSU_CONTROL
#define PSU_DEFAULT_OFF
#define PSU_ACTIVE_HIGH
```

### Comandos G-code para control de fuente

| Comando | Descripción |
|--------|-------------|
| `M80` | Enciende la fuente |
| `M81` | Apaga la fuente |

### Ejemplo de cierre automático al finalizar impresión

```gcode
M104 S0      ; Apagar hotend
M140 S0      ; Apagar cama
M84          ; Apagar motores
M81          ; Apagar fuente
```

---

## 🧰 Comandos adicionales de diagnóstico

| Comando | Descripción |
|--------|-------------|
| `M115` | Información del firmware |
| `M420 V` | Estado de malla de nivelación |
| `M119` | Estado de endstops |

---

## 🧩 Buenas prácticas de validación

- Ejecutar `M503` tras cada arranque para verificar configuración activa
- Usar `M500` tras cada ajuste validado para persistencia
- Documentar cada sesión de validación con fecha, binario y comandos usados
- Validar comportamiento de macros activadas por binario y commit

---

## 📚 Índice de contenido

- [G-code de configuración y validación (Marlin)](#g-code-de-configuración-y-validación-marlin)
  - [🧠 Configuración de EEPROM](#-configuración-de-eeprom)
    - [🔧 Macros que controlan el comportamiento de EEPROM](#-macros-que-controlan-el-comportamiento-de-eeprom)
    - [🧪 Comandos G-code para gestión de EEPROM](#-comandos-g-code-para-gestión-de-eeprom)
    - [🧩 Buenas prácticas para EEPROM](#-buenas-prácticas-para-eeprom)
  - [🔥 Configuración y validación térmica de hotend y cama caliente](#-configuración-y-validación-térmica-de-hotend-y-cama-caliente)
    - [🔧 Macros de protección térmica y control PID](#-macros-de-protección-térmica-y-control-pid)
    - [🔧 Macros de sensores, límites y extrusión segura](#-macros-de-sensores-límites-y-extrusión-segura)
    - [🧪 Comandos G-code para validación térmica](#-comandos-g-code-para-validación-térmica)
    - [🧪 Comandos G-code para autotune y ajuste PID](#-comandos-g-code-para-autotune-y-ajuste-pid)
    - [🧩 Buenas prácticas de validación térmica](#-buenas-prácticas-de-validación-térmica)
  - [🧭 Configuración del proceso de homing y velocidades de posicionamiento](#-configuración-del-proceso-de-homing-y-velocidades-de-posicionamiento)
    - [🔧 Macros relacionadas con homing y velocidades](#-macros-relacionadas-con-homing-y-velocidades)
    - [🧪 Comandos G-code para homing y ajuste de velocidad](#-comandos-g-code-para-homing-y-ajuste-de-velocidad)
    - [🧩 Buenas prácticas de validación de homing](#-buenas-prácticas-de-validación-de-homing)
  - [📐 Configuración de nivelación de cama manual con tramming](#-configuración-de-nivelación-de-cama-manual-con-tramming)
    - [🔧 Macros relacionadas con nivelación manual](#-macros-relacionadas-con-nivelación-manual)
    - [🧪 Archivos G-code de validación por material](#-archivos-g-code-de-validación-por-material)
    - [🧪 Comandos G-code útiles en nivelación manual](#-comandos-g-code-útiles-en-nivelación-manual)
    - [🧩 Buenas prácticas de validación en tramming manual](#-buenas-prácticas-de-validación-en-tramming-manual)
  - [🧵 Configuración del estacionamiento de la boquilla](#-configuración-del-estacionamiento-de-la-boquilla)
    - [Flujo sugerido para cambio de filamento](#flujo-sugerido-para-cambio-de-filamento)
  - [🔌 Implementación del apagado automático de fuente](#-implementación-del-apagado-automático-de-fuente)
    - [Requisitos físicos](#requisitos-físicos)
    - [Configuración en Marlin](#configuración-en-marlin)
    - [Comandos G-code para control de fuente](#comandos-g-code-para-control-de-fuente)
    - [Ejemplo de cierre automático al finalizar impresión](#ejemplo-de-cierre-automático-al-finalizar-impresión)
  - [🧰 Comandos adicionales de diagnóstico](#-comandos-adicionales-de-diagnóstico)
  - [🧩 Buenas prácticas de validación](#-buenas-prácticas-de-validación)
  - [📚 Índice de contenido](#-índice-de-contenido)
