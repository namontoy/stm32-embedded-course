# Desarrollo STM32 con VS Code en Linux — Guía de instalación

> **Curso:** Taller V - Microcontroladores y Electrónica Digital
> **Alcance:** STM32F411 / STM32F446 (tarjetas Nucleo)
> **Última verificación:** Septiembre 2026, sobre STM32Cube for VS Code Extension Pack v3.x
>
> **⚠️ Nota de mantenimiento:** El ecosistema STM32Cube para VS Code cambia de arquitectura con frecuencia (pasó de STM32CubeCLT a un gestor de paquetes/"bundle manager" en menos de dos años). Antes de reusar esta guía en un nuevo semestre, verifica que los pasos siguen siendo válidos.

---

## 1. Propósito y audiencia

Esta guía explica cómo instalar y configurar Visual Studio Code para desarrollar firmware en microcontroladores STM32F411 y STM32F446 (tarjetas Nucleo), en Linux Mint o Ubuntu (o cualquier distribución basada en Debian). Está pensada para estudiantes de Ingeniería Física con buenos conocimientos de electrónica analógica, pero sin experiencia previa necesaria en sistemas embebidos ni en el uso de terminal Linux más allá de lo básico.

**Requisitos previos:**
- Una laptop/PC con Linux Mint o Ubuntu (64-bit)
- Una tarjeta NUCLEO-F411RE o NUCLEO-F446RE + cable USB
- Acceso a `sudo` en tu máquina
- Conexión a internet (el instalador descarga herramientas bajo demanda)

---

## 2. Panorama general de herramientas

A diferencia de STM32CubeIDE (que trae todo integrado), en VS Code cada pieza se instala/gestiona por separado. Desde finales de 2025, ST simplificó esto: ya **no necesitas instalar STM32CubeCLT por separado**. La extensión de VS Code trae su propio "gestor de paquetes" (*bundle manager*) que descarga el compilador, CMake, Ninja, el servidor GDB de ST-LINK y STM32CubeProgrammer automáticamente, la primera vez que los necesitas.

```
┌─────────────────────────────────────────────────────────────┐
│                        Visual Studio Code                    │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  STM32Cube for VS Code (paquete de extensiones,       │    │
│  │  ~15 piezas)                                           │    │
│  │  - Creación / importación de proyectos                 │    │
│  │  - Gestor de paquetes → descarga automáticamente:      │    │
│  │      • GNU Tools for STM32 (GCC)                       │    │
│  │      • CMake + Ninja                                   │    │
│  │      • Servidor GDB de ST-LINK                         │    │
│  │      • STM32CubeProgrammer (CLI)                       │    │
│  │  - Depurador propio (DAP)                              │    │
│  └──────────────────────────────────────────────────────┘    │
│                          ▲                                   │
│                          │ se usa desde la semana 7 (HAL)     │
│                  ┌───────┴────────┐                          │
│                  │  STM32CubeMX   │  (se instala aparte,      │
│                  │                │   genera proyecto CMake)  │
│                  └────────────────┘                          │
└──────────────────────────┬────────────────────────────────────┘
                            │ USB / SWD (ST-LINK integrado)
                     ┌──────┴───────┐
                     │ NUCLEO-F4xx  │
                     └──────────────┘
```

**Nota:** STM32CubeMX **no es necesario para las primeras semanas del curso** (programación en registros, bare-metal). Solo lo instalaremos cuando el curso introduzca HAL (semana 7+). Ver sección 9.

---

## 3. Permisos USB en Linux (hazlo primero)

Este es el paso que con más frecuencia bloquea a estudiantes en el primer laboratorio, así que lo resolvemos antes que nada — antes incluso de instalar VS Code. En Linux, un usuario normal no tiene permiso por defecto para hablar con el ST-LINK vía USB; hace falta una regla `udev`.

**Pasos:**

1. **Agrega tu usuario al grupo `dialout`** (necesario para los puertos seriales USB, ej. la consola de depuración virtual del ST-LINK):
   ```bash
   sudo usermod -aG dialout $USER
   ```
   Cierra sesión y vuelve a entrar (o reinicia) para que el cambio tenga efecto.

2. **Instala las reglas udev del ST-LINK desde la extensión** (una vez instalada en el paso 6): abre el panel lateral de STM32Cube en VS Code → busca la sección **"STM32Cube Resources"** → haz clic en **"ST-Link USB Drivers"**.

3. **Verifica manualmente si algo falla:** conecta la tarjeta y ejecuta:
   ```bash
   lsusb | grep -i stm
   ```
   Deberías ver algo como `STMicroelectronics ST-LINK/V2.1`. Si no aparece nada, revisa el cable USB (algunos cables son solo de carga, sin datos) y vuelve a intentarlo.

---

## 4. Lista de verificación del sistema

