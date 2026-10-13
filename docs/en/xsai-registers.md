# Data Registers {#sec:xsai-registers}

This chapter describes the matrix-related registers added by Xianglufeng. Scalar
and vector registers are identical to Kunminghu V2R2.

## Matrix registers

| Type | Name | Purpose |
| --- | --- | --- |
| A / B tile | `tr0` – `tr3` | Operands of the matrix multiply-accumulate |
| C / acc tile | `acc0` – `acc3` | Accumulation and results |

On both the integer and the bf16 path the `acc` accumulator is fp32 precision;
low-precision inputs do not change the accumulator width.

## Native tile geometry

The matrix unit implements only two native geometries, and software **must use
the specification read by the probe**:

| Configuration | Native geometry (M × N × K) | Notes |
| --- | --- | --- |
| default | 128 × 64 × 128 | Used by the v13-series bitstream |
| mini | 64 × 64 × 64 | Used by the min-v6-series bitstream |

If the specification and the bitstream do not match, the result is guaranteed
to be wrong; a whole layer failing on the board while QEMU is fully correct is
usually exactly this case.

## Configuration registers

* `mcfg0` – `mcfg7`: accessed by configuration instructions; they describe the
  tile shape, the data type and the memory access mode.

### Shape and type parameters

* `msettilem` / `msettilen` / `msettilek`: set the M, N and K dimensions of the
  tile.
* On the bf16 path `msettilek` counts in **elements** (`msettilek=32`), whereas
  the N parameter of the matrix multiply-accumulate instruction counts in
  **bytes** (corresponding to 64 bytes).
* When the K dimension is not an integer multiple of 128, padding or tail
  handling is required.

TODO: bit fields of each field and the behaviour for illegal configurations.

## Synchronization registers

* `sync0` – `sync15`: synchronization and dependency tracking for asynchronous
  matrix operations.

TODO: semantics, initialisation and context-switch requirements.

## Layout and alignment constraints

* The operand buffer must reside in the **physically contiguous memory pool**
  (`/dev/mem` @ `0x100000000`): the AMU computes row addresses as
  `PA_base + row × stride` and does not re-walk the TLB for every row, so a
  non-contiguous buffer is silently misread once the row count is large;
* Transposed memory access follows row-major / column-major conventions,
  controlled by the stride of the memory access instruction;
* When comparing dumps in differential verification, exclude the padding
  introduced by 16 B alignment (uninitialised memory) and compare only the valid
  elements, otherwise false failures appear.

TODO: diagrams and illegal-configuration behaviour.
