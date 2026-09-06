---
layout: post
title: "Optimizing Embedded Linux Power: From Measurement to Practical Tradeoffs"
author: "Sitara MPU Software Team"
date: 2026-09-05
categories: [Linux, Power Management, Performance]
tags: [Embedded Linux, AM62L, CPUFreq, Runtime PM, CPUIdle]
description: "A practical look at Linux power-management knobs used to reduce power in a display-enabled embedded application."
featured: false
---

# Optimizing Embedded Linux Power: From Measurement to Practical Tradeoffs

Power optimization on an embedded Linux system is rarely about finding one magic switch. It is usually a sequence of measurements, changes and tradeoffs.

A Sitara MPU case study looked at an **AM62L-based low-power HMI application** and explored system, CPU, display and DDR-related optimization opportunities while keeping the display functional.

The baseline system in the presented configuration consumed about **448 mW**. The interesting part was not simply the final number—it was understanding which changes produced meaningful savings and which did not.

## Start with the workload

Power optimization has to respect the application's requirements.

The case study was a Linux display application with a 60 Hz display, DDR at 1600 MT/s, two Cortex-A53 cores and several peripherals.

A low-power state that shuts down the display subsystem would not be useful if the application requires the display to remain active.

That is why power management is fundamentally a workload problem.

## Static versus dynamic optimization

The presentation groups Linux power techniques into two broad areas.

**Static optimizations** reduce power by configuring the system differently—for example, disabling unused devices or reducing clocks.

**Dynamic optimizations** react to runtime behavior, such as CPU idle states and frequency scaling.

Both matter, but they have different tradeoffs.

## Remove what the application does not need

Device Tree provides a straightforward way to keep unnecessary devices from being probed.

In the AM62L case study, nodes unrelated to the display subsystem and required UART functionality were removed or disabled. This produced approximately **18 mW of measured savings** in the presented setup.

The lesson is simple: every enabled block has a potential power cost, and a product configuration should not automatically carry hardware functionality that the application never uses.

## Clock frequency is another lever

Reducing clock frequencies can reduce power, particularly for inactive or lightly loaded peripherals.

The case study reduced frequencies for clocks in the WKUP power domain and measured approximately **56 mW of savings**.

Linux can expose clock information through the kernel clock framework, while platform-specific tools can help with experimentation and diagnosis.

Clock availability and dependencies are platform-specific, so changes should be validated against the actual SoC architecture and firmware configuration.

## CPU idle: lower power is not always better

Linux CPUIdle allows inactive CPUs to enter idle states.

Deeper states generally offer greater power savings, but entering and leaving those states has latency and residency implications.

If a CPU repeatedly enters and exits an idle state for very short periods, the transition overhead can reduce the benefit.

For that reason, CPUIdle tuning is about finding the balance between idle-state depth, expected idle duration, entry/exit latency and application responsiveness.

## DVFS and CPUFreq

Dynamic voltage and frequency scaling provides another runtime optimization mechanism.

Linux CPUFreq can select operating performance points according to workload and policy. Common governors include `powersave`, `ondemand`, `schedutil`, `performance` and `userspace`.

In the AM62L example, voltage scaling was not available in the presented configuration, but CPU frequency was reduced to 200 MHz using the userspace governor. The measured saving was approximately **8 mW**.

This is a useful reminder that the theoretical power-management toolbox and the practical platform toolbox are not always identical.

## What about CPU hotplug?

Turning off a CPU sounds attractive, but the measured result in this case was different.

The study turned off one of the two A53 cores and observed **minimal power savings**.

This illustrates an important engineering principle:

> A feature that sounds power-efficient does not necessarily produce meaningful system-level savings.

The reliable answer comes from measurement.

## Power optimization is a measurement discipline

A repeatable approach is:

1. Establish a representative workload.
2. Measure the baseline.
3. Identify active hardware and software components.
4. Apply one optimization at a time.
5. Measure the resulting change.
6. Validate performance and latency.
7. Keep only changes that improve the product-level objective.

Power optimization is therefore not a checklist. It is an iterative engineering process.

---

**Related talk:** *Optimizing Power Consumption in Embedded Linux: Techniques and Tradeoffs*

**Slides:** https://hosted-files.sched.co/osselcna2026/48/EOSS-power-optimizations-final.pdf

**Video:** https://www.youtube.com/watch?v=tlJ85uYjqXk&t