| Requisito | Detalle |
|---|---|
| Sistema operativo | Linux Mint 21+ o Ubuntu 22.04+ (64-bit) |
| Espacio en disco | ~3–4 GB libres (VS Code + extensiones + herramientas descargadas) |
| Internet | Necesario para descargar paquetes la primera vez |
| Cuenta ST | Gratuita, necesaria para descargar STM32CubeMX (sección 9) |
| Hardware | NUCLEO-F411RE o NUCLEO-F446RE + cable USB con datos |

---

## 5. Instalar Visual Studio Code

```bash
# Agregar el repositorio oficial de Microsoft
sudo apt update
sudo apt install -y wget gpg
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | sudo tee /etc/apt/sources.list.d/vscode.list
rm packages.microsoft.gpg

# Instalar
sudo apt update
sudo apt install -y code
```

**Verifica:**
```bash
code --version
```

---

## 6. Instalar el paquete de extensiones STM32Cube

Abre VS Code, ve a la vista de Extensiones (`Ctrl+Shift+X`), busca **"STM32Cube for Visual Studio Code"** e instala el paquete. Esto instala automáticamente alrededor de **15 extensiones** (interfaz gráfica, gestor de paquetes de herramientas, soporte de dispositivos, depurador propio, etc.) — no necesitas instalarlas una por una.

No es necesario instalar STM32CubeCLT antes de este paso: la extensión descargará el compilador y las herramientas de compilación/depuración por sí misma la primera vez que crees o abras un proyecto.

---

## 7. Extensiones adicionales recomendadas (opcional)

Empieza solo con el paquete oficial de STM32Cube — cubre edición, compilación y depuración de forma nativa. Las siguientes son mejoras opcionales, útiles a medida que el curso avanza:

| Extensión | Para qué sirve | ¿Cuándo instalarla? |
|---|---|---|
| `usernamehw.errorlens` | Muestra errores del compilador en la misma línea, no solo al pasar el mouse | Desde el día 1 — reduce fricción para quienes recién leen errores de GCC |
| `ms-vscode.vscode-serial-monitor` | Monitor serial integrado, útil para las semanas de USART | Cuando el curso llegue a USART (más adelante) |
| `marus25.cortex-debug` | Depurador comunitario alternativo | Solo si el depurador propio de la extensión falla — ver Apéndice §12 |
| `mcu-debug.peripheral-viewer` | Inspección en vivo de registros de periféricos durante depuración | Opcional, muy útil dado el enfoque del curso en registros |

---

## 8. Conectar y verificar la tarjeta

1. Conecta la NUCLEO por USB.
2. Verifica que Linux la reconozca:
   ```bash
   lsusb | grep -i stm
   dmesg | tail -20
   ```
3. Debería aparecer un dispositivo de almacenamiento montado automáticamente (llamado algo como `NODE_F411RE`) y, tras instalar los drivers (sección 3), el ST-LINK debería quedar accesible sin necesitar `sudo`.

---

## 9. Crear tu primer proyecto

Hay dos caminos, según en qué parte del curso estén los estudiantes:

### 9.1 Semanas 1–6 (bare-metal, sin HAL) — Proyecto vacío

No requiere STM32CubeMX.

1. En la barra lateral de STM32Cube, haz clic en **"Create empty project"** (o desde la paleta de comandos `Ctrl+Shift+P` → "STM32").
2. Asigna un nombre y una carpeta.
3. Selecciona tu tarjeta: **NUCLEO-F411RE** o **NUCLEO-F446RE**.
4. Confirma: tipo de proyecto **CMake**, toolchain **GCC**.
5. Genera el proyecto y ábrelo ("Open in this window").

**Estructura generada:**
```
mi-proyecto/
├── CMakeLists.txt
├── cmake/
│   └── gcc-arm-none-eabi.cmake        # archivo de toolchain
├── Core/
│   └── Src/
│       └── main.c                     # plantilla mínima
├── STM32F411CEUX_FLASH.ld             # script del linker
└── startup_stm32f411ceux.s            # arranque en ensamblador
```

**⚠️ Importante:** este proyecto vacío **no incluye los headers de CMSIS** (`stm32f4xx.h`, `core_cm4.h`). Sin ellos no puedes usar nombres como `RCC->AHB1ENR` o `GPIOA->MODER`. Cópialos desde el paquete de firmware que ya tienes instalado:

```bash
# Copia los headers CMSIS a tu proyecto
mkdir -p mi-proyecto/Drivers/CMSIS
cp -r ~/STM32Cube/Repository/STM32Cube_FW_F4_V1.27.1/Drivers/CMSIS/Device \
      mi-proyecto/Drivers/CMSIS/
cp -r ~/STM32Cube/Repository/STM32Cube_FW_F4_V1.27.1/Drivers/CMSIS/Include \
      mi-proyecto/Drivers/CMSIS/
```

Luego agrega las rutas a `CMakeLists.txt`:

```cmake
target_include_directories(${PROJECT_NAME} PRIVATE
    Drivers/CMSIS/Device/ST/STM32F4xx/Include
    Drivers/CMSIS/Include
)
target_compile_definitions(${PROJECT_NAME} PRIVATE STM32F411xE)
```

