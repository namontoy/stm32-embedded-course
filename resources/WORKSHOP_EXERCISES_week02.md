# Workshop Exercises — Weeks 00 to 03
## STM32F4xx Embedded Systems Course

This document contains guided workshop exercises for weeks 00 through 03. Each exercise is designed to be led by the professor with 20 students working simultaneously on their own laptops connected to a real STM32F4xx microcontroller. All results are verified through the STM32CubeIDE debugger — variable inspector for weeks 00 through 02, and the SFR (Special Function Registers) viewer for week 03. There is no `printf` or serial output at any point.

Exercises within each week follow an incremental difficulty path. The professor should guide the group step by step, allowing students to predict the result before running the code and then verify their prediction in the debugger. This prediction-then-verification habit is intentional and should be reinforced throughout the course.

---

## Week 00 — Number Systems, Variable Sizes, and Representations

The core idea for this week is simple and powerful: declare variables, assign values, and observe what the microcontroller actually stores. Every exercise teaches something about how numbers live inside memory, and the debugger is the window into that world. Students should always predict the result before running the code.

### Exercise 0.1 — First contact with the debugger

The student creates a new project in STM32CubeIDE without CubeMX code generation, replaces the auto-generated `for(;;)` with `while(1){}`, declares a `uint8_t` variable, assigns it the value 42, sets a breakpoint on the next line, and inspects the variable in the debugger. The goal is simply to verify the complete workflow: write, compile, flash, debug, and inspect. The victory condition is seeing the number 42 appear in the variable inspector. This exercise is about building confidence with the environment, not about learning anything complex.

### Exercise 0.2 — Same number, different representations

The student declares three variables and assigns the same value written in three different ways:

```c
uint8_t dec = 65;
uint8_t hex = 0x41;
uint8_t bin = 0b01000001;
```

In the debugger they inspect all three and discover they hold identical values. Then they change the debugger's display format between decimal, hexadecimal, and binary to see how the same number looks in each representation. This builds the foundational intuition that decimal, hex, and binary are simply different ways of writing the same underlying value — the microcontroller only knows bits.

### Exercise 0.3 — How big is a variable?

The student declares four variables of different sizes:

```c
uint8_t  a = 255;
uint16_t b = 255;
uint32_t c = 255;
uint8_t  d = 256;
```

The first three hold 255 as expected, but `d` shows 0 in the debugger — it overflowed because 256 does not fit in 8 bits. Then the student tries `uint8_t e = 257;` and observes the value 1. The key lesson is that the data type defines the container, and values that exceed the container's capacity wrap around. This is not a bug — it is predictable, mathematical behavior that embedded developers must understand and anticipate.

### Exercise 0.4 — The sign matters

The student declares:

```c
uint8_t a = 200;
int8_t  b = 200;
int8_t  c = -1;
```

Variable `a` shows 200, but `b` shows -56 — because 200 interpreted as a 2's complement 8-bit signed value wraps into negative territory. Then they switch the debugger display for `c` to hexadecimal and see `0xFF`. This is the moment where 2's complement stops being a paper exercise and becomes something the student can observe directly inside the microcontroller. The question to ask the group: "why does -1 appear as 0xFF in an 8-bit signed variable?"

### Exercise 0.5 — Boundaries and overflow

The student explores the natural boundaries of each type by stepping through the following sequence:

```c
int8_t  x = 127;
x = x + 1;   /* what happens? */

uint8_t y = 255;
y = y + 1;   /* what happens? */
```

They step through each line and observe: `x` becomes -128 (signed overflow wraps to the most negative value), and `y` becomes 0 (unsigned overflow wraps to zero). Then they explore the boundary constants defined in `stdint.h` — `INT8_MAX`, `INT8_MIN`, `UINT8_MAX`, `UINT16_MAX` — assigning them to variables and inspecting their values. The goal is to build a mental table of limits for each type and understand overflow as a predictable, mathematical consequence of fixed-width storage.

### Exercise 0.6 — Hexadecimal mental arithmetic

The student declares `uint8_t result;` and performs a series of additions in hexadecimal, predicting the result on paper before each one:

