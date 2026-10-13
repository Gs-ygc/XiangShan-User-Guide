# Registers {#sec:xsai-registers}

This chapter covers the matrix-related registers added by Xianglufeng. Scalar and
vector registers are identical to Kunminghu V2R2.

## Matrix registers

| Type | Name | Purpose |
| --- | --- | --- |
| A/B tiles | `tr0` – `tr3` | Operands of the matrix multiply-accumulate |
| C/acc tiles | `acc0` – `acc3` | Accumulator and results |

TODO: tile view, bank/slot mapping and element layout.

## Configuration registers

* `mcfg0` – `mcfg7`: accessed by configuration instructions; describe tile shape,
  data type and access mode.

TODO: field-by-field description.

## Synchronization registers

* `sync0` – `sync15`: synchronization and dependency tracking for asynchronous
  matrix work.

TODO: semantics, initialisation and context-switch requirements.

## Layout and alignment constraints

* Element width, tile shape and memory layout constraints;
* Row-major/column-major conventions for transposed access;
* Tail and out-of-range handling.

TODO: diagrams and illegal-configuration behaviour.
