# STM32 Development with VS Code on Linux — Setup Guide

> **Course:** Taller V - Microcontroladores y Electrónica Digital
> **Scope:** STM32F411 / STM32F446 (Nucleo boards)
> **Last verified:** September 2026, against STM32Cube for VS Code Extension Pack v3.x
>
> **⚠️ Maintenance note:** The STM32Cube-for-VS-Code ecosystem has changed architecture more than once in under two years (STM32CubeCLT → bundle manager). Before reusing this guide next semester, re-verify the steps still hold.

---

## 1. Purpose & audience

This guide explains how to install and configure Visual Studio Code for STM32F411 and STM32F446 (Nucleo board) firmware development, on Linux Mint or Ubuntu (or any Debian-based distro). It's written for Physics Engineering students with solid analog electronics knowledge, but no assumed prior experience in embedded systems or advanced Linux terminal use.

**Prerequisites:**
- A laptop/PC running Linux Mint or Ubuntu (64-bit)
- A NUCLEO-F411RE or NUCLEO-F446RE board + USB cable
- `sudo` access on your machine
- Internet access (the installer downloads tools on demand)

---

## 2. Toolchain overview

Unlike STM32CubeIDE (which bundles everything), in VS Code each piece is installed/managed separately. Since late 2025, ST simplified this: you **no longer need to install STM32CubeCLT separately**. The VS Code extension ships its own *bundle manager* that automatically downloads the compiler, CMake, Ninja, the ST-LINK GDB server, and STM32CubeProgrammer the first time you need them.

```
┌─────────────────────────────────────────────────────────────┐
│                        Visual Studio Code                    │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  STM32Cube for VS Code (extension pack, ~15 pieces)   │    │
│  │  - Project creation / import                          │    │
│  │  - Bundle manager → auto-downloads:                   │    │
│  │      • GNU Tools for STM32 (GCC)                       │   │
│  │      • CMake + Ninja                                   │   │
│  │      • ST-LINK GDB server                              │   │
│  │      • STM32CubeProgrammer (CLI)                       │   │
│  │  - Own debug adapter (DAP)                              │  │
│  └──────────────────────────────────────────────────────┘    │
│                          ▲                                   │
│                          │ used starting week 7 (HAL)         │
│                  ┌───────┴────────┐                          │
│                  │  STM32CubeMX   │  (installed separately,   │
│                  │                │   generates CMake project)│
│                  └────────────────┘                          │
└──────────────────────────┬────────────────────────────────────┘
                            │ USB / SWD (ST-LINK on-board)
                     ┌──────┴───────┐
                     │ NUCLEO-F4xx  │
                     └──────────────┘
```

**Note:** STM32CubeMX is **not needed for the early weeks** of the course (bare-metal register programming). We only install it when the course introduces HAL (week 7+). See section 9.

---

## 3. Linux USB permissions (do this first)

This is the step that most often blocks students in the first lab session, so we resolve it before anything else — even before installing VS Code. On Linux, a normal user doesn't have permission by default to talk to the ST-LINK over USB; a `udev` rule is required.

**Steps:**

