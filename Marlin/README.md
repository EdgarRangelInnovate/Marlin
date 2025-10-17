# G-code de configuración y validación (Marlin)

Este documento recopila los comandos G-code relevantes para ajustes, restauración y validación de firmware en impresoras 3D con Marlin. Está diseñado para uso privado, con enfoque en trazabilidad, mantenimiento y control técnico. Todos los comandos pueden enviarse por USB desde terminales como Pronterface, OctoPrint, Repetier o desde el menú LCD si está habilitado.

---

## 🧠 Configuración de EEPROM

| Comando | Descripción |
|--------|-------------|
| `M500` | Guarda los ajustes actuales en EEPROM |
| `M501` | Carga los ajustes guardados desde EEPROM |
| `M502` | Restaura los valores por defecto del firmware (requiere `M500` para persistirlos) |
| `M503` | Muestra los ajustes actuales guardados en EEPROM |

---

## 🔥 Configuración de protección térmica y PID

| Comando | Descripción |
|--------|-------------|
| `M303 E0 S200 C8` | Autotune PID para el hotend a 200 °C con 8 ciclos |
| `M303 E-1 S60 C8` | Autotune PID para la cama a 60 °C con 8 ciclos |
| `M301` | Establece valores PID para el hotend |
| `M304` | Establece valores PID para la cama caliente |

---

## 🧭 Configuración de homing y velocidades

| Comando | Descripción |
|--------|-------------|
| `G28`   | Realiza homing en todos los ejes (X, Y, Z) |
| `M210`  | Edita velocidades de homing si está habilitado `EDITABLE_HOMING_FEEDRATE` |

---

## 📐 Configuración de nivelación de cama

| Comando | Descripción |
|--------|-------------|
| `G29`   | Ejecuta auto-nivelación si tienes Z-Probe |
| `G29 T` | Muestra el mapa de malla |
| `M420 S1` | Activa la malla guardada en EEPROM |

> ⚠️ Si usas nivelación manual (`BED_TRAMMING` sin `BED_TRAMMING_USE_PROBE`), estos comandos no aplican. El proceso se realiza desde el menú LCD.

---

## 🧪 Validación térmica de componentes

| Comando | Descripción |
|--------|-------------|
| `M105` | Consulta temperatura actual |
| `M104 S0` | Apaga el hotend |
| `M140 S0` | Apaga la cama caliente |
| `M109 S200` | Espera a que el hotend alcance 200 °C |
| `M190 S60` | Espera a que la cama alcance 60 °C |

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
  - [🔥 Configuración de protección térmica y PID](#-configuración-de-protección-térmica-y-pid)
  - [🧭 Configuración de homing y velocidades](#-configuración-de-homing-y-velocidades)
  - [📐 Configuración de nivelación de cama](#-configuración-de-nivelación-de-cama)
  - [🧪 Validación térmica de componentes](#-validación-térmica-de-componentes)
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
