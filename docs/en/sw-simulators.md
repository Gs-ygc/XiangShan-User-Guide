# Functional Simulators {#sec:sw-simulators}

## NEMU

* Matrix extension support (configuration, MMA, memory access, synchronization);
* Undefined-behaviour checks (out-of-range matrix register access, missing
  `mfence`, ...);
* Checkpoint sampling and compression.

TODO: build, run and check toggles.

## QEMU

* Supports matrix ISA versions (e.g. XSAME-v0.6);
* Both user and system modes; a sampling acceleration method is available.

TODO: build, run and sampling usage.

## Choosing a simulator

| Scenario | Recommendation | Note |
| --- | --- | --- |
| Fast functional alignment | NEMU | Lightweight, rich checks |
| Full Linux/model | QEMU system mode | Complete system environment |
| Step-by-step RTL comparison | Either + difftest | See [Differential Testing](debug-difftest.md#sec:debug-difftest) |

TODO: performance magnitudes and limitations.
