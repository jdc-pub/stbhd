# Exercise 4 — Add a root disk, modules, and namespaces

## Objective

Move from a temporary initramfs to a persistent ext4 root disk. Then use that system to observe syscalls, load a kernel module, and build a small Linux container-style launcher from kernel primitives.

## What you will learn

- How the kernel discovers and mounts a root block device.
- The relationship between a kernel build, its module ABI, and a `.ko` file.
- How syscalls form the boundary between applications and the kernel.
- How namespaces, mount isolation, `pivot_root`, and cgroups fit together.

## Steps

1. Use the matching QEMU kernel build from Exercise 3. Before booting without an initramfs, ensure required drivers are built in (not modules): virtio-mmio/virtio block, ext4, devtmpfs, the GIC interrupt controller, and the PL011 serial console. Rebuild and save this configuration.
2. Create a disposable Debian root filesystem containing `strace`, a shell, a usable init, and required libraries. One route is to create a container, install the packages, export its filesystem, and unpack the tar archive into a staging directory. Inspect the archive before extraction; a container export is not yet a disk image. Preserve file ownership and symlinks when extracting in Linux.
3. Make a disposable sparse ext4 image in the Linux environment. Never point `mkfs` at a physical disk:

   ```sh
   truncate -s 2G rootfs.ext4
   mkfs.ext4 -F -d rootfs rootfs.ext4
   ```

   Use an image size appropriate to the root tree, and verify its contents by mounting it read-only or inspecting it with filesystem tools. Keep the image out of Git.
4. Attach it to QEMU as a virtio block device and boot without `-initrd`. A representative device setup is:

   ```sh
   qemu-system-aarch64 \
     -M virt -cpu cortex-a72 -m 2048 \
     -nographic -no-reboot \
     -kernel Image \
     -drive if=none,file=rootfs.ext4,format=raw,id=rootdisk \
     -device virtio-blk-device,drive=rootdisk \
     -append 'console=ttyAMA0 root=/dev/vda rw rootwait init=/sbin/init'
   ```

   Match command syntax to your QEMU version and verify the guest sees `/dev/vda`. If the kernel cannot find the root device, inspect the device-driver config and boot log before changing the root path. Keep a working initramfs boot as a recovery route.
5. Inside the guest, run `strace` on small programs. Start with `strace -f -o /tmp/trace.txt /bin/echo hello`, then compare a shell command and a file-reading command. Identify `execve`, `openat`, `read`, `write`, and process-exit calls. Use `strace -e trace=...` to narrow the view and explain at least one call's arguments/result.
6. Write a small character-device module in this exercise directory. Begin with a module that registers a device and logs load/unload. Extend it to implement a predictable read/write operation. Use dynamic major allocation, expose or print its major/minor number, and clean up every registered resource on exit.
7. Enable `CONFIG_MODULES` in the exact kernel config, build the kernel and modules from the same source/config/output tree, and install/copy the module into the guest. Check `uname -r`, `modinfo`, and the module's vermagic before loading it. Load with `insmod`, inspect `dmesg`, create a node with `mknod` if devtmpfs/udev did not create one, read/write the device, then unload with `rmmod`. Keep the guest disposable; module bugs can crash it.
8. Implement a toy launcher incrementally, preferably in a small C or Rust program:
   - First create a child in new UTS and mount namespaces and demonstrate a hostname and mount change that do not affect the parent.
   - Add a PID namespace and make the child run a small init-like process that reaps children.
   - Prepare a minimal root directory, mount a private `/proc`, use `pivot_root`, detach the old root, and run a shell or test command.
   - Add cgroup v2 controls for one resource such as memory or CPU. Observe membership and limits from both sides.
   - Log syscall errors with enough context to diagnose missing privileges, mounts, or kernel config.
9. Keep the experiment unprivileged where possible and use only trusted commands. This toy launcher is not a secure sandbox; namespaces and cgroups alone do not make it safe to run hostile programs. Do not mount or modify host filesystems from the guest.

## Deliverables

- A QEMU root-disk creation recipe and a successful boot command.
- A few annotated `strace` examples.
- Source and build instructions for the character-device module.
- A toy launcher with staged commits/tests and notes about the namespaces and cgroup controls used.

## Checkpoint and concepts

The kernel mounts `/dev/vda` as its root filesystem without an initramfs; a module built against that exact kernel loads and performs its test; and your launcher demonstrates process/mount isolation plus at least one resource limit. Contrast Linux containers, which isolate processes sharing a kernel, with Apple's lightweight VM per container, which gives each container a guest kernel.
