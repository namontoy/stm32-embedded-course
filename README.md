# stm32-embedded-course - Short description
AI-assisted STM32F4xx embedded systems course. Weekly configuration files for a Copilot CLI learning companion (Haiku 4.5 model - from Anthropic) that guides students through bare-metal ARM Cortex-M4 programming without replacing their thinking.

# STM32F4xx Embedded Systems Course — AI Configuration

This repository contains the AI companion configuration files for a 15-week university embedded systems course based on the STM32F4xx (ARM Cortex-M4) microcontroller.

## Philosophy

The AI is not here to write your code. It is here to help you think. Each week's configuration teaches the AI what you already know, what you are learning now, and how to guide you without doing the work for you.

This is not magic. It is engineering — and now you can see exactly how it works.

## How it works

Each week lives in its own branch (`week-01` through `week-15`).
The `AGENTS.md` file at the root is read automatically by Copilot CLI.
The `ai-config/` folder contains the full configuration — read it, understand it, learn from it.

## Getting started

1. Fork this repository
2. Clone your fork locally
3. Add upstream: `git remote add upstream <professor repo URL>`
4. Each week: `git fetch upstream week-XX && git checkout week-XX`

## Rules
- Do not modify `AGENTS.md` or `ai-config/`
- Save your work in `my-work/`
- Push only to your own fork
