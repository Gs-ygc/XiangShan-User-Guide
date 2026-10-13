# Memory Subsystem and HBL2 {#sec:xsai-memory}

This chapter covers the high-bandwidth L2 and the matrix memory access path
added over Kunminghu V2R2; the general memory model is described in the baseline
manual.

## HBL2 structure

TODO: capacity, bank organisation, ports and bandwidth parameters.

## Matrix memory access path

* Matrix requests return through a dedicated response path;
* Read hit: after a local hit, the data is returned directly through the
  dedicated bypass;
* Read miss: a coherence read request is issued, and on return the local cache
  line is updated and the data sent back;
* Read-after-write conflict: the RAW bypass pushes the newest in-pipeline data
  directly;
* Write request: write-through, requires waiting for a write-authorisation
  response.

## Operand memory constraints

* The matrix operand buffer must reside in the **physically contiguous memory
  pool** (`/dev/mem` @ `0x100000000`);
* The AMU computes row addresses as `PA_base + row × stride` and **does not
  re-walk the TLB for every row**, so a non-contiguous buffer is silently
  misread once the row count is large (for example 63 rows, or 127 rows and
  above) instead of reporting an error;
* The allocation mode (pool mode / anonymous mode) must be decided by the
  external environment and must not be compiled into the artifact: anonymous
  mode is only used for QEMU user mode, and if it is unconditionally carried
  onto the board, the symptom is `SIGABRT` (`rc=134`), or a wrong result under
  the anonymous pool that is caught by the guard (`rc=255`).

## Visibility and memory ordering

* Ordering constraints between matrix accesses and other memory accesses are
  given by the synchronization instructions and CSR semantics;
* Synchronization with scalar / vector code uniformly uses `mfence`;
* `mrelease` / `macquire` must cover all exit branches, otherwise intermittent
  errors occur (not stably reproducible);
* The behaviour is undefined (UB) when synchronization is missing.

TODO: rule list.

## Interface to the external LLC

* Multiple LLC backends are supported;
* A difference in the handling of matrix responses and ordinary responses once
  caused an accounting desynchronisation problem; after the fix both are unified
  to complete in the same cycle.

TODO: interface parameters and configuration examples.

## Known limitations

* Behavioural and performance differences under trimmed configurations;
* Historically software assumed by default that matrix load/store hit in L2 (a
  workaround for an early defect), so the regular test path never triggers the
  miss path and the miss-related statistics have long been abnormal values.
  Before evaluating miss performance, first verify on a small bare-metal case
  that the miss path itself is correct.
