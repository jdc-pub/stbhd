# Exercise 2 — Build, boot, and patch a kernel

## Objective

Keep a pinned, vendored copy of upstream Linux in this exercise directory, build an ARM64 kernel with a container-focused configuration, boot it with Apple's `container` CLI, and prove that a source change made it into the running kernel.

## Repository layout and source policy

The intended source location is `vendor/linux/`. This is vendored source, not a Git submodule and not a nested Git repository: upstream files are tracked by this repository so a checkout contains the source directly. Preserve upstream licensing and copyright files. Record the upstream repository URL, exact commit ID (or release tag and resolved commit), snapshot/archive checksum, date imported, and any local patches in `vendor/README.md`. Do not describe a moving `master` branch as a reproducible version. Avoid committing generated object files or build products; keep the build tree on the ext4 volume from Exercise 1.

The mainline source tree is large. Before importing it, make sure the repository host/storage budget is appropriate and agree on whether you want the entire source tree in Git or a separate large-file strategy. Do not silently substitute a submodule, partial clone, or download-on-build script: those do not satisfy the vendored-source goal.

## What you will learn

- How `.config` selects kernel features and how `olddefconfig` updates it.
- The distinction between source, configuration, the linkable `vmlinux`, and the bootable ARM64 `Image`.
- How a virtualized container launch uses a supplied kernel.
- How to trace a source edit through compile, boot, and kernel logs.

## Steps

1. Import a pinned upstream Linux mainline snapshot into `vendor/linux/`. Keep a record of the exact revision and verify that the extracted tree has no nested `.git` directory. Preserve files such as `COPYING`, `LICENSES/`, and relevant per-file notices.
2. Obtain Apple's `config-arm64` from the `kernel/` directory of the `apple/containerization` project. Store an unchanged copy in this exercise (for example, `config/containerization-arm64.config`) and note the upstream revision from which it came. This is a container-oriented starting configuration, not a general-purpose QEMU config.
3. Copy the vendored tree and config into `/src` on the Linux ext4 volume, using the CLI's file-copy mechanism. Build there so case-distinct kernel source filenames remain distinct and Git does not accumulate compiler outputs.
4. In the copied source tree, install the saved configuration and set a recognizable local release suffix. Preserve the unmodified Apple config separately:

   ```sh
   cp /path/to/containerization-arm64.config .config
   scripts/config --set-str LOCALVERSION '-bsthd'
   scripts/config --disable LOCALVERSION_AUTO
   make ARCH=arm64 olddefconfig
   grep -E 'CONFIG_LOCALVERSION=|CONFIG_LOCALVERSION_AUTO=' .config
   ```

   If `scripts/config` is unavailable or behaves differently for this revision, edit `.config` carefully and confirm the resulting values. `olddefconfig` can change options as kernel versions evolve; inspect and save the resolved config.
5. Build the ARM64 `Image`, starting with a conservative parallelism appropriate to available RAM:

   ```sh
   make -j2 ARCH=arm64 Image
   ```

   The image is `arch/arm64/boot/Image`. Retain `vmlinux`, `.config`, the build log, and compiler/tool versions on the volume for diagnosis and later debugging.
6. Copy `Image` to macOS. Launch a disposable container with the custom kernel option supported by the installed CLI (Apple's CLI currently exposes `-k` / `--kernel`):

   ```sh
   container run --name bsthd-kernel-test -k /path/to/Image debian:trixie uname -a
   ```

   Use the exact image, arguments, and lifecycle options accepted by your installed CLI. Do not replace a host kernel. If the command fails, save the complete CLI error and boot log, then compare the kernel config to the container's requirements.
7. Query the guest release and boot logs:

   ```sh
   uname -a
   uname -r
   ```

   Fetch the boot log with the CLI's supported equivalent of `container logs --boot <container-id>`. Confirm that the local release suffix appears. Also perform a control run with the default kernel and note the differences.
8. Make a minimal traceable change in the working copy. Add a distinctive `pr_err()` message near the beginning of `start_kernel()` in `init/main.c`; do not edit the vendored copy directly if you want to keep a clean upstream baseline. Save the patch (for example, under `patches/0001-print-start-kernel-marker.patch`), rebuild `Image`, copy it out, and launch a new disposable container. Search the boot log for the marker.
9. Rebuild once after reverting the patch, or apply it from the saved patch file, to prove that your loop is repeatable. Record the patch state and kernel release for each run.

## Deliverables

- Vendored upstream Linux under `vendor/linux/`, without a nested `.git` directory.
- `vendor/README.md` with revision, origin, import date/checksum, license note, and local patch policy.
- A preserved input config and the resolved `.config` used for the build.
- A patch that adds the boot marker, plus notes showing the marker in the matching guest's log.

Do not check in full compiler output, object files, `vmlinux`, or `Image` unless you explicitly decide the repository should store large build artifacts. The source and small reproducibility metadata are the deliverables; builds belong in the Linux volume.

## Checkpoint and concepts

You can build an ARM64 image from the vendored revision, boot it in a disposable container, identify it via `uname -r`, and observe a message compiled from your own patch. Explain why a matching source revision and config are needed to reproduce the image, and why a successful compile alone does not prove that the container runtime can boot it.
