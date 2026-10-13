# Performance Analysis and Tuning {#sec:sw-perf}

## Benchmarking

TODO: benchmark composition, how to run, and the metric definitions
(end-to-end throughput/latency, per-operator cycles and utilisation).

## Profiling tools

* Operator performance table: cycles/run, IPC, GOP/s, effective bandwidth,
  arithmetic intensity, peak utilisation;
* Hotspot localisation: PC hotspots, uop expansion, issue waits, memory ROI;
* Structured tracing and report generation.

TODO: tool usage and sample report.

## Tuning case studies

TODO: optimisation walkthroughs and gains for typical operators (GEMM, softmax,
FlashAttention, ...).

## Common bottlenecks

* Insufficient memory bandwidth utilisation (low effective bandwidth on some
  operators);
* Tile partitioning and buffering decisions dominate results;
* Launch overhead dominates small operators, calling for fusion.
