# bare-metal-stm32 project — Timer-interrupt LED blink

> **Course:** Taller V - Microcontroladores y Electrónica Digital
> **Prerequisite:** the main installation guide completed (VS Code + STM32Cube extension pack already installed)
> **Goal:** create a project called `bare-metal-stm32` that blinks the LED using **TIM2 with interrupts** (no busy-wait), using only the most basic structures: `RCC`, `GPIOA`, `TIM2`, and bitwise operations — no HAL.
> **Approach for this project:** CMSIS headers are **linked** from ST's local repository (not copied into the project).

---

## 1. Create the empty project

Use the same wizard from the installation guide (STM32Cube sidebar → **"Create empty project"**):

1. Project name: `bare-metal-stm32` (exactly this, lowercase with hyphens).
2. Board: **NUCLEO-F411RE** or **NUCLEO-F446RE**.
3. Project type: **CMake**, toolchain: **GCC**.
4. Generate and open the project ("Open in this window").

---

## 2. Link the CMSIS headers (without copying)

Instead of copying folders into the project, we point `CMakeLists.txt` directly at ST's local repository. This avoids duplicating files, but fixes the path to this machine — if the project is shared, only one line (`CMSIS_ROOT`) needs to change.

Open `CMakeLists.txt` and add this near the top (after `project(...)`):

```cmake
# Path to the STM32Cube firmware package (adjust the version if it changes)
set(CMSIS_ROOT "/home/namontoy/STM32Cube/Repository/STM32Cube_FW_F4_V1.28.3")
```

And in your executable/target definition, add:

```cmake
target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMSIS_ROOT}/Drivers/CMSIS/Core/Include
    ${CMSIS_ROOT}/Drivers/CMSIS/Device/ST/STM32F4xx/Include
)

target_compile_definitions(${PROJECT_NAME} PRIVATE
    STM32F411xE   # use STM32F446xx if working with a NUCLEO-F446RE
)
```

### 2.1 The `system_stm32f4xx.c` file

The assembly startup file (`startup_stm32f...s`) normally calls a `SystemInit()` function before jumping to `main()`. That function lives in `system_stm32f4xx.c`, inside the same CMSIS package. Link it in as a source file too:

```cmake
target_sources(${PROJECT_NAME} PRIVATE
    ${CMSIS_ROOT}/Drivers/CMSIS/Device/ST/STM32F4xx/Source/Templates/system_stm32f4xx.c
)
```

**If, when building, you get:**
- `undefined reference to 'SystemInit'` → you were missing this file, and this step fixes it.
- `multiple definition of 'SystemInit'` → the empty project already ships its own copy; just remove this `target_sources` block.

Either outcome is normal — it depends on how current the project-creation wizard is.

---

## 3. Install and configure clangd (code navigation)

This procedure fixes "Go to Definition" / hover not working on structures like `RCC` or `GPIOA`, whose headers now live outside the project:

