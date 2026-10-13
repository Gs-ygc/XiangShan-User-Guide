# System Software and Runtime {#sec:sw-system}

## Device tree and capability discovery

* Matrix capabilities are described by device-tree nodes (`riscv,matrix-isa-caps` and similar), replacing the earlier approach of describing capabilities with a bitmap (`xmisa`);
* Firmware is responsible for filling in the capability description, and the operating system and runtime decide from it whether to enable the matrix path;
* The reported capabilities must match the actual RTL configuration, otherwise software generates kernels against the wrong tile shape.

TODO: complete list of device-tree nodes and properties.

## Operating system support

TODO: kernel version, patch set, and how matrix registers are handled across context switches.

## Runtime and memory

* Matrix operands must reside in a physically contiguous memory pool (`/dev/mem` @ `0x100000000`);
* The allocation mode (pool mode / anonymous mode) must be decided by the external environment and must not be compiled into the artifact:
  * anonymous mode is only for QEMU user mode;
  * if it is carried onto the board unconditionally, the result is `SIGABRT` (`rc=134`), or a wrong result under the anonymous pool that is intercepted by the guard (`rc=255`).
* AMU row addressing is computed as `PA_base + row × stride` and the TLB is not re-checked per row, so a non-contiguous buffer is read incorrectly and silently when the row count is large.

## Boot flow

TODO: flow and limitations of checkpoint-restore boot (GCPT), see [Checkpoint and Slicing](debug-checkpoint.md#sec:debug-checkpoint).
