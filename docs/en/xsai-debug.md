# Debug Support {#sec:xsai-debug}

## Differential verification support

* Differential comparison of matrix instructions, including matrix register hash
  comparison;
* Both paths are supported: the CPU reference model and the GPU reference model.

See [Differential Testing](debug-difftest.md#sec:debug-difftest).

## Checkpoint and sampling

* `CHECKPOINT_HERE` semantic sampling points;
* The QEMU checkpoint and RTL restore flow.

See [Checkpoint and Slicing](debug-checkpoint.md#sec:debug-checkpoint).

## Debug interfaces

TODO: add the differences from the baseline debug interfaces (breakpoints, single
step, consistency with the asynchronous matrix state).

## Known issues

* Under a cut-down configuration a vector instruction hangs in a specific
  scenario (not reproducible in the full configuration); attribution and
  upstream reporting are pending.

TODO: add the conclusion.

## Troubleshooting entry points

* For high-frequency failures in build, run, cloud platform and software
  adaptation see [Troubleshooting](debug-troubleshooting.md#sec:debug-troubleshooting);
* When results disagree, first rule out undefined behaviour (reading an unwritten
  matrix register region, missing synchronization, out-of-bounds reads and
  writes), then narrow down to the specific instruction and moment.
