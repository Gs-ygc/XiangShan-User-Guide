# Functional Simulators {#sec:sw-simulators}

Functional simulators are used to get software running end to end without RTL
and to perform operator-level and network-level differential verification.

## NEMU

* The matrix extension has been merged upstream and supports matrix instructions
  and matrix register state;
* When it serves as the reference model for differential verification, it
  compares matrix registers by hash (see
  [Differential Testing](debug-difftest.md#sec:debug-difftest));
* It comes with its own performance statistics: with AVX-512 enabled, it reaches
  a 181.71× speedup for FP16 and 285.52× for FP32 over interpreted execution.

## QEMU

* User mode (`qemu-riscv64`): runs operator-level tests and the single-operator
  tests shipped with frameworks, and iterates fastest;
* System mode: runs the full boot flow and Linux user programs;
* Several people keep patching the same QEMU branch (the XSAME version), so
  remember to synchronize before merging.

**What QEMU conclusions mean on the matrix path**: QEMU models arithmetic
semantics and does not cover hardware geometry constraints (tile spec,
physically contiguous pool, row addressing), so passing everything in QEMU does
not imply that it is correct on the board. QEMU conclusions must be labelled
"arithmetic criteria only", and only re-verification on the board is
authoritative.

## gem5

* Used for design exploration and performance evaluation, with a deviation
  against RTL of about 2.76%;
* Suitable for architecture parameter sweeps, not for deciding functional
  correctness.

## Division of labour with RTL simulation

| Tool | Speed | Scope of criteria | Use case |
| --- | --- | --- | --- |
| QEMU (user mode) | Fastest | Arithmetic only | Single operators, fast network-level iteration |
| NEMU | Fast | Instruction semantics + register state | Reference model for differential verification |
| gem5 | Medium | Performance magnitude | Architecture parameter exploration |
| Verilator / VCS | Slow | Full functionality | Re-verification before submission, waveform debugging |
| FPGA | Near real time | Full functionality (authoritative) | Board-level conclusions |

TODO: build methods and version mappings for each simulator.
