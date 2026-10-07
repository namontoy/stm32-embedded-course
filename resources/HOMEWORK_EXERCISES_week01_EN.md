# Homework Exercises — Week 01
## Workshop V - Microcontrollers and Digital Electronics

This document contains the exercises for week 01. You should work through them in order, at your own pace, picking up from wherever the workshop session left off. All exercises run directly on the STM32F4xx microcontroller connected to your laptop through STM32CubeIDE. There is no `printf` or serial output — the debugger is your only window into what the microcontroller is doing, and learning to use it well is itself one of the goals of these exercises.

Before running any piece of code, always predict the result first. Write your prediction on paper or in a comment. Then run the code and compare what you see in the debugger against what you expected. If they match, great — you understood it. If they don't match, even better — you have something real to investigate and learn from. Never skip the prediction step.

Work with your classmates. Discuss your predictions, compare your results, and help each other when you get stuck. If you need guidance, use the AI teaching assistant — but always arrive with your own attempt first.

---

## Week 01 — C Language Fundamentals

All exercises this week continue to use the debugger as the only verification tool. The pattern is the same as week 00: predict, run, verify, understand. The focus shifts from number representations to arithmetic operations, shift operators, and control structures. Some exercises include code to type directly. Others describe what the code should do — you must write it yourself.

### Exercise 1.1 — Arithmetic operators and integer division

Declare the following variables and inspect each result in the debugger:

```c
uint8_t a          = 17;
uint8_t b          = 5;
uint8_t div_result = a / b;
uint8_t mod_result = a % b;
uint8_t mul_result = a * b;
```

Before running, predict each result. Pay particular attention to `div_result` — C performs integer division, which means the decimal part is simply discarded. After verifying, think about the relationship between `div_result` and `mod_result`: if you multiply `div_result` by `b` and add `mod_result`, what do you get? Verify your answer in the debugger.

### Exercise 1.2 — Overflow in arithmetic

Declare the following variables and predict each result before running:

```c
uint8_t x   = 200;
uint8_t y   = 100;
uint8_t sum = x + y;
```

The result will not be 300. Why not? After verifying in the debugger, think about this: if `sum` is wrong and your program uses it to make a decision — for example, to check if a sensor value exceeded a safety threshold — what could happen in a real system? This is not a hypothetical problem. Overflow-related bugs have caused real failures in real embedded systems.

### Exercise 1.3 — Shift operators as multiplication and division

Declare `uint8_t val = 3;` and then declare the following variables, predicting each result before running:

```c
uint8_t left1  = val << 1;
uint8_t left2  = val << 2;
uint8_t left3  = val << 3;
uint8_t right1 = val >> 1;
```

Inspect each result in binary display format to watch the bits move. Then test the edge case: what happens when you shift a bit off the end of the variable? Write a variable initialized to `0b10000000` and left-shift it by 1. What do you observe? Think about the connection between this behavior and the bit-by-bit exercise from week 00.

### Exercise 1.4 — Boolean evaluation and if/else

Write a program that declares three variables with different values, then uses `if` and `if-else` statements to set a `result` variable based on conditions. The logic should test at least one non-zero value as a condition, one zero value as a condition, and one equality comparison. Set a breakpoint after each `if` block and watch `result` change in the debugger as you step through.

Think carefully about how C decides whether a condition is true or false. What is the only value that C considers false? What does that tell you about how the language treats any non-zero number?

### Exercise 1.5 — The for loop as a counter

Write a `for` loop that counts from 0 to 9 and increments a variable called `counter` on each iteration. Set a breakpoint after the loop completes and verify that `counter` holds the value you expect. Then modify the loop in three different ways, one at a time: make it count in steps of 2, make it count backwards from 10 to 1, and make it stop at a different value. For each modification, predict the final value of `counter` before running.

### Exercise 1.6 — The while loop and a simple accumulator

Write a `while` loop that adds all integers from 1 to 10 and stores the running total in a variable called `sum`. Before running, calculate the expected result on paper. After verifying in the debugger, modify the exercise: change the data type of `sum` to `uint16_t` and sum all integers from 1 to 100. What is the expected result? Would it fit in a `uint8_t`? Verify your answer.

### Exercise 1.7 — The do-while loop

Write two separate pieces of code side by side. The first uses a `while` loop with a condition that is false from the start. The second uses a `do-while` loop with the same false condition. Both loops, if they execute, should set a result variable to 42. Predict which result variable will be 42 and which will remain at 0. Verify in the debugger. In your own words, describe the fundamental difference between `while` and `do-while`.

### Exercise 1.8 — The switch-case as a decision tree

Write a `switch-case` structure that maps an `input` variable to an `output` variable according to at least four different cases, with a `default` case for any value that does not match. Set `input` to different values, recompile, and verify in the debugger that `output` changes correctly each time. Then deliberately remove one of the `break` statements and observe what happens — this behavior is called "fall-through" and it is important to understand, even if you almost never want it to happen accidentally.
