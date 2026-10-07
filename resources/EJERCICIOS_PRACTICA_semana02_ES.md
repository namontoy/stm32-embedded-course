# Ejercicios de Tarea — Semana 02
## Taller V - Microcontroladores y Electrónica Digital

Este documento contiene los ejercicios para la semana 02. Debes trabajarlos en orden, a tu propio ritmo, continuando desde donde quedó la sesión de taller. Todos los ejercicios se ejecutan directamente en el microcontrolador STM32F4xx conectado a tu portátil a través de STM32CubeIDE. No hay `printf` ni salida serial — el depurador es tu única ventana a lo que está haciendo el microcontrolador, y aprender a usarlo bien es en sí mismo uno de los objetivos de estos ejercicios.

Antes de ejecutar cualquier fragmento de código, predice siempre el resultado primero. Escribe tu predicción en papel o en un comentario. Luego ejecuta el código y compara lo que ves en el depurador con lo que esperabas. Si coinciden, perfecto — lo entendiste. Si no coinciden, mejor aún — tienes algo real que investigar y aprender. Nunca omitas el paso de predicción.

Trabaja con tus compañeros. Discute tus predicciones, compara tus resultados y ayúdense mutuamente cuando se atoren. Si necesitas orientación, usa el asistente de enseñanza con IA — pero llega siempre con tu propio intento primero.

---

## Semana 02 — Operadores Bit a Bit e Introducción a las FSM

Los ejercicios de esta semana tienen dos pistas. La primera pista es práctica en el IDE — aplicar los operadores lógicos bit a bit a variables y luego a registros de hardware reales. La segunda pista es trabajo de diseño en papel — construir diagramas de estados FSM para sistemas del mundo real. Ambas pistas son igualmente importantes. Los ejercicios de código construyen las herramientas; los ejercicios de FSM construyen la disciplina para usarlas bien.

En esta semana, se proporcionan fragmentos de código para los ejercicios fundamentales, pero los ejercicios posteriores se vuelven más descriptivos. Lee la descripción cuidadosamente, entiende lo que se pide y escribe el código tú mismo.

### Ejercicio 2.1 — Operador AND: la máscara que revela

Escribe y ejecuta el siguiente código, prediciendo el resultado antes de ejecutar:

```c
uint8_t value  = 0b10110101;
uint8_t mask   = 0b00001111;
uint8_t result = value & mask;
```

Inspecciona `result` en formato de visualización binario. El operador AND conserva solo los bits donde la máscara tiene un 1 y fuerza todos los demás bits a 0 — actúa como una ventana que revela bits específicos mientras oculta otros. Después de verificar, responde esta pregunta en papel antes de continuar: ¿qué máscara usarías para extraer solo los 4 bits superiores (bits 7 a 4)? Escribe el código y verifica tu respuesta.

### Ejercicio 2.2 — Operador OR: la máscara que establece

Escribe y ejecuta el siguiente código, prediciendo el resultado:

```c
uint8_t value  = 0b10100000;
uint8_t mask   = 0b00000101;
uint8_t result = value | mask;
```

Inspecciona `result` en vista binaria. Responde estas dos preguntas antes de continuar: si un bit ya es 1 y le aplicas OR con 1, ¿qué le pasa a ese bit? Si un bit es 0 y le aplicas OR con 0, ¿qué ocurre? Tus respuestas explican por qué `|=` es la forma segura de establecer bits en un registro de hardware sin perturbar los bits que no tenías intención de cambiar.

### Ejercicio 2.3 — Operador NOT: el complemento

Escribe y ejecuta el siguiente código, prediciendo cada resultado:

```c
uint8_t a       = 0b00001111;
uint8_t result1 = ~a;

uint8_t b       = 0b10100101;
uint8_t result2 = ~b;
```

Después de verificar, construye el patrón de limpieza de bits paso a paso. Declara `uint8_t value = 0b11111111;` y luego limpia los bits 0 y 1 sin cambiar ningún otro bit. Haz esto usando el patrón `&= ~()`. Antes de escribir el código, trabaja la máscara y su complemento en papel primero, luego verifica tu resultado en el depurador.

### Ejercicio 2.4 — Operador XOR: el toggle

Escribe y ejecuta el siguiente código, prediciendo cada resultado:

```c
uint8_t value   = 0b10110011;
uint8_t mask    = 0b00001111;
uint8_t result1 = value ^ mask;
uint8_t result2 = result1 ^ mask;
```

