# Homework Exercises — Week 00
## Workshop V - Microcontrollers and Digital Electronics

This document contains the exercises for week 00. You should work through them in order, at your own pace, picking up from wherever the workshop session left off. All exercises run directly on the STM32F4xx microcontroller connected to your laptop through STM32CubeIDE. There is no `printf` or serial output — the debugger is your only window into what the microcontroller is doing, and learning to use it well is itself one of the goals of these exercises.

Before running any piece of code, always predict the result first. Write your prediction on paper or in a comment. Then run the code and compare what you see in the debugger against what you expected. If they match, great — you understood it. If they don't match, even better — you have something real to investigate and learn from. Never skip the prediction step.

Work with your classmates. Discuss your predictions, compare your results, and help each other when you get stuck. If you need guidance, use the AI teaching assistant — but always arrive with your own attempt first.

---

## Week 00 — Number Systems, Variable Sizes, and Representations

In this week's exercises, everything you do follows the same simple pattern: declare a variable, assign a value, set a breakpoint on the next line, and inspect the variable in the debugger. The microcontroller's memory is the laboratory, and the debugger is your instrument. Pay attention to the display format options in the debugger — you can view the same variable in decimal, hexadecimal, and binary, and switching between them will teach you a great deal.

### Exercise 0.1 — First contact with the debugger

Create a new project in STM32CubeIDE without using CubeMX code generation. The IDE will generate a skeleton project with a `main.c` file. The first thing you should always do is replace the auto-generated `for(;;)` loop with `while(1){}` — this is how we write an infinite loop in this course.

Inside `main()`, before the `while(1){}`, declare the following variable and set a breakpoint on the line immediately after it:

```c
uint8_t my_variable = 42;
```

Compile, flash, and run the project in debug mode. When execution stops at your breakpoint, find `my_variable` in the variable inspector and verify that it contains the value 42. This exercise is about one thing only: making sure your complete workflow — write, compile, flash, debug, inspect — is working correctly. If you see 42 in the variable inspector, you are ready to continue.

### Exercise 0.2 — Same number, different representations

Declare the following three variables:

```c
uint8_t dec = 65;
uint8_t hex = 0x41;
uint8_t bin = 0b01000001;
```

Before running the code, predict: do these three variables hold the same value or different values? Run the code, set a breakpoint after the declarations, and inspect all three in the debugger. Then try changing the display format of each variable between decimal, hexadecimal, and binary. Observe how the same underlying value looks different depending on the representation. Think about this: if the microcontroller only works with bits, what does it mean that you can write the same value in three different ways in your source code?

### Exercise 0.3 — How big is a variable?

Declare the following variables and predict what value each one will hold before running the code:

```c
uint8_t  a = 255;
uint16_t b = 255;
uint32_t c = 255;
uint8_t  d = 256;
```

Inspect all four in the debugger. One of them will surprise you. After verifying the result, add one more variable:

```c
uint8_t  e = 257;
```

What value does `e` hold? Why? Think about what it means that the variable type defines the "size of the container" and what happens when the value you want to store is larger than the container can hold.

### Exercise 0.4 — The sign matters

Declare the following variables:

```c
uint8_t a = 200;
int8_t  b = 200;
int8_t  c = -1;
```

Before running, predict the value of each variable in the debugger. Run the code and inspect the results. At least one will be different from what you might expect. After inspecting the values in decimal, switch the display format of variable `c` to hexadecimal. What do you see? Think about why -1 has that particular hexadecimal representation in an 8-bit signed variable. This is 2's complement in action — the same concept you studied in the theory class, now visible inside real hardware.

### Exercise 0.5 — Boundaries and overflow

Type the following code and step through it line by line in the debugger, inspecting the variable after each step:

```c
int8_t  x = 127;
x = x + 1;

uint8_t y = 255;
y = y + 1;
```

What happens to `x` after you add 1 to 127? What happens to `y` after you add 1 to 255? After observing both results, explore the boundary constants that `stdint.h` provides. Declare variables and assign them the values `INT8_MAX`, `INT8_MIN`, `UINT8_MAX`, and `UINT16_MAX`, then inspect each one in the debugger. Build a mental table of the minimum and maximum value for each type. This knowledge is not optional in embedded systems — choosing the wrong type for a variable is a real source of bugs in real products.

### Exercise 0.6 — Hexadecimal mental arithmetic

Declare `uint8_t result;` and write the following assignments one at a time. Before each one, write your prediction on paper. Then run and verify in the debugger:

```c
result = 0x0F + 0x01;
result = 0xFF + 0x01;
result = 0xA0 + 0x5F;
result = 0xA0 + 0x60;
```

For each result, ask yourself: did the addition overflow? How can you tell from the hexadecimal value alone? If your predictions were wrong, go back and work through the addition by hand in binary to understand where your reasoning failed. The debugger gives honest, immediate feedback — use it.

### Exercise 0.7 — Building a number bit by bit

Start with `uint8_t x = 0;` and write the following assignments one at a time, setting a breakpoint after each one and inspecting `x` in binary display format before moving to the next:

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

Look at the pattern that forms in the binary view. Each assignment activates exactly one bit, and each step doubles the previous value. Think about the connection between this pattern and the shift registers you simulated in week 00's theory — and think about what operation in C might produce this pattern automatically, without having to write eight separate assignments.

### Exercise 0.8 — The bang operator and boolean logic

Declare the following variables and write your prediction for each result before running the code:

```c
uint8_t a = 5;
uint8_t b = 0;
uint8_t c = 255;

uint8_t r1 = !a;
uint8_t r2 = !b;
uint8_t r3 = !c;
uint8_t r4 = ~c;
```

Inspect all four results in the debugger. Pay careful attention to `r3` and `r4` — they are computed from the same variable `c`, but using different operators. Are the results the same? Now add one more experiment:

```c
uint8_t r5 = ~a;
```

Compare `r1` (which is `!a`) with `r5` (which is `~a`). Are they the same? The difference between what you observe here is one of the most important distinctions in C programming for embedded systems. Make sure you can explain in your own words what `!` does versus what `~` does, and why they produce different results when applied to a value like 5.
