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

## 🧵 Configuración y validación del estacionamiento de la boquilla

La función de estacionamiento de boquilla permite mover el cabezal a una posición segura durante pausas o cambios de filamento, evitando colisiones con la pieza impresa y permitiendo extrusión de prueba antes de retomar la impresión. Esta función se activa mediante el comando `G27` y puede integrarse en flujos de mantenimiento o recuperación.

---

### 🔧 Macros relacionadas con estacionamiento de boquilla

```c++
#define NOZZLE_PARK_FEATURE             // Habilita la función de parqueo
#define NOZZLE_PARK_POINT { (X_MIN_POS + 10), (Y_MAX_POS - 10), 20 }  // Posición segura
#define NOZZLE_PARK_MOVE 0             // Movimiento en XY simultáneo
#define NOZZLE_PARK_Z_RAISE_MIN 2      // Elevación mínima en Z antes de mover
#define NOZZLE_PARK_XY_FEEDRATE 100    // Velocidad de movimiento en XY
#define NOZZLE_PARK_Z_FEEDRATE 5       // Velocidad de elevación en Z
```

| Macro | Descripción | Recomendación |
|-------|-------------|----------------|
| `NOZZLE_PARK_FEATURE` | Activa el comando `G27` para estacionar la boquilla | ✅ Activar |
| `NOZZLE_PARK_POINT` | Define la posición segura de parqueo | ✅ Ajustar según geometría |
| `NOZZLE_PARK_MOVE` | Define el orden de movimiento | ⚖️ `0` si no hay obstáculos, `3` o `4` si hay piezas altas |
| `NOZZLE_PARK_Z_RAISE_MIN` | Elevación mínima antes de mover en XY | ✅ Evita colisiones |
| `NOZZLE_PARK_*_FEEDRATE` | Velocidades de movimiento | ✅ Ajustar según mecánica y seguridad |

---

### 🧪 Comandos G-code para estacionamiento y cambio de filamento

| Comando | Descripción |
|--------|-------------|
| `G27 P0` | Estaciona la boquilla si Z está por debajo del punto |
| `G27 P1` | Siempre eleva Z hasta la altura de parqueo |
| `G27 P2` | Eleva Z por la cantidad definida sin exceder `Z_MAX_POS` |
| `M600`   | Inicia cambio de filamento (si está habilitado) |
| `M25` / `M24` | Pausa y retoma impresión |

---

### 🧪 Flujo recomendado para cambio de filamento

```gcode
M25         ; Pausar impresión
G27 P1      ; Estacionar boquilla
M600        ; Cambiar filamento
G1 E10 F300 ; Extruir prueba
M24         ; Retomar impresión
```

> Este flujo permite validar la extrusión antes de continuar la impresión, evitando errores por obstrucción o mala carga del filamento.

---

### 🧩 Buenas prácticas de validación

- Verificar que la boquilla se eleva antes de moverse en XY
- Confirmar que la posición de parqueo no interfiere con la pieza impresa
- Validar que `M600` realiza correctamente el cambio de filamento (si está habilitado)
- Documentar el comportamiento por binario y lógica `P` usada en `G27`

---

## 🧰 Comandos adicionales de diagnóstico

Estos comandos permiten validar el estado del firmware, sensores y malla antes de ejecutar flujos de mantenimiento, recuperación o impresión crítica. Son útiles para depurar fallos, confirmar configuración activa y documentar el entorno técnico por binario.

| Comando | Descripción | Uso recomendado |
|--------|-------------|------------------|
| `M115` | Muestra versión del firmware, compilador, fecha y opciones habilitadas | ✅ Al inicio de sesión técnica o tras actualización |
| `M420 V` | Muestra la malla de nivelación activa (si está habilitada) | ✅ Antes de imprimir piezas grandes o sensibles |
| `M119` | Muestra el estado actual de los endstops (activado/desactivado) | ✅ Para validar sensores físicos y lógica de inversión |

---

### 🧩 Buenas prácticas de diagnóstico

- Registrar la salida de `M115` por binario y commit para trazabilidad
- Confirmar que `M420 V` refleja la malla esperada tras `G29` o carga desde EEPROM
- Validar que `M119` responde correctamente al presionar manualmente cada endstop
- Documentar resultados por sesión si se detectan inconsistencias o fallos intermitentes

---

> Estos comandos no modifican el estado de la impresora, pero permiten confirmar que la configuración activa coincide con la esperada. Son especialmente útiles tras cambios de firmware, ajustes físicos o migraciones de hardware.

---

## 🖥️ Controlador de pantalla: CR10_STOCKDISPLAY

La pantalla instalada corresponde al modelo **CR10_STOCKDISPLAY**, utilizada en impresoras Creality como Ender-3, CR-10 y CR-7. Este tipo de pantalla es una **12864 LCD gráfica** con perilla (encoder), conectada mediante los puertos **EXP1 y EXP2** a la placa base.

---

### 🔧 Configuración en Marlin

