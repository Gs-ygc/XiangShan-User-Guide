# Introduction {#sec:xsai-introduction}

## Scope

Xianglufeng (project code name XSAI) is a RISC-V processor for AI inference
built on top of XiangShan Kunminghu V2R2. This manual documents **only the delta
over Kunminghu V2R2**; for everything else see the _XiangShan Open-Source
Processor User Guide_.

## Relation to Kunminghu V2R2

* Baseline: XiangShan Kunminghu V2R2
* Delta: AME matrix extension, matrix control path, high-bandwidth L2 (HBL2),
  external LLC, asynchronous matrix execution
* Software-visible changes: new instructions and CSRs, device-tree capability
  description, matrix access alignment and transpose constraints, asynchronous
  synchronization semantics
* Debug-visible changes: matrix register support in difftest, matrix-related
  performance counters, checkpoint sampling

## Delta at a glance

| Dimension | What is added | See |
| --- | --- | --- |
| ISA | ZAMES / ZMASYNC, matrix load/store and transpose, low-precision convert instructions | [Instruction Set Extensions](xsai-instruction-set.md#sec:xsai-instruction-set) |
| Registers | Matrix, configuration and synchronization registers | [Registers](xsai-registers.md#sec:xsai-registers) |
| CSRs | Matrix extension state and precision capability description | [Modes and CSRs](xsai-csr.md#sec:xsai-csr) |
| Exceptions | Illegal-instruction, alignment and UB conventions for the new instructions | [Exceptions and Interrupts](xsai-exception.md#sec:xsai-exception) |
| Memory | HBL2, matrix access path, RAW bypass and miss handling | [Memory Subsystem and HBL2](xsai-memory.md#sec:xsai-memory) |
| PMU | AI-related HPM events, structured tracing | [Performance Monitor](xsai-pmu.md#sec:xsai-pmu) |
| Debug | Matrix support in difftest, checkpoint sampling | [Debug Support](xsai-debug.md#sec:xsai-debug) |
| Clocking | Asynchronous or same-clock connection between CUTE and the core (configurable) | [Microarchitecture Delta](xsai-microarchitecture.md#sec:xsai-microarchitecture) |

## Reading paths

* Hardware users: instruction set → registers → CSRs → memory subsystem
* Verification and debug: the Simulation and Debug chapters
* Software developers: the Software Development chapters
