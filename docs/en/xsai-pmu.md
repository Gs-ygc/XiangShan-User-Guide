# Performance Monitor {#sec:xsai-pmu}

This chapter covers the performance monitoring additions of Xianglufeng; the
baseline HPM is described in the _XiangShan User Guide_.

## HPM events

TODO: matrix/memory/AI related event numbers and meanings.

## Structured tracing

* Key units can record structured events for offline analysis;
* Relation to waveforms and HPM.

TODO: usage and field description.

## Recommended analysis flow

1. Operator-level performance table (cycles, IPC, effective bandwidth,
   arithmetic intensity, peak utilisation);
2. HPM to classify the bottleneck (memory / compute / scheduling);
3. Structured tracing to localise the module and the moment.

TODO: worked example.
