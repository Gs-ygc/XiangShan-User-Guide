# Performance Analysis and Tuning {#sec:sw-perf}

## Benchmark

TODO: benchmark suite composition and how to run it.

Performance numbers in a report must state their basis explicitly, otherwise the numbers are meaningless:

| Basis | Meaning |
| --- | --- |
| Cycles per call | Cycles for one operator call (including synchronization overhead) |
| Cycles per MAC | Divided by the MAC count, for comparing different shapes |
| End-to-end throughput | End-to-end token/s or fps |

## Profiling tools

* Operator performance table: cycles/run, IPC, GOP/s, effective bandwidth, arithmetic intensity, share of peak;
* Hotspot localisation: PC hotspots, uop expansion, issue wait, memory ROI;
* Structured tracing and report generation.

TODO: tool usage and report examples.

## Tuning case studies

TODO: optimisation process and gains for typical operators (GEMM, softmax, FlashAttention, ...).

## Common bottlenecks

* **Launch overhead dominates small operators**: operator fusion is needed, or a workload guard hands the tasks that are not worth it back to RVV;
* **Insufficient memory bandwidth utilisation**: the effective bandwidth of some operators is clearly below peak;
* **Tile partitioning and buffering strategy matter a lot**: when M utilisation is poor (the convolution hook point supplies only 8 pixels at a time) the task should fall back to the small core;
* **Clock configuration**: when the matrix unit and the core run at different frequencies, the placement of synchronization points has a large effect on the overall gain (see [Microarchitecture Delta](xsai-microarchitecture.md#sec:xsai-microarchitecture)).
