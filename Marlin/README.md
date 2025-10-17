# G-code de configuración y validación (Marlin)

Este documento recopila los comandos G-code relevantes para ajustes, restauración y validación de firmware en impresoras 3D con Marlin. Todos pueden enviarse por USB desde terminales como Pronterface, OctoPrint, Repetier o desde el menú LCD si está habilitado.

---

## 🧠 EEPROM

| Comando | Descripción |
|--------|-------------|
| `M500` | Guarda los ajustes actuales en EEPROM |
| `M501` | Carga los ajustes guardados desde EEPROM |
| `M502` | Restaura los valores por defecto del firmware (requiere `M500` para persistirlos) |
| `M503` | Muestra los ajustes actuales guardados en EEPROM |

---

## 🧭 Homing y velocidades

| Comando | Descripción |
|--------|-------------|
| `G28`   | Realiza homing en todos los ejes (X, Y, Z) |
| `M210`  | Permite editar las velocidades de homing si `EDITABLE_HOMING_FEEDRATE` está activado |

---

## 🔥 Protección térmica y PID

| Comando | Descripción |
|--------|-------------|
| `M303 E0 S200 C8` | Ejecuta autotune PID para el hotend a 200 °C con 8 ciclos |
| `M303 E-1 S60 C8` | Ejecuta autotune PID para la cama a 60 °C con 8 ciclos |
| `M301` | Establece valores PID para el hotend |
| `M304` | Establece valores PID para la cama caliente |

---

## 📐 Nivelación de cama

| Comando | Descripción |
|--------|-------------|
| `G29`   | Ejecuta auto-nivelación si tienes Z-Probe y está activado |
| `G29 T` | Muestra el mapa de malla de nivelación |
| `G28`   | Siempre debe ejecutarse antes de `G29` para homing completo |
| `M420 S1` | Activa la malla de nivelación guardada en EEPROM |

> ⚠️ Si usas nivelación manual (`BED_TRAMMING` sin `BED_TRAMMING_USE_PROBE`), estos comandos no aplican. El proceso se realiza desde el menú LCD.

---

## 🧪 Validación térmica

| Comando | Descripción |
|--------|-------------|
| `M105` | Solicita la temperatura actual del hotend y cama |
| `M104 S0` | Apaga el hotend |
| `M140 S0` | Apaga la cama caliente |
| `M109 S200` | Espera a que el hotend alcance 200 °C |
| `M190 S60` | Espera a que la cama alcance 60 °C |

---

## 🧰 Otros útiles

| Comando | Descripción |
|--------|-------------|
| `M115` | Muestra información del firmware (versión, compilador, etc.) |
| `M420 V` | Muestra el estado de la malla de nivelación |
| `M119` | Muestra el estado actual de los endstops |

---

## 🧩 Recomendaciones

- Usa `M503` tras cada arranque para verificar configuración activa
- Usa `M500` tras cada ajuste validado para persistencia
- Documenta cada sesión de validación con fecha, binario y comandos usados

## 🔌 Control de apagado de fuente por G-code (`M81` / `M80`)

Este documento explica cómo implementar el control de encendido/apagado de la fuente de alimentación desde Marlin, utilizando el pin `PS_ON_PIN` de la mainboard. Esta función permite apagar la impresora automáticamente al finalizar una impresión, mejorando seguridad y eficiencia energética.

---

### 🧩 Requisitos físicos

#### 1. Fuente compatible

- Fuente ATX o similar que permita control por señal lógica (ej. pin verde de encendido)
- Alternativamente, módulo de relé o MOSFET que controle la línea de alimentación principal

#### 2. Conexión del pin de control

- Selecciona un **pin libre** en la mainboard (ej. `PA1`, `PB2`, etc.)
- Conecta ese pin al circuito de control de la fuente:
  - Si usas fuente ATX: conecta el pin al cable verde (PS_ON) mediante transistor o relé
  - Si usas relé: conecta el pin al gate del MOSFET o entrada del módulo

> ⚠️ **Advertencia:** No conectar directamente el pin al cable de 220 V o línea de poder. Siempre usar aislamiento mediante relé, optoacoplador o transistor.

---

