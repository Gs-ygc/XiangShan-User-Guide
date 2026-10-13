# System Software and Runtime {#sec:sw-system}

## Firmware and device tree

* Firmware performs matrix extension initialisation and CSR support;
* The device tree describes matrix capabilities (`riscv,matrix-isa-caps`) and
  related hardware resources;
* **Watch `timebase-frequency`**: RTC dividers differ between simulation and
  FPGA, so a DTB must not be reused across simulators and RTL without checking
  (see [Troubleshooting](debug-troubleshooting.md#sec:debug-troubleshooting)).

TODO: firmware changes and device-tree example.

## Linux and rootfs

* Inference frameworks run on riscv64 Linux;
* rootfs/initramfs build flow; models can be packed into `gcpt.bin`.

TODO: build steps and size constraints.

## Runtime constraints

* Alignment and stride requirements for matrix accesses;
* Tile shape and data layout constraints;
* Quantization formats and scaling-factor conventions.

TODO: constraint list and common mistakes.