1. **Add your user to the `dialout` group** (needed for USB serial ports, e.g. the ST-LINK's virtual debug console):
   ```bash
   sudo usermod -aG dialout $USER
   ```
   Log out and back in (or reboot) for this to take effect.

2. **Install ST-LINK udev rules from the extension** (once installed in step 6): open the STM32Cube sidebar in VS Code → find the **"STM32Cube Resources"** section → click **"ST-Link USB Drivers"**.

3. **Verify manually if something fails:** plug in the board and run:
   ```bash
   lsusb | grep -i stm
   ```
   You should see something like `STMicroelectronics ST-LINK/V2.1`. If nothing shows up, check the USB cable (some cables are charge-only, no data lines) and try again.

---

## 4. System requirements checklist

| Requirement | Detail |
|---|---|
| Operating system | Linux Mint 21+ or Ubuntu 22.04+ (64-bit) |
| Disk space | ~3–4 GB free (VS Code + extensions + downloaded tools) |
| Internet | Needed to download packages the first time |
| ST account | Free, needed to download STM32CubeMX (section 9) |
| Hardware | NUCLEO-F411RE or NUCLEO-F446RE + data-capable USB cable |

---

## 5. Install Visual Studio Code

```bash
# Add Microsoft's official repo
sudo apt update
sudo apt install -y wget gpg
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | sudo tee /etc/apt/sources.list.d/vscode.list
rm packages.microsoft.gpg

# Install
sudo apt update
sudo apt install -y code
```

**Verify:**
```bash
code --version
```

---

## 6. Install the STM32Cube extension pack

Open VS Code, go to the Extensions view (`Ctrl+Shift+X`), search **"STM32Cube for Visual Studio Code"**, and install the pack. This automatically installs around **15 extensions** (GUI, CLI tool bundle manager, device support, its own debugger, etc.) — you don't install them individually.

You don't need to install STM32CubeCLT before this step: the extension will download the compiler and build/debug tools itself the first time you create or open a project.

---

## 7. Additional recommended extensions (optional)

Start with just the official STM32Cube pack — it covers editing, building, and debugging natively. The following are optional quality-of-life additions, useful as the course progresses:

| Extension | What it's for | When to add it |
|---|---|---|
| `usernamehw.errorlens` | Shows compiler errors inline instead of on-hover | Day 1 — reduces friction reading GCC errors |
| `ms-vscode.vscode-serial-monitor` | Integrated serial monitor, useful for USART weeks | When the course reaches USART (later) |
| `marus25.cortex-debug` | Alternative community debugger | Only if the pack's built-in debugger misbehaves — see Appendix §12 |
| `mcu-debug.peripheral-viewer` | Live peripheral-register inspection during debug | Optional, quite useful given the course's register-level focus |

---

## 8. Connect and verify the board

1. Plug in the NUCLEO board over USB.
2. Verify Linux recognizes it:
   ```bash
   lsusb | grep -i stm
   dmesg | tail -20
   ```
3. A mass-storage device should auto-mount (named something like `NODE_F411RE`), and after installing the drivers (section 3), the ST-LINK should be accessible without needing `sudo`.

---

## 9. Create your first project

There are two paths, depending on where students are in the course:

### 9.1 Weeks 1–6 (bare-metal, no HAL) — Empty project

Does not require STM32CubeMX.

1. In the STM32Cube sidebar, click **"Create empty project"** (or from the Command Palette `Ctrl+Shift+P` → "STM32").
2. Give it a name and a folder.
3. Select your board: **NUCLEO-F411RE** or **NUCLEO-F446RE**.
4. Confirm: project type **CMake**, toolchain **GCC**.
5. Generate the project and open it ("Open in this window").

**Generated structure:**
```
my-project/
├── CMakeLists.txt
├── cmake/
│   └── gcc-arm-none-eabi.cmake        # toolchain file
├── Core/
│   └── Src/
│       └── main.c                     # minimal template
├── STM32F411CEUX_FLASH.ld             # linker script
└── startup_stm32f411ceux.s            # assembly startup
```

**⚠️ Important:** this empty project does **not** include CMSIS headers (`stm32f4xx.h`, `core_cm4.h`). Without them, you can't use names like `RCC->AHB1ENR` or `GPIOA->MODER`. Copy them from the firmware package you already have installed:

```bash
# Copy CMSIS headers into your project
mkdir -p my-project/Drivers/CMSIS
cp -r ~/STM32Cube/Repository/STM32Cube_FW_F4_V1.27.1/Drivers/CMSIS/Device \
      my-project/Drivers/CMSIS/
cp -r ~/STM32Cube/Repository/STM32Cube_FW_F4_V1.27.1/Drivers/CMSIS/Include \
      my-project/Drivers/CMSIS/
```

Then add the paths to `CMakeLists.txt`:

```cmake
target_include_directories(${PROJECT_NAME} PRIVATE
    Drivers/CMSIS/Device/ST/STM32F4xx/Include
    Drivers/CMSIS/Include
)
target_compile_definitions(${PROJECT_NAME} PRIVATE STM32F411xE)
```

### 9.2 Week 7 onward (HAL) — CubeMX-generated project

1. Install STM32CubeMX (requires Java): download it from ST's site and install it.
2. In CubeMX: configure pins/peripherals → **Project Manager** tab → Toolchain: **CMake** → **Generate Code**.
3. In VS Code: **File → Open Folder** → select the generated folder. When prompted "Configure as STM32Cube project?", choose **Yes**.

---

## 10. Build, flash, and verify: blink-LED test

Replace the contents of `Core/Src/main.c` with this minimal, register-level example (no HAL), for the user LED (LD2, wired to PA5 on both NUCLEO-F411RE and NUCLEO-F446RE):

```c
#include "stm32f4xx.h"

#define LED_PIN 5U   /* PA5 = LD2 on NUCLEO-F411RE / NUCLEO-F446RE */

static void delay(volatile uint32_t count)
{
    while (count--) {
        __NOP();
    }
}

int main(void)
{
    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;          /* 1. Enable GPIOA clock */
    GPIOA->MODER &= ~(0x3U << (LED_PIN * 2));     /* 2. Clear PA5 mode bits */
    GPIOA->MODER |=  (0x1U << (LED_PIN * 2));     /* 3. PA5 as general-purpose output */

    while (1) {
        GPIOA->ODR ^= (1U << LED_PIN);            /* Toggle the LED */
        delay(1000000);
    }
}
```

**Build and flash:**

1. Build: hammer icon in the status bar, or `Ctrl+Shift+P` → **CMake: Build**.
2. Flash: `Ctrl+Shift+P` → **STM32Cube: Flash** (or the matching status-bar icon).
3. **Verification:** the board's green LED (LD2) should start blinking. If it doesn't, check section 12.

---

## 11. First debug session

1. Set a breakpoint on the `GPIOA->ODR ^= (1U << LED_PIN);` line by clicking left of the line number.
2. Press `F5` to start debugging.
3. When it stops at the breakpoint, check the value of `GPIOA->ODR` in the Variables panel or the peripheral-register view (if you installed `mcu-debug.peripheral-viewer`).
4. Resume (`F5`) and confirm the LED keeps blinking.

---

## 12. Troubleshooting appendix

- **The ST-LINK doesn't show up in `lsusb`.** Try a different USB cable (data-capable, not charge-only). Re-plug the board. Confirm you installed the drivers from section 3.

- **Permission denied when flashing/debugging without `sudo`.** Confirm your user is in the `dialout` group (section 3, step 1) and that you logged out after adding yourself.

- **Debugging hangs and times out ("Remote replied unexpectedly to 'vMustReplyEmpty'").** A known issue (2026) has been reported where the bundled ST-LINK GDB server doesn't correctly receive the STM32CubeProgrammer path when launched from a classic CubeMX-generated CMake project. Check the **"STM32Cube Debug Core"** output channel for the real error. If it persists, fall back to `marus25.cortex-debug` (already installed) with a manually configured `launch.json`.

- **Strange IntelliSense errors on valid code.** The extension defaults to `clangd` for code completion. If you see false errors on code that compiles fine, try disabling STM32Cube's clangd contribution and using `ms-vscode.cpptools` instead.

- **Using STM32CubeIDE and VS Code on the same machine.** It's safe to have both installed; they don't interfere as long as you don't open the same project in both at once. Each manages its own tools independently (they no longer share STM32CubeCLT).

---

## 13. Version reference

Versions confirmed at the time of writing (September 2026):

| Tool | Version |
|---|---|
| STM32Cube for VS Code (extension pack) | v3.x |
| GNU Tools for STM32 (GCC, auto-downloaded) | 13.3.1+st.9 |
| CMake (auto-downloaded) | 4.0.1+st.3 |
| Ninja (auto-downloaded) | 1.13.1+st.1 |
| STM32CubeMX (separate install, week 7+ only) | 6.11.0+ |

**Remember:** this ecosystem moves fast. Before each semester, confirm the "Create empty project" flow still works the same way and that the auto-downloaded versions haven't introduced breaking changes.