1. **Install the clangd binary** (the STMicroelectronics extension only calls it from PATH, it doesn't bundle it):
   ```bash
   sudo apt install clangd
   ```

2. **Disable the Microsoft C/C++ extension's IntelliSense engine** (it conflicts with clangd). In `.vscode/settings.json`:
   ```json
   {
     "C_Cpp.intelliSenseEngine": "disabled"
   }
   ```

3. **Point clangd at the compile database.** Create a `.clangd` file at the project root:
   ```yaml
   CompileFlags:
     CompilationDatabase: build/Debug
   ```

4. **Reload the window** to restart the language server: Command Palette (`Ctrl+Shift+P`) → **"Developer: Reload Window"**.

**Verify:** `Ctrl+Click` or `F12` on `RCC` or `RCC_AHB1ENR_GPIOAEN` in your code — it should jump straight into the matching CMSIS header.

---

## 4. Write the TIM2 interrupt-driven blinky

### 4.1 The plan, in registers

- `RCC->AHB1ENR` enables the GPIOA clock.
- `RCC->APB1ENR` enables the TIM2 clock.
- `TIM2->PSC` (prescaler) and `TIM2->ARR` (auto-reload) set the update-event frequency.
- `TIM2->DIER` enables the update interrupt.
- `TIM2->CR1` starts the counter.
- `NVIC_EnableIRQ(TIM2_IRQn)` enables the interrupt line in the NVIC (comes from CMSIS Core, which is why we linked that header).
- The function `TIM2_IRQHandler()` must be named **exactly that** — it's the weak symbol name the startup file already reserves in the vector table. If the name doesn't match, the interrupt never connects to your code.

### 4.2 Timing calculation

With the default post-reset clock (HSI, no PLL configured), `TIM2CLK = 16 MHz` on both boards:

- `PSC = 15999` → `16,000,000 / 16000 = 1000 Hz` → one "tick" every 1 ms.
- `ARR = 499` → 500 ticks → update event every 500 ms.
- Result: the LED toggles every 500 ms → a full blink cycle of 1 second.

### 4.3 Full code (`Core/Src/main.c`)

Function prototypes go at the top; implementations come after `main()`:

```c
#include "stm32f4xx.h"

#define LED_PIN   5U   /* PA5 = LD2 on NUCLEO-F411RE / NUCLEO-F446RE */

void TIM2_IRQHandler(void);
static void gpio_led_init(void);
static void tim2_init(void);

int main(void)
{
    gpio_led_init();
    tim2_init();

    while (1) {
        /* everything happens inside the interrupt */
    }
}

void TIM2_IRQHandler(void)
{
    if (TIM2->SR & TIM_SR_UIF) {
        TIM2->SR &= ~TIM_SR_UIF;          /* clear the update flag */
        GPIOA->ODR ^= (1U << LED_PIN);    /* toggle the LED */
    }
}

static void gpio_led_init(void)
{
    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
    GPIOA->MODER &= ~(0x3U << (LED_PIN * 2));
    GPIOA->MODER |=  (0x1U << (LED_PIN * 2));   /* PA5 as output */
}

static void tim2_init(void)
{
    RCC->APB1ENR |= RCC_APB1ENR_TIM2EN;   /* enable TIM2 clock */

    TIM2->PSC = 15999U;   /* 16 MHz / 16000 = 1 ms tick */
    TIM2->ARR = 499U;     /* 500 ticks -> event every 500 ms */

    TIM2->DIER |= TIM_DIER_UIE;   /* enable update interrupt */
    TIM2->CR1  |= TIM_CR1_CEN;    /* start the counter */

    NVIC_EnableIRQ(TIM2_IRQn);    /* enable the interrupt line in the NVIC */
}
```

---

## 5. Build, flash, and verify

1. Build: hammer icon, or `Ctrl+Shift+P` → **CMake: Build**.
2. Flash: `Ctrl+Shift+P` → **STM32Cube: Flash**.
3. **Verification:** the green LED (LD2) should blink once per second, while the main `while(1)` does nothing — all the logic lives in `TIM2_IRQHandler`.

---

## 6. Debugger check (optional)

1. Set a breakpoint inside `TIM2_IRQHandler`, on the line that toggles the LED.
2. Press `F5`.
3. Confirm the program stops there roughly every 500 ms as you resume (`F5`) repeatedly — that confirms the interrupt is firing at the expected rate.

---

## 7. Troubleshooting

- **The LED doesn't blink, but the program "runs."** Check that `TIM2_IRQHandler` is spelled exactly that way (case-sensitive). If the name doesn't match the startup file's weak symbol, the interrupt falls through to the *Default_Handler* and your code never runs.
- **The LED blinks way faster than expected, or the program hangs.** This almost always means the `TIM_SR_UIF` flag isn't being cleared inside the ISR — without clearing it, the interrupt re-fires immediately.
- **Linker error `undefined reference to 'SystemInit'` or `multiple definition of 'SystemInit'`.** See section 2.1.
- **`Ctrl+Click` doesn't navigate into CMSIS headers.** Repeat section 3 — you're probably missing the `clangd` binary install, or forgot to reload the window after creating `.clangd`.
