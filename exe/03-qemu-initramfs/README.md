# Exercise 3 — Boot a kernel and initramfs in QEMU

## Objective

Boot an ARM64 kernel on QEMU's generic `virt` machine, provide your own initial userspace, and inspect the kernel handoff with a serial console and debugger. This virtual platform is not an emulation of the M1 SoC.

## What you will learn

- How firmware or a virtual machine hands control and boot data to an ARM64 kernel.
- What the kernel command line and device tree are for.
- How an initramfs supplies the first userspace process.
- How a serial console and GDB help diagnose early boot.

## Prerequisites and artifacts

Use the Linux build environment and a second kernel configuration dedicated to QEMU. Keep it separate from the Apple container config in Exercise 2. Install QEMU on macOS (for example, with Homebrew) and check the installed `qemu-system-aarch64` and GDB versions. On Apple Silicon, QEMU may use Hypervisor.framework acceleration for normal runs; use TCG emulation if the accelerator prevents the debugger workflow.

Create this exercise's artifacts under the directory or on the Linux build volume, but do not commit large generated binaries:

- `config/qemu-arm64.config`
- `initramfs/` containing `/init`, BusyBox, and required directories
- `Image`, matching `vmlinux`, and compressed initramfs (generated)
- a boot script and concise run/debug notes

## Steps

1. In the Linux build box, copy the vendored kernel source to the ext4 build volume. Start from ARM64 `defconfig`, then inspect the configuration for serial console, initramfs, devtmpfs, and the virtual devices used by QEMU. Later, enable virtio block and ext4 for Exercise 4. Save the final config and source revision together. Build `Image` and retain the matching unstripped `vmlinux`.
2. Make a minimal initramfs directory. Include a statically linked BusyBox, links for the applets you use, executable `/init`, and a console device node if the chosen boot setup requires one. A simple first `/init` can mount pseudo-filesystems, print a marker, and exec a shell:

   ```sh
   #!/bin/busybox sh
   /bin/busybox mount -t proc proc /proc
   /bin/busybox mount -t sysfs sysfs /sys
   /bin/busybox mount -t devtmpfs devtmpfs /dev
   echo 'bsthd: initramfs is running'
   exec /bin/busybox sh
   ```

   Install the BusyBox binary at the path referenced by the script and ensure `/init` is executable. Create needed directories (`bin`, `dev`, `proc`, `sys`, `tmp`). If `devtmpfs` is not available, create `/dev/console` in the initramfs using an environment that supports device nodes.
3. Archive the contents from inside the initramfs root, not the parent directory, using `newc` format and gzip:

   ```sh
   find . -print0 | cpio --null -o --format=newc | gzip -9 > ../initramfs.cpio.gz
   ```

   Inspect the archive listing, confirm `init` appears at the archive root with executable permissions, and verify the archive is non-empty.
4. Boot on the generic ARM `virt` machine with the kernel and initramfs:

   ```sh
   qemu-system-aarch64 \
     -M virt -cpu cortex-a72 -m 2048 \
     -nographic -no-reboot \
     -kernel Image -initrd initramfs.cpio.gz \
     -append 'console=ttyAMA0 rdinit=/init'
   ```

   Use the matching image paths and inspect QEMU's help if option names differ. Confirm boot text and the `/init` marker arrive through `ttyAMA0`. Expect the guest shell to remain attached to the terminal. Learn how to quit QEMU cleanly before starting.
5. Modify `/init` to print a second unique message or run a different command, rebuild only the initramfs, and boot again. This shows that kernel and userspace are independent artifacts.
6. Debug early boot with the exact matching `vmlinux`. Add `nokaslr` to the guest command line, start QEMU paused with its GDB stub (commonly `-S -s`), and run an AArch64-capable GDB:

   ```gdb
   file vmlinux
   target remote :1234
   hbreak start_kernel
   continue
   ```

   Inspect registers and the backtrace, step briefly, then continue to userspace. QEMU's GDB stub typically listens on TCP port 1234; use a different port if it is already occupied. Do not use symbols from a different kernel build.
7. Save a reproducible QEMU command or script with paths passed as arguments/environment variables. Keep the serial output from one successful run as a small text log if useful.

## Deliverables

- A saved QEMU kernel config and build notes.
- A minimal initramfs recipe and `/init` source.
- A repeatable QEMU boot command/script and an annotated successful boot log.
- Debugger notes identifying the breakpoint and at least one observed kernel state.

## Checkpoint and concepts

You reach a shell from your own initramfs, see your marker over the serial console, and stop at `start_kernel` with matching symbols. Draw the path from QEMU's machine setup and boot data through kernel initialization to `/init`. Explain how `rdinit=` selects an initramfs program and how that differs from selecting a later init on a mounted root filesystem.
