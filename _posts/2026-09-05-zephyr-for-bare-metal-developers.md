---
layout: post
title: "Does Zephyr Scare the Bare-Metal Developer?"
author: "Sitara MPU Software Team"
date: 2026-09-05
categories: [Zephyr, Embedded Software]
tags: [Bare Metal, RTOS, Device Tree, Kconfig, Embedded]
description: "A practical comparison of bare-metal development and Zephyr, focused on the concepts that initially make Zephyr feel complex."
featured: false
---

# Does Zephyr Scare the Bare-Metal Developer?

Bare-metal developers value control.

You know which register you are touching, which interrupt is firing and which line of code configures the peripheral. The mental model is direct: application code talks to hardware.

Then you encounter Zephyr.

There is Device Tree. Kconfig. `west`. Drivers. APIs. Build layers. Suddenly, even a simple LED example can look much more complicated.

So the question is fair:

**Is Zephyr actually scary?**

A Sitara MPU presentation approached this question from the perspective of a bare-metal developer rather than assuming that an RTOS is automatically better.

## Bare metal starts simple

A typical bare-metal application can access hardware registers directly.

For a small application, this is attractive because there are few layers between the code and the hardware.

The problem appears when the product grows.

Imagine an ADC application that starts with a simple SPI configuration and a read function. Then the product needs to move to another MCU. SPI parameters change. The system needs a sleep mode. Sampling rates increase and DMA becomes necessary. Multiple tasks need access to the ADC.

The application gradually acquires:

- Configuration systems
- Device models
- Drivers
- Power management
- Synchronization
- Testing infrastructure

At some point, the team has recreated pieces of what an embedded operating system already provides.

## Zephyr is structure, not magic

Zephyr introduces abstractions that can initially feel like additional complexity.

But those abstractions exist because the product itself has become more complex.

Device Tree describes hardware configuration.

Kconfig selects software features.

Drivers provide standardized interfaces.

The kernel provides scheduling and synchronization primitives.

The important shift is from:

**"I directly control this peripheral."**

to:

**"I describe the hardware and use a standardized interface to access it."**

That is a different mental model—but it can become a useful one.

## The LED example

Consider a simple GPIO toggle.

In bare metal, an application may configure a GPIO directly and manipulate the relevant registers or a low-level GPIO library.

In Zephyr, the application can use the GPIO API and obtain its configuration through Device Tree.

That introduces more layers, but it also separates application intent from board-specific implementation.

The application says, effectively:

> I have an LED and I want to toggle it.

The board configuration describes where that LED is.

This becomes increasingly useful when the same application needs to run across different boards.

## Portability comes with a price

Abstraction is not free.

You need to understand Device Tree syntax, Kconfig options, driver APIs and the Zephyr build system.

There is also a learning curve around tools such as `west`.

For a tiny one-off application, bare metal may remain the simpler solution.

For a growing product family, however, the structure provided by Zephyr can prevent the application from becoming a collection of board-specific conditionals and duplicated infrastructure.

## The real question is not "Zephyr or bare metal?"

The more useful question is:

**Where does the complexity belong?**

If the product is simple and tightly coupled to one piece of hardware, direct control can be a reasonable choice.

As products gain multiple boards, peripherals, concurrency requirements, power-management needs and testing requirements, maintaining all that infrastructure yourself becomes increasingly expensive.

Zephyr moves some of that complexity into standardized frameworks.

That does not eliminate complexity.

It **organizes complexity**.

## Keep your mental model

The transition becomes easier when familiar concepts are mapped to their Zephyr equivalents.

- GPIO access → GPIO API
- Hardware configuration → Device Tree
- Feature selection → Kconfig
- Threads and synchronization → Zephyr kernel
- Board-specific configuration → board/device-tree definitions
- Build and workspace management → `west`

Once those relationships become familiar, the abstraction layers stop feeling mysterious.

## Choosing the right tool

There is no requirement to replace bare metal everywhere.

The right choice depends on product requirements, team expertise, hardware complexity, portability, power-management needs and long-term maintenance.

The useful takeaway is that Zephyr does not necessarily replace the developer's understanding of hardware.

Instead, it can provide a structure around that understanding.

---

**Related talk:** *Does Zephyr Scare the Bare-Metal Embedded Developer World?*

**Slides:** https://hosted-files.sched.co/ossindia2026/9c/BareMetal-Zephyr-OSS2026-Khasim.pdf

**Video:** https://www.youtube.com/watch?v=fNgZAINL28k
