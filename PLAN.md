# `bsthd`

## Plan

> [!CAUTION]
> The text in this section was written by Claude Opus 5.5 with maximum reasoning effort.

1. **Build box.** Run `container volume create kbuild`, then `container run -it --name kbox -c 8 -m 4G -v kbuild:/src debian:trixie bash`, and keep all source code under `/src`. Volumes are formatted as ext4, and that matters: the kernel tree has filenames that differ only by uppercase vs. lowercase, which a folder shared from macOS will mangle. `container cp` copies files between a running container and your Mac.

2. **Your kernel, your containers.** Clone mainline Linux and start from the `config-arm64` file in Apple's containerization repo. Set `CONFIG_LOCALVERSION` so you can recognize your build, and build the `Image` target. Copy it to the Mac and pass it to `container run` with `-k`; running `uname -r` inside the container proves your kernel booted. Then add a `pr_err()` line to `start_kernel`, rebuild, and find your message in `container logs --boot`. That's your patch → build → boot loop.

3. **Own the boot.** Running VMs inside `container` (nested virtualization) needs an M3 or newer, so install QEMU on macOS, where `-accel hvf` gives you fast boots. Use a second kernel built from the stock arm64 `defconfig`. Boot it with an initramfs whose `/init` is a static C or Rust program you wrote: it mounts `/proc`, `/sys` and `/dev`, reaps zombies, and starts a static busybox shell. To watch boot in a debugger, run QEMU inside the container (emulated, so slower) with `-s -S` and `nokaslr`. Then attach gdb to `vmlinux`, set `hbreak start_kernel`, and step until your init runs.

4. **Real root filesystem.** `container export` a Debian container that has strace installed, and turn the tarball into a disk image with `mkfs.ext4 -d`. Boot it in QEMU with `root=/dev/vda`, with `init=` pointing at your own init, and no initramfs. Inside that VM:
   - Trace syscalls with strace.
   - Write and load a character-device kernel module.
   - Build a toy container runtime from namespaces, `pivot_root` and cgroups. It's a good contrast with Apple's design of one lightweight VM per container.

5. **Firmware.** coreboot's "Starting from scratch" tutorial builds coreboot's own toolchain and a ROM for an emulated x86 board, then boots it with `qemu-system-x86_64 -bios build/coreboot.rom -serial stdio`. Run it in the container and add `-display none`, since there's no screen. Follow the serial log stage by stage until the payload takes over.

6. **A tiny VMM.** Apple's Hypervisor.framework works on the M1. Write a Rust program that creates a VM, maps memory, runs a few guest instructions, and handles the exits for a fake serial port (UART). Sign it with the `com.apple.security.hypervisor` entitlement; a local self-signed signature is enough. Stretch goal: boot your kernel in it.
