# Microarchitecture Delta {#sec:xsai-microarchitecture}

This chapter describes only what Xianglufeng adds to or modifies over the
Kunminghu V2R2 microarchitecture.

## Matrix compute unit (CUTE)

* An independent matrix compute unit interacting with the scalar core through
  the matrix control path;
* Internally it contains the memory access path (AMU), the transpose engine, the
  matrix multiply-accumulate path and the result register file;
* It implements only two native tile geometries: `128×64×128` (default) and
  `64×64×64` (mini).

TODO: block diagrams, pipeline stages and the basis for the area / frequency
figures.

## Control path and instruction commit

* The core performs instruction decode, the necessary integer source register
  reads and zero-cycle write-back;
* A command is loaded into a control entry and then committed by the ROB like an
  ordinary instruction;
* Vector register reads and writes and RAW / WAW dependencies remain tracked by
  the out-of-core execution unit;
* Matrix instructions execute asynchronously: the commit order and the
  completion order may differ, and the order is constrained by the
  synchronization instructions.

TODO: command format, completion events and the exception reporting path.

## Clock domains

* The matrix unit (`acceleratorClock`) and the core may run in different clock
  domains;
* Commands, completion notifications, matrix data, performance statistics and
  difftest events each cross the domains through their own asynchronous FIFOs;
* **Same-clock direct connection is retained**, and the asynchronous logic can
  be disabled as a configuration option.

TODO: cross-domain FIFO parameters, recommended clock ratios and the performance
cost.

## High-bandwidth L2 and external LLC

* HBL2 improves the data supply for the matrix unit;
* External LLC backends are supported, working together with the matrix memory
  access path.

TODO: capacity / bandwidth / port parameters.

## Out-of-core vector and element-wise unit (planned)

* A planned out-of-core element-wise / reduce unit, targeting operations such as
  softmax and quantization / dequantization;
* Intended interface: packaged and driven in the same manner as matrix-like
  tasks, reusing the arithmetic and the organisation of an external vector
  substrate.

TODO: the direction is not yet finalised; pending the conclusion of a dedicated
discussion.
