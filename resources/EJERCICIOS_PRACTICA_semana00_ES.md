# Ejercicios de Tarea — Semana 00
## Taller V - Microcontroladores y Electrónica Digital

Este documento contiene los ejercicios para la semana 00. Debes trabajarlos en orden, a tu propio ritmo, continuando desde donde quedó la sesión de taller. Todos los ejercicios se ejecutan directamente en el microcontrolador STM32F4xx conectado a tu portátil a través de STM32CubeIDE. No hay `printf` ni salida serial — el depurador es tu única ventana a lo que está haciendo el microcontrolador, y aprender a usarlo bien es en sí mismo uno de los objetivos de estos ejercicios.

Antes de ejecutar cualquier fragmento de código, predice siempre el resultado primero. Escribe tu predicción en papel o en un comentario. Luego ejecuta el código y compara lo que ves en el depurador con lo que esperabas. Si coinciden, perfecto — lo entendiste. Si no coinciden, mejor aún — tienes algo real que investigar y aprender. Nunca omitas el paso de predicción.

Trabaja con tus compañeros. Discute tus predicciones, compara tus resultados y ayúdense mutuamente cuando se atoren. Si necesitas orientación, usa el asistente de enseñanza con IA — pero llega siempre con tu propio intento primero.

---

## Semana 00 — Sistemas Numéricos, Tamaños de Variables y Representaciones

En los ejercicios de esta semana, todo lo que haces sigue el mismo patrón simple: declarar una variable, asignar un valor, poner un breakpoint en la siguiente línea e inspeccionar la variable en el depurador. La memoria del microcontrolador es el laboratorio, y el depurador es tu instrumento. Presta atención a las opciones de formato de visualización en el depurador — puedes ver la misma variable en decimal, hexadecimal y binario, y cambiar entre ellos te enseñará mucho.

### Ejercicio 0.1 — Primer contacto con el depurador

Crea un nuevo proyecto en STM32CubeIDE sin usar la generación de código de CubeMX. El IDE generará un proyecto esqueleto con un archivo `main.c`. Lo primero que siempre debes hacer es reemplazar el bucle `for(;;)` generado automáticamente por `while(1){}` — así es como escribimos un bucle infinito en este curso.

Dentro de `main()`, antes del `while(1){}`, declara la siguiente variable y coloca un breakpoint en la línea inmediatamente después:

```c
uint8_t my_variable = 42;
```

Compila, flashea y ejecuta el proyecto en modo de depuración. Cuando la ejecución se detenga en tu breakpoint, encuentra `my_variable` en el inspector de variables y verifica que contiene el valor 42. Este ejercicio trata de una sola cosa: asegurarte de que tu flujo de trabajo completo — escribir, compilar, flashear, depurar, inspeccionar — funciona correctamente. Si ves 42 en el inspector de variables, estás listo para continuar.

### Ejercicio 0.2 — El mismo número, diferentes representaciones

Declara las siguientes tres variables:

```c
uint8_t dec = 65;
uint8_t hex = 0x41;
uint8_t bin = 0b01000001;
```

Antes de ejecutar el código, predice: ¿estas tres variables contienen el mismo valor o valores diferentes? Ejecuta el código, coloca un breakpoint después de las declaraciones e inspecciona las tres en el depurador. Luego intenta cambiar el formato de visualización de cada variable entre decimal, hexadecimal y binario. Observa cómo el mismo valor subyacente se ve diferente según la representación. Piensa en esto: si el microcontrolador solo trabaja con bits, ¿qué significa que puedas escribir el mismo valor de tres maneras diferentes en tu código fuente?

### Ejercicio 0.3 — ¿Qué tan grande es una variable?

Declara las siguientes variables y predice qué valor tendrá cada una antes de ejecutar el código:

```c
uint8_t  a = 255;
uint16_t b = 255;
uint32_t c = 255;
uint8_t  d = 256;
```

Inspecciona las cuatro en el depurador. Una de ellas te sorprenderá. Después de verificar el resultado, agrega una variable más:

```c
uint8_t  e = 257;
```

¿Qué valor tiene `e`? ¿Por qué? Piensa en qué significa que el tipo de variable define el "tamaño del contenedor" y qué ocurre cuando el valor que quieres almacenar es más grande de lo que el contenedor puede contener.

