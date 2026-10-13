# Debug Support {#sec:xsai-debug}

## Differential testing support

* Matrix instruction diff comparison, including matrix register hash comparison;
* Both CPU and GPU reference models are supported.

See [Differential Testing](debug-difftest.md#sec:debug-difftest).

## Checkpoint and sampling

* `CHECKPOINT_HERE` semantic sampling points;
* QEMU checkpoint and RTL restore flow.

See [Checkpoint and Slicing](debug-checkpoint.md#sec:debug-checkpoint).

## Debug interfaces

TODO: differences from the baseline debug interfaces (breakpoints, single step,
consistency with asynchronous matrix state).

## Known issues

* A vector instruction hangs in a specific scenario under the minimal
  configuration (not reproducible in the full configuration); attribution and
  upstream reporting are pending.

TODO: conclusion.
