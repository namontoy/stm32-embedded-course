# Proyecto bare-metal-stm32 — LED parpadeante con timer e interrupciones

> **Curso:** Taller V - Microcontroladores y Electrónica Digital
> **Requisito previo:** haber completado la guía de instalación (VS Code + paquete de extensiones STM32Cube ya instalados)
> **Objetivo:** crear un proyecto llamado `bare-metal-stm32` que parpadee el LED usando **TIM2 con interrupciones** (no espera activa), usando solo las estructuras más básicas: `RCC`, `GPIOA`, `TIM2` y operaciones a nivel de bits — sin HAL.
> **Enfoque de este proyecto:** los headers de CMSIS se **enlazan** desde el repositorio local de ST (no se copian dentro del proyecto).

---

## 1. Crear el proyecto vacío

Usa el mismo asistente de la guía de instalación (barra lateral de STM32Cube → **"Create empty project"**):

1. Nombre del proyecto: `bare-metal-stm32` (exactamente así, en minúsculas y con guiones).
2. Tarjeta: **NUCLEO-F411RE** o **NUCLEO-F446RE**.
3. Tipo de proyecto: **CMake**, toolchain: **GCC**.
4. Genera y abre el proyecto ("Open in this window").

---

## 2. Enlazar los headers de CMSIS (sin copiarlos)

En lugar de copiar carpetas dentro del proyecto, apuntamos `CMakeLists.txt` directamente al repositorio local de ST. Esto evita duplicar archivos, pero significa que la ruta queda fija a esta máquina — si el proyecto se comparte, solo hay que cambiar una línea (`CMSIS_ROOT`).

Abre `CMakeLists.txt` y agrega, cerca del inicio (después de `project(...)`):

```cmake
# Ruta al paquete de firmware STM32Cube (ajusta la versión si cambia)
set(CMSIS_ROOT "/home/<home_estudiante>/STM32Cube/Repository/STM32Cube_FW_F4_V1.28.3")
```

Y en la definición de tu ejecutable/target, agrega:

```cmake
target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMSIS_ROOT}/Drivers/CMSIS/Core/Include
    ${CMSIS_ROOT}/Drivers/CMSIS/Device/ST/STM32F4xx/Include
)

target_compile_definitions(${PROJECT_NAME} PRIVATE
    STM32F411xE   # usa STM32F446xx si trabajas con NUCLEO-F446RE
)
```

### 2.1 El archivo `system_stm32f4xx.c`

El archivo de arranque en ensamblador (`startup_stm32f...s`) normalmente llama a una función `SystemInit()` antes de saltar a `main()`. Esa función vive en `system_stm32f4xx.c`, dentro del mismo paquete CMSIS. Enlázalo también como fuente:

```cmake
target_sources(${PROJECT_NAME} PRIVATE
    ${CMSIS_ROOT}/Drivers/CMSIS/Device/ST/STM32F4xx/Source/Templates/system_stm32f4xx.c
)
```

**Si al compilar obtienes:**
- `undefined reference to 'SystemInit'` → te faltaba este archivo, y este paso lo soluciona.
- `multiple definition of 'SystemInit'` → el proyecto vacío ya traía su propia copia; simplemente elimina este bloque `target_sources`.

Cualquiera de los dos resultados es normal — depende de qué tan actualizado esté el asistente de creación de proyectos.

---

## 3. Instalar y configurar clangd (navegación de código)

Este procedimiento resuelve el problema de "Go to Definition" / hover que no funciona en estructuras como `RCC` o `GPIOA`, cuyos headers ahora viven fuera del proyecto:

1. **Instala el binario de clangd** (la extensión de STMicroelectronics solo lo invoca desde el PATH, no lo incluye):
   ```bash
   sudo apt install clangd
   ```

2. **Desactiva el motor de IntelliSense de la extensión de Microsoft C/C++** (entra en conflicto con clangd). En `.vscode/settings.json`:
   ```json
   {
     "C_Cpp.intelliSenseEngine": "disabled"
   }
   ```

3. **Apunta clangd a la base de datos de compilación.** Crea un archivo `.clangd` en la raíz del proyecto:
   ```yaml
   CompileFlags:
     CompilationDatabase: build/Debug
   ```

4. **Recarga la ventana** para reiniciar el servidor de lenguaje: paleta de comandos (`Ctrl+Shift+P`) → **"Developer: Reload Window"**.

**Verificación:** haz `Ctrl+Click` o `F12` sobre `RCC` o `RCC_AHB1ENR_GPIOAEN` en tu código — debería saltar directamente al header de CMSIS correspondiente.

---

## 4. Escribir el blinky con TIM2 por interrupción

### 4.1 La idea, en registros

