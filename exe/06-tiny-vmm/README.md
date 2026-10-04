# Exercise 6 — Write a tiny ARM virtual-machine monitor

## Objective

Write a small Rust program on the M1 that uses Apple's Hypervisor.framework to execute guest instructions. Add a simple memory-mapped serial device only after a minimal guest runs. This exercise builds understanding of virtualization APIs and device emulation without requiring a complete machine model.

A virtual-machine monitor (VMM) is the program that creates and manages a guest. Hypervisor.framework supplies host virtualization APIs; it does not automatically provide the devices or firmware expected by a guest operating system.

## What you will learn

- The separation between host and guest CPU state and memory.
- How guest execution exits to the VMM.
- How a VMM emulates a device at an MMIO boundary.
- Why booting a full OS requires firmware-like setup and multiple device models.

## Steps

1. Read Apple's Hypervisor.framework documentation for the installed macOS SDK and check availability/entitlement requirements for the host system. First compile and run a tiny program that creates and destroys a VM. Confirm the `com.apple.security.hypervisor` entitlement is present in the signed executable; do not assume that ad-hoc signing or a local identity is sufficient on every macOS version.
2. Create a Rust crate in this exercise directory. Keep the Hypervisor.framework FFI narrow, check every API return code, and ensure VM memory and vCPU resources are released on both normal and error paths. Add a README section in the crate for build, sign, and run commands once verified locally.
3. Create the VM and allocate a page-aligned host memory region for guest RAM. Map it at a guest physical address with appropriate permissions. Add bounds checks so guest addresses cannot index outside the allocated region.
4. Create one vCPU, initialize its architectural state and program counter, and run a tiny known ARM64 instruction sequence from guest memory. Initially the guest can execute a few arithmetic instructions and stop. Read back a register or other state to prove the expected instructions ran.
5. Add explicit handling for every exit reason the test can produce. Log the exit type and relevant guest state. Do not treat unexpected exits as success. Add tests for guest-memory bounds, invalid instruction behavior, and cleanup on error.
6. Add a fake UART only after the instruction-only guest works. Choose and document a guest physical MMIO address. Have guest code write a character to a UART-like transmit register; detect the MMIO exit, validate address/width/direction, and print the character on the host. Handle unsupported accesses explicitly.
7. Use a guest program with multiple writes and a readable status register as a test. Verify that the VMM observes each access exactly once and that an unsupported register or access width fails safely.
8. Stretch goal: attempt to boot a Linux kernel from Exercise 3. Revisit the ARM64 boot protocol and provide the boot data and device tree expected by the kernel. A useful guest needs more than a CPU and serial port: it generally needs a timer, interrupt controller, memory map, and any device drivers selected in the kernel config. Bring up one dependency at a time and keep the minimal instruction guest as a regression test.

## Deliverables

- A Rust crate that creates a VM and executes a known guest instruction sequence.
- A tested and documented host signing/entitlement procedure.
- A small UART-like MMIO implementation and guest-side test program.
- Optional kernel boot notes; do not make full Linux boot a prerequisite for considering this exercise complete.

## Checkpoint and concepts

The guest executes deterministically, the VMM reports expected exit state, and a guest MMIO write reaches the host terminal. Explain the distinction between CPU virtualization, guest physical memory, a VM exit, device emulation, and OS boot. Full Linux boot is a stretch goal because each missing virtual device or boot handoff requirement can prevent the kernel from reaching userspace.
