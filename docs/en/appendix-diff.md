# Appendix A: Delta over Kunminghu V2R2 {#sec:appendix-diff}

| Module | Delta | Affected areas | See |
| --- | --- | --- | --- |
| Matrix unit | Matrix compute and memory paths, transpose engine | ISA, memory, performance | [Microarchitecture Delta](xsai-microarchitecture.md#sec:xsai-microarchitecture) |
| Matrix control path | Command dispatch, completion notification, commit semantics | Instruction commit, exceptions | as above |
| Registers | Matrix, configuration, synchronization registers | Programming model | [Data Registers](xsai-registers.md#sec:xsai-registers) |
| CSRs / device tree | Extension state and precision capability description | Firmware, OS | [Privileged Modes and Control and Status Registers](xsai-csr.md#sec:xsai-csr) |
| High-bandwidth L2 | HBL2 and the dedicated matrix access path | Memory subsystem, performance | [Memory Subsystem and HBL2](xsai-memory.md#sec:xsai-memory) |
| Clock domains | Asynchronous matrix unit, same-clock mode retained | Integration, timing | [Microarchitecture Delta](xsai-microarchitecture.md#sec:xsai-microarchitecture) |
| PMU | AI-related events and structured tracing | Tuning | [Performance Monitor](xsai-pmu.md#sec:xsai-pmu) |
| Verification | Matrix support in difftest, checkpoint sampling | Verification flow | [Differential Testing](debug-difftest.md#sec:debug-difftest), [Checkpoint](debug-checkpoint.md#sec:debug-checkpoint) |

TODO: add quantitative status (pass rates, coverage).
