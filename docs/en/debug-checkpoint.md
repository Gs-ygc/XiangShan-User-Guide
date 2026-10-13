# Checkpoint and Slicing {#sec:debug-checkpoint}

## Why slice

Running a full workload in RTL is expensive (a single-layer checkpoint can take
more than 20 million cycles, i.e. 5–6 hours). Slicing the region of interest is
what brings simulation cost back to an iterable scale.

## Validation gradient

The default gradient for software issues is fast-to-slow:

```bash
make run-qemu                                  # fastest: sort out software and boot-path issues
make run-nemu                                  # next: also the golden-model middle layer
make ckpt                                      # create a checkpoint
make run-nemu PAYLOAD=<checkpoint.zstd>        # validate the checkpoint in NEMU
make run-emu  PAYLOAD=<checkpoint.zstd>        # finally run in RTL
```

## Workload-side contract

The region of interest is declared by the workload itself through
`nemu_signal(...)`, while outer scripts choose the checkpoint flow:

| Signal | Meaning |
| --- | --- |
| `DISABLE_TIME_INTR` | Prepare for sampling: disable timer interrupts |
| `NOTIFY_PROFILER` | Enter the region of interest |
| `NOTIFY_PROFILE_EXIT` | Leave the region of interest |
| `GOOD_TRAP` | Terminate cleanly |

A minimal skeleton brackets the region with these signals, after which
`make profile`, `make cluster` and `make ckpt` can operate on it.

## CHECKPOINT_HERE

`CHECKPOINT_HERE` samples **exactly where** `nemu_signal(CHECKPOINT_HERE)` is
inserted instead of waiting for the end of a basic block:

* the checkpoint carries accurate workload semantics (no need to guess which
  software region a sample belongs to);
* graph node names are made unique automatically and the sampling point is given
  as a run-time argument, so no code change is required;
* repeated insertions and switching between operator combinations are easy.

## Model packing and system-level flows

* Large models can be packed into `gcpt.bin` and reused across functional
  simulators and RTL, avoiding repeated weight loading;
* workloads requiring Linux can use an OS image together with the checkpoint
  restore flow;
* the restore path historically hit infinite loops and illegal instructions;
  the fixes are merged — confirm the revision in use if you see anomalies.

## Cautions

* QEMU and RTL assume different `timebase-frequency` values, so DTBs are not
  interchangeable (see [Troubleshooting](debug-troubleshooting.md#sec:debug-troubleshooting));
* finer slices iterate faster, but verify that cache state at the slice start
  matches the full run, otherwise performance conclusions drift.
