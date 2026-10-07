# Ejercicios de Tarea — Semana 01
## Taller V - Microcontroladores y Electrónica Digital

Este documento contiene los ejercicios para la semana 01. Debes trabajarlos en orden, a tu propio ritmo, continuando desde donde quedó la sesión de taller. Todos los ejercicios se ejecutan directamente en el microcontrolador STM32F4xx conectado a tu portátil a través de STM32CubeIDE. No hay `printf` ni salida serial — el depurador es tu única ventana a lo que está haciendo el microcontrolador, y aprender a usarlo bien es en sí mismo uno de los objetivos de estos ejercicios.

Antes de ejecutar cualquier fragmento de código, predice siempre el resultado primero. Escribe tu predicción en papel o en un comentario. Luego ejecuta el código y compara lo que ves en el depurador con lo que esperabas. Si coinciden, perfecto — lo entendiste. Si no coinciden, mejor aún — tienes algo real que investigar y aprender. Nunca omitas el paso de predicción.

Trabaja con tus compañeros. Discute tus predicciones, compara tus resultados y ayúdense mutuamente cuando se atoren. Si necesitas orientación, usa el asistente de enseñanza con IA — pero llega siempre con tu propio intento primero.

---

## Semana 01 — Fundamentos del Lenguaje C

Todos los ejercicios de esta semana continúan usando el depurador como única herramienta de verificación. El patrón es el mismo que la semana 00: predecir, ejecutar, verificar, entender. El foco cambia de las representaciones numéricas a las operaciones aritméticas, los operadores de desplazamiento y las estructuras de control. Algunos ejercicios incluyen código para escribir directamente. Otros describen lo que debe hacer el código — tú debes escribirlo tú mismo.

### Ejercicio 1.1 — Operadores aritméticos y división entera

Declara las siguientes variables e inspecciona cada resultado en el depurador:

```c
uint8_t a          = 17;
uint8_t b          = 5;
uint8_t div_result = a / b;
uint8_t mod_result = a % b;
uint8_t mul_result = a * b;
```

Antes de ejecutar, predice cada resultado. Presta especial atención a `div_result` — C realiza división entera, lo que significa que la parte decimal simplemente se descarta. Después de verificar, piensa en la relación entre `div_result` y `mod_result`: si multiplicas `div_result` por `b` y le sumas `mod_result`, ¿qué obtienes? Verifica tu respuesta en el depurador.

### Ejercicio 1.2 — Desbordamiento en aritmética

Declara las siguientes variables y predice cada resultado antes de ejecutar:

```c
uint8_t x   = 200;
uint8_t y   = 100;
uint8_t sum = x + y;
```

El resultado no será 300. ¿Por qué no? Después de verificar en el depurador, piensa en esto: si `sum` es incorrecta y tu programa la usa para tomar una decisión — por ejemplo, para verificar si un valor de sensor superó un umbral de seguridad — ¿qué podría ocurrir en un sistema real? Este no es un problema hipotético. Los errores relacionados con desbordamiento han causado fallas reales en sistemas embebidos reales.

### Ejercicio 1.3 — Operadores de desplazamiento como multiplicación y división

Declara `uint8_t val = 3;` y luego declara las siguientes variables, prediciendo cada resultado antes de ejecutar:

```c
uint8_t left1  = val << 1;
uint8_t left2  = val << 2;
uint8_t left3  = val << 3;
uint8_t right1 = val >> 1;
```

Inspecciona cada resultado en formato de visualización binario para ver los bits moviéndose literalmente hacia la izquierda y la derecha. Luego prueba el caso límite: ¿qué ocurre cuando desplazas un bit fuera del extremo de la variable? Escribe una variable inicializada a `0b10000000` y desplázala a la izquierda 1 posición. ¿Qué observas?

### Ejercicio 1.4 — Evaluación booleana e if/else

Escribe un programa que declare tres variables con diferentes valores, luego use sentencias `if` e `if-else` para establecer una variable `result` basándose en condiciones. La lógica debe probar al menos un valor distinto de cero como condición, un valor cero como condición y una comparación de igualdad. Coloca un breakpoint después de cada bloque `if` y observa cómo cambia `result` en el depurador mientras recorres el código.

Piensa cuidadosamente en cómo C decide si una condición es verdadera o falsa. ¿Cuál es el único valor que C considera falso? ¿Qué te dice eso sobre cómo el lenguaje trata cualquier número distinto de cero?

### Ejercicio 1.5 — El bucle for como contador

Escribe un bucle `for` que cuente de 0 a 9 e incremente una variable llamada `counter` en cada iteración. Coloca un breakpoint después de que el bucle termine y verifica que `counter` tiene el valor que esperas. Luego modifica el bucle de tres maneras diferentes, una a la vez: haz que cuente en pasos de 2, haz que cuente hacia atrás de 10 a 1, y haz que se detenga en un valor diferente. Para cada modificación, predice el valor final de `counter` antes de ejecutar.

### Ejercicio 1.6 — El bucle while y un acumulador simple

Escribe un bucle `while` que sume todos los enteros del 1 al 10 y almacene el total acumulado en una variable llamada `sum`. Antes de ejecutar, calcula el resultado esperado en papel. Después de verificar en el depurador, modifica el ejercicio: cambia el tipo de dato de `sum` a `uint16_t` y suma todos los enteros del 1 al 100. ¿Cuál es el resultado esperado? ¿Cabría en un `uint8_t`? Verifica tu respuesta.

### Ejercicio 1.7 — El bucle do-while

Escribe dos piezas de código separadas una al lado de la otra. La primera usa un bucle `while` con una condición que es falsa desde el inicio. La segunda usa un bucle `do-while` con la misma condición falsa. Ambos bucles, si se ejecutan, deben establecer una variable resultado en 42. Predice cuál variable resultado será 42 y cuál permanecerá en 0. Verifica en el depurador. Con tus propias palabras, describe la diferencia fundamental entre `while` y `do-while`.

### Ejercicio 1.8 — El switch-case como árbol de decisiones

Escribe una estructura `switch-case` que mapee una variable `input` a una variable `output` según al menos cuatro casos diferentes, con un caso `default` para cualquier valor que no coincida. Establece `input` a diferentes valores, recompila y verifica en el depurador que `output` cambia correctamente cada vez. Luego elimina deliberadamente una de las sentencias `break` y observa qué ocurre — este comportamiento se llama "fall-through" y es importante entenderlo, aunque casi nunca quieras que ocurra accidentalmente.