```c
result = 0x0F + 0x01;   /* predict: 0x10 */
result = 0xFF + 0x01;   /* predict: 0x00 — overflow */
result = 0xA0 + 0x5F;   /* predict: 0xFF */
result = 0xA0 + 0x60;   /* predict: 0x00 — overflow */
```

After each assignment the student inspects the result in the debugger and compares it to their prediction. This exercise builds hexadecimal arithmetic fluency through the prediction-and-verify cycle. Students who struggled with hex arithmetic on paper will find that the debugger provides immediate, honest feedback.

### Exercise 0.7 — Building a number bit by bit

The student starts with `uint8_t x = 0;` and assigns specific hexadecimal values to activate one bit at a time, inspecting the binary representation in the debugger after each step:

```c
x = 0x01;   /* bit 0 */
x = 0x02;   /* bit 1 */
x = 0x04;   /* bit 2 */
x = 0x08;   /* bit 3 */
x = 0x10;   /* bit 4 */
x = 0x20;   /* bit 5 */
x = 0x40;   /* bit 6 */
x = 0x80;   /* bit 7 */
```

The pattern that emerges — each step doubles the previous value — connects directly to the week 0 simulation experience with shift registers, and previews the shift operator that arrives in week 1. The goal is to internalize the relationship between hex digits and individual bit positions.

### Exercise 0.8 — The bang operator and boolean logic

The student declares:

```c
uint8_t a = 5;
uint8_t b = 0;
uint8_t c = 255;

uint8_t r1 = !a;    /* what is the result? */
uint8_t r2 = !b;    /* what is the result? */
uint8_t r3 = !c;    /* what is the result? */
uint8_t r4 = ~c;    /* what is the result? */
```

Before running, students predict each value. In the debugger they observe: `r1 = 0` (5 is non-zero, so logically true, and `!true = 0`), `r2 = 1` (0 is false, and `!false = 1`), `r3 = 0` (255 is non-zero, so logically true, and `!true = 0`). Then `r4 = ~c` gives 0 — but for a completely different reason: `~` is the bitwise NOT that flips every individual bit, so all eight 1-bits in 255 become 0-bits. The critical question to ask the group: "both `!c` and `~c` give 0 when c is 255 — but try `~a` where a is 5. Are `!a` and `~a` still the same?" They will find that `!a = 0` and `~a = 250` — completely different results. This side-by-side comparison between logical NOT (`!`) and bitwise NOT (`~`) is foundational for everything that comes in weeks 2 and 3.

---

## Week 01 — C Language Fundamentals

All exercises this week are written and run in STM32CubeIDE connected to the MCU. The debugger variable inspector remains the only feedback tool — no `printf`, no serial output. Students should continue the habit of predicting results before running code, then verifying with the debugger. The focus is on arithmetic, shift operators, control structures, and understanding how C evaluates boolean expressions.

### Exercise 1.1 — Arithmetic operators and integer division

The student declares several variables and performs basic arithmetic, with special attention to division and modulus:

```c
uint8_t a = 17;
uint8_t b = 5;
uint8_t div_result = a / b;    /* predict: 3, not 3.4 */
uint8_t mod_result = a % b;    /* predict: 2 */
uint8_t mul_result = a * b;    /* predict: 85 */
```

The key lesson is integer division — the decimal part simply disappears. `17 / 5` gives `3`, not `3.4`, because there are no floating point types involved. The `%` operator then gives the remainder `2`. The question to ask the group: "how would you verify that `div_result * b + mod_result` equals `a`?" Let them try it in the debugger and see the relationship between `/` and `%` directly.

### Exercise 1.2 — Overflow in arithmetic

The student explores what happens when arithmetic produces a result that does not fit in the variable's type:

```c
uint8_t x = 200;
uint8_t y = 100;
uint8_t sum     = x + y;      /* predict: 44, not 300 — overflow */
uint8_t product = 20 * 15;    /* predict: 44 — same overflow, same result */
```

The sum 200 + 100 = 300 overflows a `uint8_t`, wrapping to 44 (because 300 - 256 = 44). The product 20 * 15 = 300 has the same problem and lands on the same value — a striking coincidence that reinforces the overflow concept. This is an important embedded systems lesson: arithmetic overflow is silent and produces wrong results without any warning. The question to ask: "in a real system, what could go wrong if you added two sensor readings that together exceeded 255 but you stored the result in a `uint8_t`?"