```c++
#define CR10_STOCKDISPLAY
#if ENABLED(CR10_STOCKDISPLAY)
  #define RET6_12864_LCD  // Controlador específico del SoC (RET o VET)
#endif
```

> Esta configuración activa el soporte para pantallas gráficas de 128x64 píxeles con controlador RET6, compatible con placas Creality V4.2.2.

---

### 📐 Conexión física

- **EXP1**: Comunicación principal (SPI paralelo)
- **EXP2**: Alimentación + señales adicionales
- **Beeper, encoder, botón**: Integrados en el PCB frontal

> No se requiere conexión serial ni firmware externo como en pantallas táctiles DWIN. La pantalla se actualiza junto con el firmware principal de Marlin.

---

### 🧩 Compatibilidad y recomendaciones

- ✅ Compatible con Marlin 2.x y placas Creality V4.2.2
- ✅ Permite navegación por menú, control manual y visualización de estado
- ⚠️ No compatible con OctoPrint ni Klipper directamente (requiere pantalla virtual o remota)
- ⚠️ No usar EXP2 para UART o Raspberry Pi sin aislamiento lógico

---

### 🧪 Validación técnica recomendada para el display

- Confirmar que el menú LCD aparece tras flashear Marlin
- Validar que el encoder responde correctamente al girar y presionar
- Verificar que el beeper suena en eventos críticos (inicio, error, pausa)
- Documentar el comportamiento por binario y commit si se modifica el controlador

---

> Para migrar a pantalla táctil o integrar control remoto, se recomienda documentar el pinout actual y liberar EXP2 si se desea usar UART. En ese caso, se debe desactivar `CR10_STOCKDISPLAY` y activar el controlador correspondiente (`DWIN_CREALITY_LCD`, `TFT_CLASSIC_UI`, etc.).

---

## 🧩 Mejoras posteriores

Este bloque agrupa mejoras opcionales que pueden implementarse tras validar la configuración térmica, mecánica y lógica de la impresora. Cada mejora puede documentarse como módulo independiente si se desea trazabilidad por binario, impacto técnico o compatibilidad de hardware.

---

### 🔇 Activar modo silencioso y adaptar sensor de cama

