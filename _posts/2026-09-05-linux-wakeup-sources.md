---
layout: post
title: "Understanding Linux Wakeup Sources: The Architecture Behind Reliable Resume"
author: "Sitara MPU Software Team"
date: 2026-09-05
categories: [Linux, Power Management]
tags: [Wakeup Sources, Suspend Resume, Device Tree, Power Domains]
description: "A practical introduction to Linux wakeup sources, suspend/resume flow, device-tree configuration and power-domain interactions."
featured: false
---

# Understanding Linux Wakeup Sources: The Architecture Behind Reliable Resume

Low-power systems only become useful when they can wake up reliably.

Putting an embedded Linux system into a low-power state is one half of the problem. The other half is ensuring that the right event can bring the system back—and that software and hardware state transitions happen in the correct order.

A Sitara MPU presentation on Linux wakeup sources walks through this problem from the Linux power-management framework to device-tree configuration and suspend/resume.

## Suspend states are not all the same

Linux supports several system suspend approaches, including suspend-to-idle, standby, suspend-to-RAM and hibernate.

They represent different points on the power-saving versus resume-latency spectrum.

A deeper low-power state can provide greater savings, but it can also require more hardware and software state to be restored.

The platform's power architecture ultimately determines which states are available and which devices can remain capable of generating a wakeup event.

## What is a wakeup source?

A wakeup source identifies a device or event that is allowed to wake the system from a low-power state.

This interacts with the SoC's power domains.

Consider a system divided into MAIN, MCU and WKUP domains. During suspend, some domains may power down while a smaller set remains available to detect a wake event.

The wakeup source therefore has to survive the transition—or have a path through always-on hardware capable of signaling the wake event.

## Device Tree provides an important part of the configuration

A device can be marked as wakeup capable in Device Tree with the `wakeup-source` property:

```dts
device@10000 {
    compatible = "vendor,device-id";
    reg = <0x10000 0x1000>;
    interrupts = <0 19 4>, <0 21 4>, <0 22 4>;
    interrupt-names = "ack", "err", "wakeup";
    wakeup-source;
};
```

This tells the kernel that the device has wakeup capability.

But marking a device as wakeup capable is not the same thing as enabling a wakeup source for every situation.

## Capability versus enablement

The Linux power-management APIs distinguish between being capable of waking the system and being enabled to do so.

For example, `device_set_wakeup_capable()` marks a device as capable of waking the system. `device_set_wakeup_enable()` controls whether that wakeup capability is enabled.

Drivers can also configure wake IRQs using power-management wake-IRQ APIs.

This distinction is important when debugging systems that appear to have a correctly described wakeup source but do not actually resume.

## The suspend and resume sequence matters

Linux performs suspend and resume through multiple stages.

The suspend path includes stages such as:

- `dpm_prepare()`
- `dpm_suspend()`
- `dpm_suspend_late()`
- `dpm_suspend_noirq()`

Resume reverses the process through corresponding phases such as:

- `dpm_resume_noirq()`
- `dpm_resume_early()`
- `dpm_resume()`
- `dpm_complete()`

A wake event can occur while the system is in a low-power state, after which the resume path restores the required software and hardware state.

Understanding these stages is particularly useful when diagnosing drivers whose behavior changes between normal runtime and suspend/resume.

## A practical debugging mindset

When a system fails to wake, ask:

1. Is the intended device marked wakeup capable?
2. Is wakeup actually enabled?
3. Is the interrupt configured as a wake IRQ?
4. Does the relevant power domain remain available?
5. Does the hardware route the event to the wake-capable controller?
6. Does the driver handle suspend and resume correctly?
7. Does the system reach the intended low-power state?

This turns wakeup debugging from trial-and-error into a structured investigation.

## The bigger picture

Reliable low-power behavior is a cross-layer problem.

Linux provides the power-management framework and APIs. Device Tree describes platform capabilities. Drivers configure wake behavior. The SoC power architecture determines what remains alive.

Understanding how these layers connect is the key to building systems that are both power efficient and responsive.

---

**Related talk:** *Mastering Wakeup Sources in Linux: Architecture, APIs, and Constraints*

**Slides:** https://cfp.embedded-recipes.org/media/er2026/submissions/P8A8DR/resources/er-presentation-fin_er0aKnx.pdf

**Video:** https://www.youtube.com/watch?v=UtoPZMNts_Q
