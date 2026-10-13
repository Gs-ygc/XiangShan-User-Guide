# Performance Monitor {#sec:xsai-pmu}

This chapter documents the performance monitoring capabilities added by
Xianglufeng; for the baseline HPM see the _XiangShan Open-Source Processor User
Guide_ (Performance Monitoring Unit chapter).

## HPM events

TODO: list the numbers and meanings of the matrix / memory-access / AI related
events.

## Structured tracing

* Key units support structured event recording for offline analysis;
* The relation to waveforms and HPM.

TODO: add the usage and the field descriptions.

## Recommended analysis flow

1. Kernel-level performance table (cycles, IPC, effective bandwidth, arithmetic
   intensity, fraction of peak);
2. HPM to classify the bottleneck type (memory access / compute / scheduling);
3. Structured tracing to localise the specific module and moment.

## Reporting conventions

Performance numbers in a report must state whether they are per call, Per-MAC or
whole-network, otherwise the numbers are not comparable. Distinguishing "the
error introduced by the matrix unit" from "the error of quantisation /
truncation itself" is equally important: the criterion for verifying that the
adaptation is correct is bit-level identical output with the matrix switch on and
off, not the accuracy difference against a particular version of the quantised
model.

TODO: add examples.