- Requiere instalar drivers silenciosos (ej. TMC2208, TMC2209) y ajustar corriente en firmware
- Adaptar sensor de cama (BLTouch, CRTouch) implica validar pines disponibles y macros de nivelación
- [Ver video explicativo](https://www.youtube.com/watch?v=neS7lB7fCww)

---

### 🧵 Instalar sensor de filamento

- Permite pausar impresión si el filamento se agota
- Requiere habilitar `FILAMENT_RUNOUT_SENSOR` en Marlin y definir `FIL_RUNOUT_PIN`
- Puede integrarse con `M600` para cambio automático

---

### ⚙️ Instalar segundo motor para eje Z

- Mejora estabilidad en impresiones altas
- Requiere duplicar driver Z o usar splitter si la placa lo permite
- Ajustar `NUM_Z_STEPPER_DRIVERS` y `Z_MULTI_ENDSTOPS` si se usan dos endstops

---

### 🔌 Implementación del apagado automático de fuente

- Debido a la extensión y trazabilidad requerida, esta mejora se documenta por separado:
  [Ver documento completo](./power-control.md)

---

### 🧠 Instalar Raspberry Pi (OctoPrint, Klipper, etc.)

- Permite control remoto, monitoreo y mejoras de flujo
- Requiere conexión USB directa y configuración de puertos
- [Ver video explicativo](https://youtu.be/o5R3KZeMPwA?si=-5OVaMIr1FrKkycg)

#### 🔗 Integración del conector EXP2 con Raspberry Pi

Este módulo documenta cómo aprovechar el conector **EXP2** de la pantalla Creality (Ender 3 V2 Pro) para establecer comunicación con una Raspberry Pi. Esta integración permite control remoto, monitoreo, o uso de firmware alternativo como Klipper u OctoPrint.

---

##### 🧩 Requisitos físicos y eléctricos

- Pantalla Creality con conectores **EXP1** y **EXP2** visibles
- Raspberry Pi (modelo 3B+, 4, Zero 2 W o superior)
- Cables Dupont o adaptador IDC 10 pines a GPIO
- Fuente de alimentación estable para la Raspberry Pi (no usar directamente desde EXP2)

---

##### 📐 Pinout típico de EXP2

| Pin | Señal | Uso en Raspberry Pi |
|-----|-------|----------------------|
| 1   | GND   | GND (pin 6, 9, 14, etc.) |
| 2   | VCC   | ⚠️ No conectar (riesgo de sobrevoltaje) |
| 3   | RX    | GPIO15 (RXD) |
| 4   | TX    | GPIO14 (TXD) |
| 5–10 | NC / control | No conectar directamente |

> ⚠️ Validar voltaje de señal: si EXP2 entrega 5 V en TX/RX, usar divisor resistivo o adaptador lógico para proteger GPIO de la Raspberry Pi (que opera a 3.3 V).

---

##### ⚙️ Configuración en Raspberry Pi

1. Habilitar UART en `raspi-config`:

   ```bash
   sudo raspi-config
   → Interface Options → Serial → Disable shell, enable hardware UART
   ```

2. Conectar RX/TX cruzado:
   - EXP2 TX → GPIO15 (RXD)
   - EXP2 RX → GPIO14 (TXD)

3. Validar comunicación con:

   ```bash
   screen /dev/serial0 115200
   ```

> Si usas Klipper, define el puerto en `printer.cfg` como `/dev/serial0` o el alias correspondiente.

---

##### 🧪 Validación técnica recomendada

- Confirmar que la pantalla responde a comandos desde Raspberry Pi
- Validar que no hay ruido eléctrico ni interferencia en la línea UART
- Documentar el comportamiento por binario y sesión
- Si usas OctoPrint, validar que el puerto aparece en `/dev` y que la velocidad es compatible (115200 o 250000)

---

##### 🧩 Buenas prácticas

- No alimentar la Raspberry Pi desde la pantalla ni desde EXP2
- Usar adaptador lógico si el voltaje de señal excede 3.3 V
- Documentar el pinout real de tu pantalla si difiere del estándar
- Si usas pantalla táctil DWIN, EXP2 puede estar bloqueado o redirigido internamente

---

¿Quieres que esta sección se convierta en un módulo independiente (`exp2-rpi.md`) o que prepare una tabla de compatibilidad por modelo de pantalla y tipo de conexión (UART, SPI, etc.)? También puedo ayudarte a validar si tu pantalla permite carga de firmware por SD o si está bloqueada por diseño.

---

### 📷 Instalar cámara

- Compatible con OctoPrint, Klipper o monitoreo local
- Requiere validar compatibilidad USB o CSI si se usa Raspberry Pi
- Puede integrarse con detección de fallos o timelapse

---

> Se recomienda documentar cada mejora como módulo independiente (`filament-sensor.md`, `dual-z.md`, `raspberry.md`, etc.) si se desea trazabilidad por commit, binario o impacto técnico.

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
  - [🧵 Configuración y validación del estacionamiento de la boquilla](#-configuración-y-validación-del-estacionamiento-de-la-boquilla)
    - [🔧 Macros relacionadas con estacionamiento de boquilla](#-macros-relacionadas-con-estacionamiento-de-boquilla)
    - [🧪 Comandos G-code para estacionamiento y cambio de filamento](#-comandos-g-code-para-estacionamiento-y-cambio-de-filamento)
    - [🧪 Flujo recomendado para cambio de filamento](#-flujo-recomendado-para-cambio-de-filamento)
    - [🧩 Buenas prácticas de validación](#-buenas-prácticas-de-validación)
  - [🧰 Comandos adicionales de diagnóstico](#-comandos-adicionales-de-diagnóstico)
    - [🧩 Buenas prácticas de diagnóstico](#-buenas-prácticas-de-diagnóstico)
  - [🖥️ Controlador de pantalla: CR10\_STOCKDISPLAY](#️-controlador-de-pantalla-cr10_stockdisplay)
    - [🔧 Configuración en Marlin](#-configuración-en-marlin)
    - [📐 Conexión física](#-conexión-física)
    - [🧩 Compatibilidad y recomendaciones](#-compatibilidad-y-recomendaciones)
    - [🧪 Validación técnica recomendada para el display](#-validación-técnica-recomendada-para-el-display)
  - [🧩 Mejoras posteriores](#-mejoras-posteriores)
    - [🔇 Activar modo silencioso y adaptar sensor de cama](#-activar-modo-silencioso-y-adaptar-sensor-de-cama)
    - [🧵 Instalar sensor de filamento](#-instalar-sensor-de-filamento)
    - [⚙️ Instalar segundo motor para eje Z](#️-instalar-segundo-motor-para-eje-z)
    - [🔌 Implementación del apagado automático de fuente](#-implementación-del-apagado-automático-de-fuente)
    - [🧠 Instalar Raspberry Pi (OctoPrint, Klipper, etc.)](#-instalar-raspberry-pi-octoprint-klipper-etc)
      - [🔗 Integración del conector EXP2 con Raspberry Pi](#-integración-del-conector-exp2-con-raspberry-pi)
        - [🧩 Requisitos físicos y eléctricos](#-requisitos-físicos-y-eléctricos)
        - [📐 Pinout típico de EXP2](#-pinout-típico-de-exp2)
        - [⚙️ Configuración en Raspberry Pi](#️-configuración-en-raspberry-pi)
        - [🧪 Validación técnica recomendada](#-validación-técnica-recomendada)
        - [🧩 Buenas prácticas](#-buenas-prácticas)
    - [📷 Instalar cámara](#-instalar-cámara)
  - [📚 Índice de contenido](#-índice-de-contenido)
