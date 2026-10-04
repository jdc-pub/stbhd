# Exercise 1 — Make a Linux build box

> [!CAUTION]
> The original plan text was written by Claude Opus 5.5 with maximum reasoning effort.

## Objective

Set up an ARM64 Linux environment with persistent storage for kernel work. The Linux kernel source tree has filenames that differ only by letter case. A case-insensitive macOS directory shared into a Linux guest can therefore be unsafe for building. Keep the working tree on a Linux ext4 volume and copy selected artifacts across the boundary.

## What you will learn

- The difference between the macOS host and a Linux guest/container.
- Why a Linux filesystem matters for a large source tree.
- The difference between a container's writable layer and persistent volume storage.
- How CPU and memory allocation affect a compile.

## Setup

1. Check that Apple's `container` CLI is installed. Read `container --help`, `container volume --help`, `container run --help`, and `container cp --help`. The CLI is version-sensitive; use the spelling and lifecycle commands supported by the installed release.
2. Create a named volume and start a Debian ARM64 container:

   ```sh
   container volume create kbuild
   container run -it --name kbox -c 4 -m 6G -v kbuild:/src debian:trixie bash
   ```

   Adjust the CPU and memory limits to fit the Mac. Four cores and 6 GiB are only a starting point; leave enough memory for macOS. If a container named `kbox` already exists, inspect it rather than creating a conflicting one.
3. In the container, install the development prerequisites:

   ```sh
   apt update
   apt install -y build-essential bc bison flex git libelf-dev libssl-dev dwarves
   ```
4. Verify the environment and filesystem:

   ```sh
   uname -a
   uname -m
   df -T /src
   touch /src/case-test /src/Case-test
   ls -l /src/*-test
   rm /src/case-test /src/Case-test
   ```

   The architecture should be ARM64 (`aarch64` in Linux). Both differently-cased test files should exist. Confirm `/src` is on the persistent Linux volume, not a macOS bind mount.
5. Write a marker file to `/src`, exit the container, and start/reopen it using the documented command for this CLI version. Confirm the marker persists. Record the exact command used; do not assume that `run`, `start`, and `exec` have identical lifecycle semantics.
6. Test copying a small text file between the container and macOS with `container cp`. Learn the source/destination syntax before copying a kernel image.

## Deliverables

- A persistent `kbuild` volume and a reusable `kbox` development environment.
- A short note in your lab notebook with CLI version, Linux release, architecture, memory/CPU allocation, and how you verified volume persistence and case sensitivity.

## Checkpoint

You can reopen the container, find the marker in `/src`, and create both case-distinct test filenames. No kernel build has happened yet: the goal is a dependable build environment, not speed tuning.

## If something goes wrong

- If the CLI reports that its machine or service is unavailable, inspect its machine/system help and start the supported service before retrying.
- If the container is not ARM64, check the selected image platform and architecture options.
- If `/src` is not persistent, check the volume name and mount target; do not fall back to putting the kernel tree in a shared macOS directory.
- If package installation fails, save the complete apt output and resolve that before proceeding.