### Ejercicio 0.4 — El signo importa

Declara las siguientes variables:

```c
uint8_t a = 200;
int8_t  b = 200;
int8_t  c = -1;
```

Antes de ejecutar, predice el valor de cada variable en el depurador. Ejecuta el código e inspecciona los resultados. Al menos uno será diferente de lo que podrías esperar. Después de inspeccionar los valores en decimal, cambia el formato de visualización de la variable `c` a hexadecimal. ¿Qué ves? Piensa por qué -1 tiene esa representación hexadecimal particular en una variable de 8 bits con signo. Esto es el complemento a 2 en acción — el mismo concepto que estudiaste en la clase de teoría, ahora visible dentro del hardware real.

### Ejercicio 0.5 — Límites y desbordamiento

Escribe el siguiente código y recórrelo línea por línea en el depurador, inspeccionando la variable después de cada paso:

```c
int8_t  x = 127;
x = x + 1;

uint8_t y = 255;
y = y + 1;
```

¿Qué le pasa a `x` después de sumarle 1 a 127? ¿Qué le pasa a `y` después de sumarle 1 a 255? Después de observar ambos resultados, explora las constantes de límite que proporciona `stdint.h`. Declara variables y asígnales los valores `INT8_MAX`, `INT8_MIN`, `UINT8_MAX` y `UINT16_MAX`, luego inspecciona cada una en el depurador. Construye una tabla mental de los valores mínimo y máximo para cada tipo. Este conocimiento no es opcional en sistemas embebidos — elegir el tipo incorrecto para una variable es una fuente real de errores en productos reales.

### Ejercicio 0.6 — Aritmética mental en hexadecimal

Declara `uint8_t result;` y escribe las siguientes asignaciones una a la vez. Antes de cada una, escribe tu predicción en papel. Luego ejecuta y verifica en el depurador:

```c
result = 0x0F + 0x01;
result = 0xFF + 0x01;
result = 0xA0 + 0x5F;
result = 0xA0 + 0x60;
```

Para cada resultado, pregúntate: ¿la suma se desbordó? ¿Cómo puedes saberlo solo con el valor hexadecimal? Si tus predicciones estuvieron equivocadas, regresa y trabaja la suma a mano en binario para entender dónde falló tu razonamiento. El depurador da retroalimentación honesta e inmediata — úsalo.

### Ejercicio 0.7 — Construyendo un número bit a bit

Comienza con `uint8_t x = 0;` y escribe las siguientes asignaciones una a la vez, colocando un breakpoint después de cada una e inspeccionando `x` en formato de visualización binario antes de pasar a la siguiente:

```c
x = 0x01;
x = 0x02;
x = 0x04;
x = 0x08;
x = 0x10;
x = 0x20;
x = 0x40;
x = 0x80;
```

El patrón que emerge — cada paso duplica el valor anterior — conecta directamente con la experiencia de simulación de registros de desplazamiento de la semana 00 de teoría, y anticipa el operador de desplazamiento que llega en la semana 01. El objetivo es internalizar la relación entre los dígitos hexadecimales y las posiciones individuales de los bits.

### Ejercicio 0.8 — El operador bang y la lógica booleana

Declara las siguientes variables y escribe tu predicción para cada resultado antes de ejecutar el código:

```c
uint8_t a = 5;
uint8_t b = 0;
uint8_t c = 255;

uint8_t r1 = !a;
uint8_t r2 = !b;
uint8_t r3 = !c;
uint8_t r4 = ~c;
```

Inspecciona los cuatro resultados en el depurador. Presta especial atención a `r3` y `r4` — se calculan a partir de la misma variable `c`, pero usando operadores diferentes. ¿Son iguales los resultados? Ahora agrega un experimento más:

```c
uint8_t r5 = ~a;
```

Compara `r1` (que es `!a`) con `r5` (que es `~a`). ¿Son iguales? La diferencia entre lo que observas aquí es una de las distinciones más importantes en la programación C para sistemas embebidos. Asegúrate de poder explicar con tus propias palabras qué hace `!` versus qué hace `~`, y por qué producen resultados diferentes cuando se aplican a un valor como 5.