- `RCC->AHB1ENR` habilita el reloj de GPIOA.
- `RCC->APB1ENR` habilita el reloj de TIM2.
- `TIM2->PSC` (prescaler) y `TIM2->ARR` (auto-reload) definen la frecuencia del evento de actualización.
- `TIM2->DIER` habilita la interrupción de actualización (*update interrupt*).
- `TIM2->CR1` enciende el contador.
- `NVIC_EnableIRQ(TIM2_IRQn)` habilita la línea de interrupción en el NVIC (viene de CMSIS Core, por eso enlazamos ese header).
- La función `TIM2_IRQHandler()` debe llamarse **exactamente así** — es el nombre del símbolo débil (*weak symbol*) que el archivo de arranque ya reserva en la tabla de vectores. Si el nombre no coincide, la interrupción nunca se conecta a tu código.

### 4.2 Cálculo de tiempos

Con el reloj por defecto tras el reset (HSI, sin PLL configurado), `TIM2CLK = 16 MHz` en ambas tarjetas:

- `PSC = 15999` → `16 000 000 / 16000 = 1000 Hz` → un "tick" cada 1 ms.
- `ARR = 499` → 500 ticks → evento de actualización cada 500 ms.
- Resultado: el LED cambia de estado cada 500 ms → ciclo completo de parpadeo de 1 segundo.

### 4.3 Código completo (`Core/Src/main.c`)

Los prototipos de las funciones van al principio; las implementaciones, después de `main()`:

```c
#include "stm32f4xx.h"

#define LED_PIN   5U   /* PA5 = LD2 en NUCLEO-F411RE / NUCLEO-F446RE */

void TIM2_IRQHandler(void);
static void gpio_led_init(void);
static void tim2_init(void);

int main(void)
{
    gpio_led_init();
    tim2_init();

    while (1) {
        /* todo ocurre dentro de la interrupción */
    }
}

void TIM2_IRQHandler(void)
{
    if (TIM2->SR & TIM_SR_UIF) {
        TIM2->SR &= ~TIM_SR_UIF;          /* limpiar la bandera de actualización */
        GPIOA->ODR ^= (1U << LED_PIN);    /* alternar el LED */
    }
}

static void gpio_led_init(void)
{
    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
    GPIOA->MODER &= ~(0x3U << (LED_PIN * 2));
    GPIOA->MODER |=  (0x1U << (LED_PIN * 2));   /* PA5 como salida */
}

static void tim2_init(void)
{
    RCC->APB1ENR |= RCC_APB1ENR_TIM2EN;   /* habilitar reloj de TIM2 */

    TIM2->PSC = 15999U;   /* 16 MHz / 16000 = tick de 1 ms */
    TIM2->ARR = 499U;     /* 500 ticks -> evento cada 500 ms */

    TIM2->DIER |= TIM_DIER_UIE;   /* habilitar interrupción de actualización */
    TIM2->CR1  |= TIM_CR1_CEN;    /* iniciar el contador */

    NVIC_EnableIRQ(TIM2_IRQn);    /* habilitar la línea de interrupción en el NVIC */
}
```

---

## 5. Compilar, flashear y verificar

1. Compila: ícono de martillo, o `Ctrl+Shift+P` → **CMake: Build**.
2. Flashea: `Ctrl+Shift+P` → **STM32Cube: Flash**.
3. **Verificación:** el LED verde (LD2) debe parpadear una vez por segundo, sin que el `while(1)` principal haga nada — toda la lógica vive en `TIM2_IRQHandler`.

---

## 6. Verificación con el depurador (opcional)

1. Coloca un punto de interrupción dentro de `TIM2_IRQHandler`, en la línea que alterna el LED.
2. Presiona `F5`.
3. Confirma que el programa se detiene ahí aproximadamente cada 500 ms al continuar (`F5`) repetidamente — eso confirma que la interrupción se está disparando al ritmo esperado.

---

## 7. Solución de problemas

- **El LED no parpadea, pero el programa "corre".** Revisa que `TIM2_IRQHandler` esté escrito exactamente con ese nombre (sensible a mayúsculas). Si el nombre no coincide con el símbolo débil del archivo de arranque, la interrupción cae en el *Default_Handler* y tu código nunca se ejecuta.
- **El LED parpadea muchísimo más rápido de lo esperado, o el programa se cuelga.** Casi siempre significa que la bandera `TIM_SR_UIF` no se está limpiando dentro del ISR — sin limpiarla, la interrupción se vuelve a disparar de inmediato.
- **Error de enlazado `undefined reference to 'SystemInit'` o `multiple definition of 'SystemInit'`.** Ver sección 2.1.
- **`Ctrl+Click` no navega a los headers de CMSIS.** Repite la sección 3 — probablemente falta instalar el binario de `clangd` o falta recargar la ventana después de crear `.clangd`.