### ⚙️ Configuración en Marlin para apagado por comando

#### 1. Asignar el pin en el archivo de pines

```c++
#define PS_ON_PIN PA1  // Reemplaza PA1 por el pin físico que estás usando
```

#### 2. Activar soporte de fuente en `Configuration.h`

```c++
#define POWER_SUPPLY 1  // 1 = ATX, 2 = XBox 360, 0 = None
```

#### 3. Opcional: mostrar control en LCD

```c++
#define PSU_CONTROL
#define PSU_DEFAULT_OFF  // Fuente apagada por defecto al arrancar
#define PSU_ACTIVE_HIGH  // Ajusta según lógica del circuito (HIGH = encendido)
```

---

### 🧪 Comandos G-code

| Comando | Descripción |
|--------|-------------|
| `M80`   | Enciende la fuente de alimentación (activa `PS_ON_PIN`) |
| `M81`   | Apaga la fuente de alimentación (desactiva `PS_ON_PIN`) |

> Puedes agregar `M81` al final del G-code de impresión para apagar la impresora automáticamente.

---

### 🧠 Validación recomendada

1. Enviar `M80` desde terminal USB → verificar que la fuente se enciende
2. Enviar `M81` → verificar que la fuente se apaga
3. Verificar que el LCD refleja el estado si usas MarlinUI
4. Registrar el comportamiento por binario y commit en tu matriz de validación

---

### 🧩 Ejemplo de integración en G-code final

```gcode
; Apagar calentadores
M104 S0
M140 S0

; Apagar motores
M84

; Apagar fuente
M81
```

---

### ✅ Recomendación para colaboradores

- Documentar el pin físico usado como `PS_ON_PIN`
- Validar si el circuito responde correctamente a `HIGH` / `LOW`
- Registrar si el apagado automático ocurre tras impresión sin errores

---

## 🧵 Estacionamiento de boquilla y cambio de filamento

Esta sección documenta cómo mover la boquilla a una posición segura durante pausas o cambios de filamento, evitando colisiones con la pieza y permitiendo extrusión de prueba antes de retomar la impresión.

---

### ⚙️ Configuración en Marlin

```c++
#define NOZZLE_PARK_FEATURE

#if ENABLED(NOZZLE_PARK_FEATURE)
  #define NOZZLE_PARK_POINT { (X_MIN_POS + 10), (Y_MAX_POS - 10), 20 }
  #define NOZZLE_PARK_MOVE          0
  #define NOZZLE_PARK_Z_RAISE_MIN   2
  #define NOZZLE_PARK_XY_FEEDRATE 100
  #define NOZZLE_PARK_Z_FEEDRATE    5
#endif
```

---

### 🧪 Comandos G-code para parqueo de boquilla

| Comando | Descripción |
|--------|-------------|
| `G27 P0` | Estaciona la boquilla en la posición definida, elevando Z si es necesario |
| `G27 P1` | Siempre eleva Z hasta la altura de parqueo antes de mover en XY |
| `G27 P2` | Eleva Z por la cantidad definida, sin exceder `Z_MAX_POS` |
| `M600`   | Inicia el proceso de cambio de filamento (si está habilitado) |

---

### 🧩 Flujo recomendado para cambio de filamento

```gcode
; Pausar impresión
M25

; Estacionar boquilla
G27 P1

; Cambiar filamento
M600  ; O hacerlo manualmente si no está habilitado

; Extruir prueba
G1 E10 F300

; Retomar impresión
M24
```

---

### ✅ Recomendación para colaboradores en parqueo de boquilla

- Validar que la boquilla se estaciona sin colisión
- Verificar que la extrusión de prueba ocurre sin obstrucción
- Registrar si el retorno a impresión (`M24`) se realiza correctamente
- Documentar el comportamiento por binario y lógica `P`

---

## 📚 Índice de contenido

- 🧠 EEPROM
- 🧭 Homing y velocidades
- 🔥 Protección térmica y PID
- 📐 Nivelación de cama
- 🧪 Validación térmica
- 🧰 Otros útiles
- 🧩 Recomendaciones
- 🔌 Control de apagado de fuente
- 🧵 Estacionamiento de boquilla y cambio de filamento
