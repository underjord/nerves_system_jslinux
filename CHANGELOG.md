# Changelog

This project does NOT follow semantic versioning. The version increases as
follows:

1. Major version updates are breaking updates to the build infrastructure.
   These should be very rare.
2. Minor version updates are made for every major Buildroot release. This
   may also include Erlang/OTP and Linux kernel updates. These are made four
   times a year shortly after the Buildroot releases.
3. Patch version updates are made for Buildroot minor releases, Erlang/OTP
   releases, and Linux kernel updates. They're also made to fix bugs and add
   features to the build infrastructure.

## v0.4.3

* U-Boot 2026.07 is downloaded from git.u-boot-project.org, where U-Boot
  releases now live (as in upstream Buildroot). Its old site, ftp.denx.de,
  no longer answers, and the Nerves download mirror does not have this
  version, so the system could not be built from source. Same tarball
  (Buildroot's `uboot.hash` still checks it), so the images are unchanged.

## v0.4.2

* Linux: a patch (in `patches/linux`) makes virtio-i2c return `-ENXIO`
  when nothing acknowledges an address, like other I2C adapters, instead
  of 0 messages done, which `I2C_RDWR` users such as Circuits.I2C took for
  success. The kernel already has virtio-gpio and virtio-i2c; a TinyEMU
  with those devices (`gpio0` and `i2c0` in the VM config) gives the
  machine `/dev/gpiochip0` and `/dev/i2c-0`.
* Prebuilt artifacts come from this fork's GitHub releases
  (`underjord/nerves_system_jslinux`), so a project can depend on a tag
  without building the system.

## v0.4.1

This is a security and bug fix release.

* Package updates
  * [nerves_system_br 1.35.5](https://github.com/nerves-project/nerves_system_br/releases/tag/v1.35.5)
  * [Buildroot 2026.08.2](https://lore.kernel.org/buildroot/buildroot-2026.08.2-announce-1794940330@buildroot.org/T/)
  * [Erlang/OTP 29.1.3](https://erlang.org/download/OTP-29.1.3.README.md)
  * [fwup 1.17.2](https://github.com/fwup-home/fwup/releases/tag/v1.17.2)
  * Linux 6.18.58

## v0.4.0

* Boot like a real Nerves device: OpenSBI (fw_jump) replaces the 2019 BBL
  and U-Boot 2026.07 (S-mode, HTIF console, virtio-blk) loads the kernel
  from the disk image. TinyEMU loads `fw_jump.bin` as the bios and
  `u-boot.bin` as the kernel; both are now built by Buildroot and installed
  in `images/`.
* A/B firmware layout on a virtio disk, modelled on nerves_system_qemu_aarch64:
  MBR, U-Boot environment block, A/B kernel slots, A/B rootfs partitions
  and an f2fs application partition mounted on /root. `fwup-ops.conf`
  provides revert/prevent-revert/factory-reset. U-Boot picks the slot from
  `nerves_fw_active` and reverts via bootcount/upgrade_available.
* U-Boot patches (in `patches/uboot`): an `ENV_IS_IN_VIRTIO` environment
  driver that layers the fwup-written block over the built-in defaults,
  and a fix for building the qemu-riscv board without distro boot.
* Kernel: F2FS enabled. f2fs-tools added.
* The application partition ships pre-formatted (`images/appfs.img`, made
  by host f2fs-tools in post-build.sh and written by the `complete` task)
  and erlinit mounts it with `sync,dirsync`. The browser persists the disk
  in IndexedDB but only sees what the guest has flushed, so writes are made
  durable immediately instead of waiting for an f2fs checkpoint.
* Requires a TinyEMU with the OpenSBI fixes (time/mvendorid/PMP CSRs,
  HTIF `reg` in the device tree, HTIF keyboard input).
* Reboot (and so booting an upgraded slot, or reverting one) needs a
  TinyEMU with the QEMU-style `sifive,test0` syscon device and its
  `syscon-reboot`/`syscon-poweroff` nodes: OpenSBI then resets the machine
  instead of shutting it down over HTIF, which ends the emulator.

## v0.3.0

* Updated to Buildroot 2026.08 via nerves_system_br 1.35.1
* Updated Linux kernel to 6.18.57 (LTS). The config follows Frank's
  unfinished linux-6.1 branch: ARCH_VIRT, NONPORTABLE, RISCV_SBI_V01 for the
  BBL-provided legacy SBI
* Updated toolchain to GCC 15.3 (nerves_toolchain_riscv64_nerves_linux_musl 15.3.1)
* Updated Erlang/OTP to 29.1.1 and Elixir to 1.20
* Requires Nerves ~> 1.15
* Requires a TinyEMU with two fixes (see ../tinyemu, branch
  emscripten-modern): a writable scounteren.TM bit, and a reserved-memory
  node for the BBL firmware region. Linux 6.x needs both; 5.15 did not.

## v0.2.0

* Updated to Buildroot 2025.11.2 via nerves_system_br 1.33.4
* Updated Linux kernel to 5.15.180 (kept at 5.15 for TinyEMU compatibility)
* Updated toolchain to GCC 14.2.0 (nerves_toolchain_riscv64_nerves_linux_musl 14.2.0)
* Updated Erlang/OTP to 28 and Elixir to 1.17+
* Requires Nerves ~> 1.13

## v0.1.0

This is the initial release.
