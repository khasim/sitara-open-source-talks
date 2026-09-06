---
layout: post
title: "Zero-Copy DSP Offload in Linux: Keeping Control in Linux and Compute on the DSP"
author: "Sitara MPU Software Team"
date: 2026-09-05
categories: [Linux, DSP, RPMsg, Heterogeneous Computing]
tags: [RPMsg, DMA, C7x, AM62D, Audio]
description: "A Linux-controlled architecture for real-time DSP workloads using RPMsg, shared DMA buffers and a zero-copy data model."
featured: false
---

# Zero-Copy DSP Offload in Linux: Keeping Control in Linux and Compute on the DSP

Modern embedded SoCs increasingly combine application processors with dedicated compute engines. The challenge is not simply getting a workload onto the accelerator—it is integrating that accelerator cleanly into a Linux software stack.

A Sitara MPU engineering presentation explores this problem using a **Linux-controlled DSP offload architecture** built around RPMsg and shared DMA-capable memory.

The central idea is:

> **Keep Linux as the system orchestrator and use the DSP as the compute accelerator.**

## The problem is integration

For multi-channel, real-time DSP workloads, general-purpose CPU execution can be inefficient.

Moving the workload to a DSP can improve compute efficiency, but naïve offload designs can introduce another bottleneck: data movement.

If audio or signal data is repeatedly copied between ARM memory and DSP memory, the cost of the copies can reduce the benefit of acceleration.

The design therefore treats **zero-copy data movement as a first-class architectural principle**.

## Separate the control plane from the data plane

The proposed architecture separates two responsibilities.

**Linux owns the control plane.**

It manages application behavior, DSP lifecycle, communication and buffer management.

**Shared memory forms the data plane.**

Large data buffers are allocated in shared, DMA-capable DDR memory rather than repeatedly copied between processor domains.

RPMsg is then used primarily to communicate control and notification information.

## A reusable heterogeneous-compute pattern

The architecture includes Linux kernel components, an RPMsg/DMA support library, shared DDR buffers and DSP firmware.

At a high level:

1. Linux allocates a DMA-capable shared buffer.
2. Linux places input data into the buffer.
3. Linux notifies the DSP through RPMsg.
4. DSP firmware processes the data.
5. Results or KPIs are written back to the shared buffer.
6. DSP notifies Linux that processing is complete.

The DSP can perform real-time signal processing without requiring the application architecture to move entirely away from Linux.

## Why RPMsg?

RPMsg provides an established communication mechanism for heterogeneous processors.

In this architecture it acts as the communication channel between the Linux-controlled application processor and the DSP.

The important distinction is that RPMsg does not have to carry the entire data payload.

Instead, the data can live in shared memory while RPMsg communicates events, commands and synchronization information.

That is what makes the architecture suitable for a zero-copy-oriented design.

## Example: audio processing

The presentation uses the **AM62D** audio and signal-processing SoC as an example.

The device combines quad Cortex-A53 cores running Linux with a C7 vector DSP for signal processing and R5F cores for real-time control.

A multi-channel audio application can remain Linux-oriented while the DSP handles the time-sensitive signal-processing chain.

This is useful for workloads such as FFTs and other real-time audio processing stages.

## Reusing existing DSP kernels

Another important goal is to avoid forcing application developers to rewrite their complete software stack just to use an accelerator.

The architecture allows pre-built DSP kernels to be reused while Linux continues to provide application-level orchestration.

That separation can make heterogeneous computing easier to integrate into existing Linux systems.

## Zero-copy is a design discipline

Calling a system "zero-copy" does not automatically make it efficient.

The full path needs to be considered:

- Where is the buffer allocated?
- Is it DMA capable?
- Which processor owns the buffer at each stage?
- How is synchronization performed?
- How are cache effects handled?
- How is completion signaled?
- Are there hidden copies inside higher-level frameworks?

The benefit comes from designing the complete data path around shared buffers and explicit ownership.

## Why this matters beyond audio

The architecture is a useful pattern for heterogeneous embedded systems more generally.

Whenever Linux needs to orchestrate a workload while a dedicated accelerator performs deterministic processing, a separation between control messaging and shared data can reduce unnecessary movement and preserve a familiar Linux application model.

**Linux controls. The accelerator computes. Shared memory carries the data.**

---

**Related talk:** *Building a Zero-Copy DSP Offload Framework in Linux Using RPMsg*

**Slides:** https://hosted-files.sched.co/ossindia2026/c5/Linux%20Based%20Zero-Copy%20DSP%20Offload%20Framework.pdf

**Video:** https://www.youtube.com/watch?v=kEb_UzENBfE
