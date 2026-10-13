# Instruction Set Extensions {#sec:xsai-instruction-set}

This chapter documents the instruction set delta of Xianglufeng over Kunminghu
V2R2. For the baseline extension list see the _XiangShan Open-Source Processor
User Guide_ (Instruction Set chapter).

## Extensions and versions

| Extension | Description | Specification |
| --- | --- | --- |
| ZAMES | Matrix extension (configuration, matrix multiply-accumulate, matrix memory access, synchronization) | proposal-14 and later |
| ZMASYNC | Synchronization companion extension of ZAMES | as above |
| Low-precision convert | `vfncvtmxfp.f.f.w`, `vfncvtxx8.f.a.w`, ... | see the multi-precision proposals |

TODO: add the mapping between each version number and the supported compiler
version.

## Configuration and synchronization

* `msettilem` / `msettilen` / `msettilek`: set the tile shape
* `msetcfg` / `mgetcfg`: configuration register read and write
* `mzero`: clear matrix registers
* `mrelease` / `macquire`: release and acquire asynchronous matrix operations

Usage constraints:

* The MSC base address register must be pinned (for example `s11`) and kept
  alive before `macquire`; if the compiler reclaims that register, the result is
  intermittent errors;
* `msyncregreset` before each tile is used;
* Synchronization with scalar / vector is done uniformly with `mfence`;
* `mrelease` / `macquire` must cover **all** exit branches.

TODO: add the encoding, operand constraints and side effects of each instruction.

## Matrix multiply-accumulate

| Family | Input precision | Output precision | Notes |
| --- | --- | --- | --- |
| `mmacc` integer | i8 × i8, i2 × i8 | i32 | for asymmetric bit widths see the low-precision chapter |
| `mmacc` floating point | bf16 × bf16, f16 × f16 | fp32 | the accumulator is always fp32 |
| Low precision | fp8 / fp4 families | fp32 | requires pack / unpack |

On the bf16 path `msettilek` counts by element (`msettilek=32`), whereas the N
parameter of the matrix multiply-accumulate instructions counts by byte
(corresponding to 64 bytes). The bf16 and int8 switches must be separate: bf16
changes the floating-point precision, and sharing a switch makes the official
fp32 tests fail on tolerance.

TODO: add the complete instruction table and the support matrix of each
configuration.

## Matrix memory access

* Whole-block and sub-block load / store (by tile shape);
* Transposed load / store (row-major access, stride control);
* Fine-grained memory access (arbitrary sizes, 8 / 16 / 32-bit elements, tail
  masking);
* Multi-channel memory access and bank assignment.

Constraints:

* The operand buffer must be physically contiguous (see [Memory Subsystem and HBL2](xsai-memory.md#sec:xsai-memory));
* The row address is given by `PA_base + row × stride`, with no per-row TLB
  rewalk;
* The tail packet (`real_count < DST_XUNIT`) is compactly laid out and cannot
  reuse a fixed stride.

TODO: add the alignment requirements and the tail handling rules.

## Asynchronous execution and ordering

* Matrix instructions execute asynchronously: the commit order and the
  completion order may differ;
* Ordering with scalar / vector instructions is constrained by synchronization
  instructions and CSR semantics;
* **Reading an unwritten matrix register region is undefined behaviour (UB)**,
  and functional simulators and difftest turn it into a detectable error.

TODO: add the synchronization mechanism and the memory-ordering requirements.

## Low-precision convert instructions

* Defined in the custom opcode space, supporting elements such as scaling
  factors and mask encodings.

TODO: add the encoding details and usage.
