# Exercise 5 — Follow firmware with coreboot in QEMU

> [!CAUTION]
> The original plan text was written by Claude Opus 5.5 with maximum reasoning effort.

## Objective

Observe a firmware boot path before an operating-system kernel starts. Build coreboot for the exact emulated board supported by its tutorial, boot it in QEMU, and follow serial output through firmware initialization and payload handoff.

This is a separate x86 exercise. It does not use the ARM64 kernel built for the M1/QEMU `virt` machine, and it does not alter Mac firmware.

## What you will learn

- What firmware does before a kernel or payload executes.
- How board configuration and the emulated machine model constrain firmware.
- How a payload differs from firmware and an operating-system kernel.
- How to use serial logs to understand boot stages.

## Steps

1. Read coreboot's ["Starting from scratch" tutorial](https://doc.coreboot.org/tutorial/part1.html) from the beginning. Record its required tools, supported QEMU machine, board name, and expected payload. Do not select a different board or assume a ROM image is interchangeable between machine models.
2. Create an isolated work directory or container for coreboot. Install only the tutorial's listed build dependencies. Record the coreboot revision and toolchain versions before building.
3. Follow the tutorial to configure the target board and payload. Review the resulting configuration and identify how serial output is enabled. Save the configuration and build log.
4. Build coreboot and locate the generated ROM. Confirm the output format/path described by the tutorial. Do not flash it to physical hardware.
5. Boot the ROM in QEMU using the machine type and firmware invocation specified by the tutorial. Route serial output to the terminal (often `-serial stdio`) and disable graphical output with `-display none` if the selected machine supports the intended headless run. A command such as `-bios build/coreboot.rom` is only appropriate if it matches the ROM format and QEMU machine for this build.
6. Capture the complete serial log. Annotate where firmware starts, performs early initialization, discovers or initializes devices, selects the payload, and transfers control. If the payload is not included or does not print to serial, diagnose its configuration rather than interpreting a silent terminal as proof that firmware did not run.
7. Change one configuration option at a time. For each change, record the expected effect and compare the resulting log. Useful comparisons include payload selection, console configuration, and one supported device option.
8. If building/running in the ARM64 Linux build container, expect x86 emulation through QEMU TCG rather than native x86 virtualization. Keep the same board model and ROM configuration; slower execution is acceptable for observing the serial log.

## Deliverables

- A pinned coreboot revision, build configuration, and build log.
- The disposable ROM build output (do not commit large generated ROMs unless explicitly desired).
- A reproducible QEMU command and annotated serial log.
- A short diagram of the firmware → payload → operating-system handoff.

## Checkpoint and concepts

You can explain which code executes before the payload, how the firmware initializes the virtual machine's devices, and where the payload takes over. Compare this firmware-driven boot to Exercise 3's direct QEMU `-kernel` launch, which provides the kernel directly and bypasses the normal firmware path.
