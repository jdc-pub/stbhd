# `stbhd`

## Computer architecture and Linux kernel lab

This repository is a hands-on learning path for an M1 MacBook Air. Each exercise has its own directory and README under [`exe/`](exe/). Work from generic ARM virtual hardware first; keep experiments isolated from macOS boot policy and partitions.

> [!CAUTION]
> The original plan text was written by Claude Opus 5.5 with maximum reasoning effort.

Keep a lab notebook for every exercise: tool versions, source revision, config, exact build and boot commands, and observed output. Do not commit generated build output, disk images, or credentials unless a step explicitly calls for a small source artifact.

## Exercises

1. [Make a Linux build box](exe/01-build-box/README.md) — create a persistent Linux environment and understand the filesystem boundary between macOS and Linux.
2. [Build, boot, and patch a kernel](exe/02-kernel/README.md) — vendor a pinned mainline kernel tree, configure ARM64, build it, and run it with Apple's container tool.
3. [Boot a kernel and initramfs in QEMU](exe/03-qemu-initramfs/README.md) — make a minimal userspace, boot generic ARM virtual hardware, and inspect boot with GDB.
4. [Add a root disk, modules, and namespaces](exe/04-rootfs-modules-containers/README.md) — boot a disk-backed system, inspect syscalls, load a module, and build a toy container launcher.
5. [Explore firmware with coreboot](exe/05-coreboot/README.md) — follow a virtual x86 machine from firmware initialization to its payload.
6. [Write a tiny virtual-machine monitor](exe/06-tiny-vmm/README.md) — use Hypervisor.framework to run guest instructions and emulate a simple device.

## Progression

The exercises intentionally move from the simplest boundary to the most complex: host/container → kernel → virtual machine and initial userspace → persistent root filesystem and kernel interfaces → firmware → hypervisor and device emulation. Finish each checkpoint before moving on; keep the ARM64/QEMU and x86/coreboot experiments conceptually separate.