### Exercise 1.3 — Shift operators as multiplication and division

The student experiments with left and right shifts, connecting to their week 0 experience with shift registers in the Digital simulator:

```c
uint8_t val    = 3;
uint8_t left1  = val << 1;    /* predict: 6  — multiply by 2 */
uint8_t left2  = val << 2;    /* predict: 12 — multiply by 4 */
uint8_t left3  = val << 3;    /* predict: 24 — multiply by 8 */
uint8_t right1 = val >> 1;    /* predict: 1  — divide by 2, truncated */
```

The student inspects each result in binary view in the debugger to see the bits literally moving left and right — exactly like the shift register they simulated in week 0. Then comes the important edge case:

```c
uint8_t edge    = 0b10000000;    /* 128 */
uint8_t shifted = edge << 1;     /* predict: 0 — the bit falls off! */
```

This visually demonstrates why bit width matters and previews the concept of bit masking that arrives in week 2.

### Exercise 1.4 — Boolean evaluation and the if/else structure

The student writes their first decision-making code and verifies the outcome through a dedicated `result` variable, changing it only when a condition is true:

```c
uint8_t condition_a = 5;
uint8_t condition_b = 0;
uint8_t condition_c = 255;
uint8_t result = 0;

if(condition_a)
{
    result = 1;    /* non-zero is true */
}

if(!condition_b)
{
    result = 2;    /* !0 is true */
}

if(condition_c == 255)
{
    result = 3;    /* equality check */
}
```

The student sets a breakpoint after each `if` block and watches `result` change in the variable inspector. The key insight is that C has no dedicated boolean type — any non-zero value is treated as true. They should also notice that `condition_c` being 255 is just as "truthy" as 5 — only 0 is false.

### Exercise 1.5 — The for loop as a counter

The student writes a loop that counts iterations and stores the final value, setting a breakpoint after the loop to inspect the result:

```c
uint8_t counter = 0;
uint8_t i;

for(i = 0; i < 10; i++)
{
    counter = counter + 1;
}
/* breakpoint here: counter should be 10 */
```

After verifying the basic result, the student modifies the loop to count in steps of 2, count backwards from 10 to 0, and stop at different values — building intuition for how the three parts of the `for` loop (initialization, condition, increment) work together. A good challenge question: "how would you modify the loop so that `counter` ends up at exactly 100 using only 10 iterations?"

### Exercise 1.6 — The while loop and a simple accumulator

The student implements an accumulator that sums all integers from 1 to 10, predicting the result on paper before running the code:

```c
uint8_t sum = 0;
uint8_t n   = 1;

while(n <= 10)
{
    sum = sum + n;
    n++;
}
/* breakpoint here: sum should be 55 */
```

After verifying that `sum` equals 55, a natural follow-up: "what happens if you change `uint8_t sum` to `uint16_t sum` and sum all numbers from 1 to 100? What is the expected result? Does it still fit in a `uint8_t`?" The sum of 1 to 100 is 5050, which overflows `uint8_t` silently but fits safely in `uint16_t` — another reinforcement of choosing the right data type.

### Exercise 1.7 — The do-while loop and guaranteed first execution

The student compares `while` and `do-while` by observing their behavior when the condition is false from the very start:

```c
uint8_t result_while   = 0;
uint8_t result_dowhile = 0;

/* this loop never executes — condition is false immediately */
while(0)
{
    result_while = 42;
}

/* this loop executes once even though condition is false */
do
{
    result_dowhile = 42;
} while(0);

/* breakpoint here: result_while = 0, result_dowhile = 42 */
```

This side-by-side comparison makes the structural difference visceral and memorable — the student sees directly in the debugger that `do-while` always executes at least once, regardless of the condition. The question to ask: "can you think of a real situation in embedded systems where you always need to perform an action at least once before deciding whether to repeat it?"

### Exercise 1.8 — The switch-case as an elegant decision tree

The student writes a mapping from a numeric input to a named output value — a deliberate preview of the FSM thinking that arrives in week 2:

