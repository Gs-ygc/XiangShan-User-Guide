# Instruction Set Extensions {#sec:xsai-instruction-set}

This chapter documents the instruction set delta over Kunminghu V2R2. For the
baseline extension list see the _XiangShan Open-Source Processor User Guide_.

## Extensions and versions

| Extension | Description | Specification |
| --- | --- | --- |
| ZAMES | Matrix extension: configuration, MMA, matrix memory access, synchronization | proposal-14 and later |
| ZMASYNC | Synchronization companion extension of ZAMES | as above |
| Low-precision convert | `vfncvtmxfp.f.f.w`, `vfncvtxx8.f.a.w`, ... | see the multi-precision proposals |

TODO: mapping between proposal versions and compiler support.

## Configuration and synchronization

* `msettilem` / `msettilen` / `msettilek`: set tile shape
* `msetcfg` / `mgetcfg`: configuration register access
* `mzero`: clear matrix registers
* `mrelease` / `macquire`: release and acquire asynchronous matrix work

TODO: encodings, operand constraints and side effects.

## Matrix multiply-accumulate

| Family | Input | Output | Notes |
| --- | --- | --- | --- |
| `mmacc` integer | i8 × i8, i2 × i8 | i32 | see the low-precision chapter for asymmetric widths |
| `mmacc` float | bf16 × bf16, f16 × f16 | fp32 | |
| Low precision | fp8 / fp4 families | fp32 | requires pack/unpack |

TODO: full instruction table and per-configuration support matrix.

## Matrix memory access

* Block and sub-block load/store by tile shape;
* Transposed load/store with row-major access and stride control;
* Fine-grained access (arbitrary sizes, 8/16/32-bit elements, tail masking);
* Multi-channel access and bank assignment.

TODO: alignment requirements, stride constraints and tail handling.

## Asynchronous execution and ordering

* Matrix instructions execute asynchronously: commit order may differ from
  completion order;
* Ordering with scalar/vector instructions is constrained by synchronization
  instructions and CSR semantics;
* **Reading unwritten matrix registers is undefined behaviour**, which
  functional simulators and difftest turn into detectable errors.

TODO: synchronization mechanism and memory-ordering requirements.

## Low-precision convert instructions

* Defined in the custom opcode space, with scaling factors and mask encodings.

TODO: encoding details and usage.