### 9.2 Semana 7 en adelante (HAL) — Proyecto desde CubeMX

1. Instala STM32CubeMX (requiere Java): descárgalo desde el sitio de ST e instálalo.
2. En CubeMX: configura pines/periféricos → pestaña **Project Manager** → Toolchain: **CMake** → **Generate Code**.
3. En VS Code: **File → Open Folder** → selecciona la carpeta generada. Cuando aparezca el aviso "Configure as STM32Cube project?", elige **Yes**.

---

## 10. Compilar, flashear y verificar: prueba de LED parpadeante

Reemplaza el contenido de `Core/Src/main.c` con este ejemplo mínimo, a nivel de registro (sin HAL), para el LED de usuario (LD2, conectado a PA5 tanto en NUCLEO-F411RE como en NUCLEO-F446RE):

```c
#include "stm32f4xx.h"

#define LED_PIN 5U   /* PA5 = LD2 en NUCLEO-F411RE / NUCLEO-F446RE */

static void delay(volatile uint32_t count)
{
    while (count--) {
        __NOP();
    }
}

int main(void)
{
    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;          /* 1. Habilitar reloj de GPIOA */
    GPIOA->MODER &= ~(0x3U << (LED_PIN * 2));     /* 2. Limpiar bits de modo de PA5 */
    GPIOA->MODER |=  (0x1U << (LED_PIN * 2));     /* 3. PA5 como salida de propósito general */

    while (1) {
        GPIOA->ODR ^= (1U << LED_PIN);            /* Alternar el LED */
        delay(1000000);
    }
}
```

**Compilar y flashear:**

1. Compila: icono de martillo en la barra de estado, o `Ctrl+Shift+P` → **CMake: Build**.
2. Flashea: `Ctrl+Shift+P` → **STM32Cube: Flash** (o el ícono correspondiente en la barra de estado).
3. **Verificación:** el LED verde (LD2) de la tarjeta debe empezar a parpadear. Si no parpadea, revisa la sección 12.

---

## 11. Primera sesión de depuración

1. Coloca un punto de interrupción (*breakpoint*) en la línea `GPIOA->ODR ^= (1U << LED_PIN);` haciendo clic a la izquierda del número de línea.
2. Presiona `F5` para iniciar la depuración.
3. Cuando se detenga en el breakpoint, revisa el valor de `GPIOA->ODR` en el panel de variables o registros de periféricos (si instalaste `mcu-debug.peripheral-viewer`).
4. Continúa (`F5`) y confirma que el LED sigue parpadeando.

---

## 12. Apéndice de solución de problemas

- **El ST-LINK no aparece en `lsusb`.** Prueba otro cable USB (con datos, no solo carga). Reconecta la tarjeta. Confirma que instalaste los drivers de la sección 3.

- **Permisos denegados al flashear/depurar sin `sudo`.** Confirma que tu usuario está en el grupo `dialout` (sección 3, paso 1) y que cerraste sesión después de agregarte.

- **La depuración se queda esperando y termina en timeout ("Remote replied unexpectedly to 'vMustReplyEmpty'").** Se ha reportado un problema conocido (2026) donde el servidor GDB de ST-LINK integrado no recibe correctamente la ruta de STM32CubeProgrammer al lanzarse desde un proyecto CMake clásico generado por CubeMX. Revisa el canal de salida **"STM32Cube Debug Core"** para ver el error real. Si persiste, usa `marus25.cortex-debug` como alternativa (ya instalado) configurando manualmente un `launch.json`.

- **Errores extraños de IntelliSense en código válido.** La extensión usa `clangd` por defecto para autocompletado. Si ves errores falsos en código que compila bien, prueba desactivando la contribución de clangd de STM32Cube y usando `ms-vscode.cpptools` en su lugar.

- **Usar STM32CubeIDE y VS Code en la misma máquina.** Es seguro tener ambos instalados; no se interfieren si no abres el mismo proyecto en los dos al mismo tiempo. Cada uno gestiona sus propias herramientas de forma independiente (ya no comparten STM32CubeCLT).

---

## 13. Referencia de versiones

Versiones confirmadas al escribir esta guía (Septiembre 2026):

| Herramienta | Versión |
|---|---|
| STM32Cube for VS Code (paquete de extensiones) | v3.x |
| GNU Tools for STM32 (GCC, auto-descargado) | 13.3.1+st.9 |
| CMake (auto-descargado) | 4.0.1+st.3 |
| Ninja (auto-descargado) | 1.13.1+st.1 |
| STM32CubeMX (instalación separada, solo semana 7+) | 6.11.0+ |

**Recuerda:** este ecosistema cambia rápido. Antes de cada semestre, confirma que el flujo de "Create empty project" sigue funcionando igual y que las versiones auto-descargadas no han introducido cambios importantes.
