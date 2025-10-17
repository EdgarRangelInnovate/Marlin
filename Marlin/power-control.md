# 🔌 Control automático de fuente con CL-400-4A (LRS-350) mediante relé o MOSFET

Este documento describe cómo implementar el apagado automático de la fuente CL-400-4A (formato LRS-350) en impresoras 3D con firmware Marlin. Se utiliza un relé o MOSFET como interfaz de corte, controlado por un pin lógico de la placa Creality V4.2.2. Esta solución permite apagar la impresora al finalizar una impresión sin modificar el formato físico original ni migrar a fuente ATX.

---

## 🧩 Requisitos físicos y eléctricos

### Fuente compatible

- Modelo: **CL-400-4A**
- Formato: **LRS-350** (compacto, metálico, montaje lateral)
- Salida típica: **24 V DC / 16.7 A**
- No incluye señal lógica PS_ON como las fuentes ATX

### Interfaz de corte: relé o MOSFET

#### Opción 1: Relé mecánico

- Módulo de relé de 1 canal, 10 A o superior
- Alimentación: 5 V o 12 V según modelo
- Control: pin lógico de la mainboard conectado al pin IN del relé
- Corte: línea positiva de 24 V entre fuente y placa base

#### Opción 2: MOSFET externo

- MOSFET canal N (ej. IRF520, IRFZ44N) con disipador
- Control: gate conectado al pin lógico de la mainboard
- Corte: línea positiva de 24 V entre fuente y placa base
- Requiere resistencia de pull-down (10 kΩ) entre gate y GND para evitar activación flotante

> ⚠️ Validar continuidad y lógica de activación antes de energizar. Usar aislamiento físico si el relé está cerca de componentes sensibles.

---

## 🔧 Compatibilidad con placa Creality V4.2.2

La placa **Creality V4.2.2** permite implementar apagado automático mediante relé o MOSFET, siempre que se seleccione un **pin digital libre**. Algunos candidatos comunes:

| Pin | Ubicación | Uso típico | ¿Disponible? |
|-----|-----------|------------|--------------|
| PA1 | J1 (Sensor de filamento) | Sensor de filamento | ✅ Si no usas sensor |
| PB2 | BLTouch/CRTouch | Nivelación automática | ✅ Si no usas Z-Probe |
| PC13 | EXP3 / LCD | Pantalla | ❌ Ocupado si usas pantalla original |

> Se recomienda usar PA1 si el sensor de filamento no está conectado. Validar en `pins_CREALITY_V4_2_2.h`.

---

## ⚙️ Configuración en Marlin

### Asignación del pin de control

```c++
#define PS_ON_PIN PA1  // Reemplaza PA1 por el pin físico que estás usando
```

> El pin debe ser digital, libre y accesible desde la mainboard.

### Activación de macros en `Configuration.h`

```c++
#define POWER_SUPPLY 0      // 0 = sin fuente ATX, pero permite control lógico
#define PSU_CONTROL         // Habilita control por software
#define PSU_DEFAULT_OFF     // Fuente apagada por defecto al arrancar
#define PSU_ACTIVE_HIGH     // HIGH activa el relé o MOSFET (ajustar según lógica)
```

> Si el relé o MOSFET se activa con nivel bajo, cambia `PSU_ACTIVE_HIGH` por `false`.

---

## 🧪 Comandos G-code para control de fuente

| Comando | Descripción |
|--------|-------------|
| `M80`   | Activa el pin `PS_ON_PIN` (enciende la fuente vía relé/MOSFET) |
| `M81`   | Desactiva el pin `PS_ON_PIN` (apaga la fuente vía relé/MOSFET) |

> Puedes enviar estos comandos desde terminal USB, menú LCD (si está habilitado), o integrarlos en el G-code de cierre de impresión.

---

## 🧪 Ejemplo de cierre automático al finalizar impresión

```gcode
; Apagar calentadores
M104 S0
M140 S0

; Apagar motores
M84

; Apagar fuente
M81
```

> Este flujo asegura que todos los componentes se apaguen antes de cortar la alimentación. Se recomienda validar que el relé o MOSFET corte físicamente la línea tras `M81`.

---

## 📐 Diagrama de conexión (texto)

### Relé

```plaintext
Mainboard PA1 ──> IN del relé ──> Relé controla línea +24 V entre fuente CL-400-4A y placa base
Fuente CL-400-4A ──> Relé ──> Placa base
```

### MOSFET

```plaintext
Mainboard PA1 ──> Gate del MOSFET
Drain ──> Entrada de alimentación de la placa base (positivo 24 V)
Source ──> Salida de la fuente CL-400-4A (positivo 24 V)
GND ──> Común entre fuente y placa
Resistencia 10 kΩ ──> Gate a GND (pull-down)
```

[Mosfet](https://www.youtube.com/watch?v=-qjtM6SZqJ0&t=10s)
[Circuito](https://www.youtube.com/watch?v=DTmVXEEyN8M)

---

## 📊 Tabla de compatibilidad por pin y lógica

| Pin | Lógica de activación | Requiere ajuste en `PSU_ACTIVE_HIGH` |
|-----|----------------------|--------------------------------------|
| PA1 | HIGH activa relé/MOSFET | `true` |
| PB2 | LOW activa relé/MOSFET | `false` |
| PC13 | No recomendado | — |

---

## 🧩 Validación técnica recomendada

- Verificar que `M80` activa el relé/MOSFET correctamente desde terminal
- Confirmar que `M81` corta la alimentación sin errores ni reinicios
- Validar que el pin `PS_ON_PIN` cambia de estado según lógica esperada (HIGH o LOW)
- Confirmar que el LCD refleja el estado de la fuente si usas MarlinUI
- Documentar el comportamiento por binario y commit en tu matriz de validación

---

## 📚 Índice de contenido

- [🔌 Control automático de fuente con CL-400-4A (LRS-350) mediante relé o MOSFET](#-control-automático-de-fuente-con-cl-400-4a-lrs-350-mediante-relé-o-mosfet)
  - [🧩 Requisitos físicos y eléctricos](#-requisitos-físicos-y-eléctricos)
    - [Fuente compatible](#fuente-compatible)
    - [Interfaz de corte: relé o MOSFET](#interfaz-de-corte-relé-o-mosfet)
      - [Opción 1: Relé mecánico](#opción-1-relé-mecánico)
      - [Opción 2: MOSFET externo](#opción-2-mosfet-externo)
  - [🔧 Compatibilidad con placa Creality V4.2.2](#-compatibilidad-con-placa-creality-v422)
  - [⚙️ Configuración en Marlin](#️-configuración-en-marlin)
    - [Asignación del pin de control](#asignación-del-pin-de-control)
    - [Activación de macros en `Configuration.h`](#activación-de-macros-en-configurationh)
  - [🧪 Comandos G-code para control de fuente](#-comandos-g-code-para-control-de-fuente)
  - [🧪 Ejemplo de cierre automático al finalizar impresión](#-ejemplo-de-cierre-automático-al-finalizar-impresión)
  - [📐 Diagrama de conexión (texto)](#-diagrama-de-conexión-texto)
    - [Relé](#relé)
    - [MOSFET](#mosfet)
  - [📊 Tabla de compatibilidad por pin y lógica](#-tabla-de-compatibilidad-por-pin-y-lógica)
  - [🧩 Validación técnica recomendada](#-validación-técnica-recomendada)
  - [📚 Índice de contenido](#-índice-de-contenido)

---

[↩️ Volver al README general](./README.md)