¿Qué tiene de especial `result2`? XOR aplicado dos veces con la misma máscara siempre regresa al valor original — XOR es su propio inverso. Piensa en qué significa esta propiedad para un problema práctico: si quisieras cambiar un bit entre 0 y 1 repetidamente, ¿cómo usarías XOR para hacerlo con una sola operación?

### Ejercicio 2.5 — Combinando operadores: el patrón completo

Declara `uint8_t simulated_register = 0x00;` y luego realiza la siguiente secuencia de operaciones, colocando un breakpoint después de cada una y verificando el resultado en vista binaria antes de pasar a la siguiente:

- Establece los bits 3 y 4 simultáneamente usando una sola operación `|=`
- Limpia el bit 3 sin cambiar el bit 4 usando `&= ~()`
- Cambia el estado del bit 4 usando `^=`
- Cambia el estado del bit 4 nuevamente usando `^=`

Escribe cada operación tú mismo — no busques la sintaxis. Si no estás seguro, razona a partir de lo que sabes sobre cada operador. Después de completar la secuencia, lee el comentario en tu código: esta variable se comporta exactamente como un registro de hardware. La única diferencia en el siguiente ejercicio es que el registro está conectado al silicio real.

### Ejercicio 2.6 — Primer registro real: habilitando el reloj

Abre el manual de referencia del STM32F4xx y navega a la sección RCC (Reset and Clock Control). Encuentra el registro AHB1ENR e identifica cuál bit habilita el reloj para GPIOA. Luego escribe una sola línea de código que habilite ese reloj usando el nombre de constante definido por CMSIS — no uses un número directamente. Después de ejecutar la línea, abre el visualizador SFR en el depurador, navega a RCC → AHB1ENR y confirma que el bit correcto cambió de estado. Esta es la misma operación `|=` del ejercicio 2.5 — la única diferencia es que este registro controla hardware real.

### Ejercicio 2.7 — Primer LED en GPIOA Pin 5

Con el reloj de GPIOA ya habilitado del ejercicio 2.6, configura el pin 5 como salida digital y enciende el LED de la placa. Necesitarás escribir en dos registros: el registro MODER para establecer el modo del pin y el registro ODR para establecer el valor de salida. Las posiciones exactas de los bits están descritas en el manual de referencia — búscalas. Después de cada escritura en un registro, verifica en el visualizador SFR antes de revisar el LED físico en tu placa Nucleo.

Este es el momento donde todo lo que has aprendido se conecta: las operaciones bit a bit de esta semana, las representaciones numéricas de la semana 00 y la idea de que el hardware se controla mediante bits en registros. Si el LED se enciende, has entendido todo correctamente.

### Ejercicio 2.8 — Diseño de FSM: el semáforo peatonal

Este ejercicio no requiere código — solo pensamiento y papel. Diseña una Máquina de Estados Finitos para un sistema de semáforo peatonal. El sistema controla dos conjuntos de luces simultáneamente: uno para vehículos (rojo, amarillo, verde) y uno para peatones (caminar, no caminar). Para tu diseño, identifica todos los estados posibles en los que puede estar el sistema, dibuja un diagrama de estados completo con transiciones etiquetadas entre estados y describe en una oración qué desencadena cada transición y cuáles son las salidas (qué luces están encendidas) en cada estado.

Un estado está definido por las salidas que produce. Si dos momentos en la vida del sistema producen salidas diferentes, son estados diferentes. Antes de finalizar tu diagrama, verifícalo leyendo cada transición en voz alta: "el sistema está en el estado X, y cuando ocurre Y, pasa al estado Z." Si alguna transición no lleva a ningún lado, o si algún estado no tiene forma de salir, tu diseño tiene un vacío.

### Ejercicio 2.9 — Diseño de FSM: la máquina expendedora

Diseña una Máquina de Estados Finitos para una máquina expendedora simple. La máquina acepta monedas de una en una y dispensa un producto cuando se han insertado suficientes monedas. Tu diseño debe manejar no solo el escenario esperado — el usuario inserta el número correcto de monedas y recibe el producto — sino también los inesperados: ¿qué ocurre si el usuario inserta demasiadas monedas? ¿Y si presiona cancelar antes de insertar suficientes? ¿Y si el producto no está disponible?

Una máquina de estados que solo maneja el camino esperado es un diseño incompleto. En sistemas embebidos, el hardware puede recibir entradas inesperadas en cualquier momento, y el software debe responder de manera sensata en cada estado a cada posible entrada. Cada estado en tu diagrama debe tener una respuesta definida para cada posible entrada — incluso si esa respuesta es "no hacer nada y permanecer en el estado actual."