```c
uint8_t input  = 2;
uint8_t output = 0;

switch(input)
{
    case 1:
        output = 10;
        break;
    case 2:
        output = 20;
        break;
    case 3:
        output = 30;
        break;
    default:
        output = 99;
        break;
}
/* breakpoint here: output should be 20 */
```

The student changes `input` to different values, re-runs, and watches `output` respond accordingly in the debugger. Then they deliberately remove one `break` statement to observe "fall-through" behavior — an important C quirk. The closing question ties it to the bigger picture: "how would you write this same logic using only `if/else` statements? Which version is easier to read when you have 10 cases instead of 3?" This question plants the seed for `switch-case` as the natural backbone of an FSM implementation, which they will build in weeks 2 and 4.

---

## Week 02 — Bitwise Logic Operators and FSM Introduction

This week has two parallel tracks. The first track is hands-on and debugger-verified: the student learns the bitwise logic operators first on regular variables, then applies them to real MCU registers for the first time. The second track is conceptual and paper-based: the student designs FSM state diagrams for real-world systems without writing any code. Both tracks should be treated with equal seriousness — the bitwise skills are the tools, and the FSM thinking is the design discipline that will guide how those tools are used.

### Exercise 2.1 — AND operator: the mask that reveals

The student uses the AND operator to extract specific bits from a value, discovering that AND acts as a "window" that reveals only the bits where the mask has a 1:

```c
uint8_t value  = 0b10110101;    /* 181 */
uint8_t mask   = 0b00001111;    /* 0x0F — lower nibble mask */
uint8_t result = value & mask;  /* predict: 0b00000101 = 5 */
```

After inspecting `result` in binary view, the question to ask the group is: "what mask would you use to extract only the upper 4 bits — the upper nibble?" Let the students derive `0xF0` themselves rather than providing it. This small act of derivation builds confidence and reinforces that masks are not magic values to memorize but logical constructions anyone can produce.

### Exercise 2.2 — OR operator: the mask that sets

The student uses OR to set specific bits without disturbing the others, building the intuition for why `|=` is the safe way to turn bits on in a register:

```c
uint8_t value  = 0b10100000;
uint8_t mask   = 0b00000101;
uint8_t result = value | mask;  /* predict: 0b10100101 */
```

The key insight is that OR can only turn bits ON, never OFF. The questions to ask: "if a bit is already 1 and you OR it with 1, what happens? And if you OR it with 0?" These two questions together explain why `|=` is safe — it can never accidentally clear a bit that was already set.

### Exercise 2.3 — NOT operator: the complement

The student explores bitwise NOT on different values and then combines it with AND to build the bit-clearing pattern they will use in every future register operation:

```c
uint8_t a       = 0b00001111;
uint8_t result1 = ~a;             /* predict: 0b11110000 = 0xF0 */

uint8_t b       = 0b10100101;
uint8_t result2 = ~b;             /* predict: 0b01011010 */

/* Now combine NOT with AND to clear specific bits */
uint8_t value   = 0b11111111;
uint8_t cleared = value & ~(0b00000011);  /* clear bits 0 and 1 */
/* predict: 0b11111100 */
```

The student should step through the last expression slowly — first computing `~(0b00000011)` mentally to get `0b11111100`, then AND-ing that with `0xFF` to get `0b11111100`. Seeing this built step by step makes the `&= ~(mask)` pattern feel logical rather than like a formula to memorize.

### Exercise 2.4 — XOR operator: the toggle

The student discovers XOR's most useful property — it toggles bits, and applying the same mask twice returns to the original value:

```c
uint8_t value   = 0b10110011;
uint8_t mask    = 0b00001111;
uint8_t result1 = value ^ mask;    /* predict: 0b10111100 */
uint8_t result2 = result1 ^ mask;  /* predict: back to 0b10110011 */
```

The revelation that XOR is its own inverse — that applying it twice cancels itself out — is genuinely surprising to most students. The question to plant for week 4: "if you wanted to blink an LED without using an if/else statement, how might XOR help you?" Don't answer it yet. Let the idea sit until they encounter LEDs in week 4.

### Exercise 2.5 — Combining operators: the full pattern

The student now practices all three fundamental register operations — set, clear, and toggle — on a single variable that simulates a hardware register. This exercise is the culmination of the bitwise track and the bridge to real hardware:

