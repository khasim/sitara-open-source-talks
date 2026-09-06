---
layout: post
title: "Scaling Secure Boot with U-Boot Binman and Hardware Security Modules"
author: "Sitara MPU Software Team"
date: 2026-09-05
categories: [Security, U-Boot, Secure Boot]
tags: [Binman, HSM, PKCS11, Embedded Linux]
description: "How U-Boot Binman and HSM-backed signing can turn secure-boot image creation into a scalable, auditable workflow."
featured: false
---

# Scaling Secure Boot with U-Boot Binman and Hardware Security Modules

Secure boot is straightforward to describe: a device should execute only software authenticated by a trusted authority. The engineering challenge becomes much harder when that process has to scale across multiple images, teams and build environments.

A Sitara MPU engineering talk explored how **U-Boot Binman** and **Hardware Security Modules (HSMs)** can be combined to make firmware signing more controlled and scalable.

## Why secure boot becomes difficult at scale

A traditional signing workflow can start simply: a developer has a private key, a script invokes a cryptographic tool, and the resulting image is packaged for boot.

That model becomes uncomfortable in production. Private keys stored on developer machines increase the risk of compromise, while manual signing becomes difficult to audit. Multiple firmware components and teams also increase the number of places where key handling can go wrong.

The question is therefore not only *how do we sign an image?* but:

> How do we integrate secure signing into an engineering workflow without exposing the private key?

## Moving the private key behind an HSM boundary

An HSM provides a protected environment for cryptographic keys and operations. Instead of exporting a private key to a build machine, the build flow asks the HSM to perform the signing operation.

The build system can prepare the data that needs to be signed, but the private key remains inside the secure boundary. Depending on the deployment model, the HSM can be a software HSM for development, a hardware token, a centralized server HSM, or a cloud HSM.

For production systems, the important architectural property is that the signing key does not have to become a normal file in the build environment.

## Where Binman fits

U-Boot Binman brings together the components required to create bootable firmware images. It provides a structured way to describe the image and its components instead of maintaining a collection of independent packaging scripts.

That makes Binman a natural point to integrate signing.

The flow is:

1. Define the image and signing configuration.
2. Build the required firmware components.
3. Let Binman assemble the image structure.
4. Invoke the signing operation through the configured cryptographic backend.
5. Generate the required certificate/signature material.
6. Produce the final bootable image.

The image-generation flow does not need to be redesigned for every type of HSM.

## PKCS#11 as the abstraction layer

A major part of the approach is **PKCS#11**, a standard interface for applications to interact with cryptographic tokens and devices.

With PKCS#11, the signing key can be referenced as a cryptographic object rather than treated as an ordinary PEM file.

This creates a useful separation:

**Image creation stays the same. Key storage and signing backend can change.**

The presentation evaluated software, hardware and server-based HSM approaches. The configuration differs between them, but the overall Binman flow can remain consistent.

## What this means for CI/CD

A scalable secure-boot pipeline should make key access intentional rather than incidental.

A centralized or service-based HSM can provide useful operational characteristics for a multi-user build environment: centralized key control, controlled access and integration with automated signing workflows.

A local hardware token can be useful for development and controlled signing scenarios, while a software HSM can provide a convenient environment for functional testing.

The important design principle is to avoid coupling the build system to a local copy of a production private key.

## The bigger lesson

Secure boot is not only a cryptography problem. It is also a **software supply-chain and engineering-process problem**.

U-Boot Binman provides structure around image creation. HSMs provide a protected signing boundary. PKCS#11 provides an abstraction between the two.

Together, these pieces can help create a signing architecture that is more portable, auditable and scalable.

---

**Related talk:** *Leveraging U-Boot Binman with Hardware Security Modules (HSM) for Secure Boot*

**Slides:** https://hosted-files.sched.co/osselcna2026/d6/Leveraging_Binman_with_HSM_for_SecureBoot.pdf

**Video:** https://www.youtube.com/watch?v=BjAIXqVyi7w
