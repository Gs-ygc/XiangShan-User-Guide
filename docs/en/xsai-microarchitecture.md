# Microarchitecture Delta {#sec:xsai-microarchitecture}

This chapter covers only what Xianglufeng adds to or changes over the Kunminghu
V2R2 microarchitecture.

## Matrix compute unit

* An independent matrix unit interacting with the scalar core through the matrix
  control path;
* Internally: memory access path, transpose engine, matrix multiply-accumulate
  path and result register file.

TODO: block diagram, pipeline stages, area/frequency figures.

## Control path and instruction commit

* The core performs decode, the necessary integer source reads and zero-cycle
  write-back;
* Commands are packed into control entries and committed by the ROB like normal
  instructions;
* Vector register accesses and RAW/WAW dependencies stay in the out-of-core
  execution unit.

TODO: command format, completion events and exception reporting.

## Clock domains

* The matrix unit and the core may run in different clock domains;
* Commands, completion notifications, matrix data, performance statistics and
  difftest events cross domains through asynchronous FIFOs;
* **Same-clock direct connection is retained** and can be enabled by
  configuration to bypass the asynchronous logic.

TODO: FIFO parameters, recommended clock ratios and performance cost.

## High-bandwidth L2 and external LLC

* HBL2 improves data supply for the matrix unit;
* External LLC backends are supported alongside the matrix access path.

TODO: capacity, bandwidth and port parameters.

## Out-of-core vector / element-wise unit (planned)

* A planned element-wise / reduce unit for softmax, quantization and
  dequantization;
* Intended interface: bundle-based task dispatch reusing an external vector
  substrate.

TODO: the direction is not finalised; pending the dedicated design review.