```c
uint8_t simulated_register = 0x00;

/* Set bits 3 and 4 */
simulated_register |= (1 << 3) | (1 << 4);
/* verify: 0b00011000 */

/* Clear bit 3 without touching bit 4 */
simulated_register &= ~(1 << 3);
/* verify: 0b00010000 */

/* Toggle bit 4 */
simulated_register ^= (1 << 4);
/* verify: 0b00000000 */

/* Toggle bit 4 again */
simulated_register ^= (1 << 4);
/* verify: 0b00010000 — back to what it was */
```

After each operation the student inspects in binary view. The message to reinforce: "this variable behaves exactly like a hardware register. The only difference next week is that the register is connected to real silicon."

### Exercise 2.6 — First real register: enabling the clock

The student opens the STM32F4xx reference manual, navigates to the RCC section, finds the AHB1ENR register description, reads which bit enables the GPIOA clock, and then writes the operation using the CMSIS-defined name:

```c
/* Enable GPIOA clock — bit found in RCC section of the reference manual */
RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
```

After executing this line, they open the SFR viewer, navigate to RCC → AHB1ENR, and verify that the correct bit changed. This is a landmark moment — the first time the student's C code changes a real control register inside the microcontroller. The key message to deliver: "this is exactly the same `|=` operation from exercise 2.5. The only difference is that this register is wired to real hardware."

### Exercise 2.7 — First LED on GPIOA Pin 5

Building directly on exercise 2.6, the student sets pin 5 of GPIOA as an output and turns the onboard LED on. The GPIO configuration details are not yet fully explained — this is still "black box" territory — but the bitwise operations are entirely theirs:

```c
/* Clock already enabled from exercise 2.6 */

/* Set pin 5 as output mode — MODER register, bits 11:10 set to 01 */
GPIOA->MODER |= (1 << 10);
GPIOA->MODER &= ~(1 << 11);

/* Turn the LED on — ODR register, bit 5 */
GPIOA->ODR |= (1 << 5);
```

After each register write, the student checks the SFR viewer and then looks at the physical LED on the Nucleo board. The emotional impact of this moment — seeing a real LED respond to a line of C code they wrote — is significant and should be acknowledged. This is the payoff for two weeks of careful, patient foundation-building.

### Exercise 2.8 — FSM design: the pedestrian traffic light

Switching to the conceptual track, the student designs a state machine for a pedestrian traffic light system entirely on paper — no code, only thinking. The system controls two lights simultaneously: a vehicle light (red, yellow, green) and a pedestrian light (walk, don't walk). The student must identify all states, draw the complete state diagram with labeled transitions, and write a brief description of what triggers each transition and what the outputs are in each state.

A common mistake to watch for is students who define too many states — treating "green for 10 seconds" and "green for 5 seconds" as different states — or too few, collapsing states that have genuinely different output behaviors. A useful guiding question is: "what makes this moment in the system's life different from that moment? If the outputs are different, it might be a different state." The goal is to develop the discipline of thinking in states and transitions before ever writing a line of code.

### Exercise 2.9 — FSM design: the vending machine

A slightly more complex FSM than the traffic light, the vending machine introduces the important idea that state machines must handle unexpected or unwanted inputs gracefully — not just the "happy path." The student designs a machine with monetary states (0 coins inserted, 1 coin inserted, 2 coins inserted), a dispense state, and a change-return state. The challenge is to handle edge cases: what happens if the user inserts more coins than needed? What if they press cancel halfway through? What if the product is out of stock?

These edge cases are not hypothetical annoyances — they represent the reality of embedded systems, where hardware can receive unexpected signals at any time and the software must respond sensibly to every possible input in every possible state. A student who designs an FSM that handles only the happy path has designed half a system. This exercise plants that lesson early, long before it becomes a debugging crisis.

---

## Week 03 — MCU Architecture and Bare-Metal Programming

All exercises this week involve reading the STM32F4xx reference manual to find register descriptions, performing register operations, and verifying results using the SFR (Special Function Registers) viewer in the STM32CubeIDE debugger. Students should always open the reference manual alongside the IDE.

*(Exercises to be added in the next session)*
