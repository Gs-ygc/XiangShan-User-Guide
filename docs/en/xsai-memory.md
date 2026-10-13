# Memory Subsystem and HBL2 {#sec:xsai-memory}

This chapter covers the high-bandwidth L2 and matrix access path added over
Kunminghu V2R2; the base memory model is described in the baseline manual.

## HBL2 structure

TODO: capacity, bank organisation, ports and bandwidth parameters.

## Matrix access path

* Matrix requests return through a dedicated response path;
* Read hit: the local hit is returned directly through the dedicated bypass;
* Read miss: a coherence read is issued; on return the local cache line is
  updated and the data returned;
* RAW conflict: the bypass pushes the newest in-pipeline data;
* Write: write-through, requiring a write-authorisation response.

TODO: timing and backpressure details.

## Visibility and ordering

* Ordering constraints between matrix accesses and other memory accesses;
* Cases requiring explicit synchronization, and the undefined behaviour when it
  is missing.

TODO: rule list.

## Interface to the external LLC

* Multiple LLC backends are supported;
* Differences between matrix and ordinary responses once caused accounting
  desynchronisation; the fix unifies them into the same cycle.

TODO: interface parameters and configuration examples.

## Known limitations

* Behavioural and performance differences under minimal configurations;
* The matrix miss path was never exercised by software for a long time, so
  verification coverage is still being extended.
