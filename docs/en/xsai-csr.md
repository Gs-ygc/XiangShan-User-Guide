# Privileged Modes and Control and Status Registers {#sec:xsai-csr}

This chapter lists only the CSR content that is new or semantically changed
relative to Kunminghu V2R2.

## Matrix extension CSRs

TODO: CSR names, addresses, bit fields and access permissions.

## Specification probe

Before enabling the matrix path, software should read the native tile geometry
of the current hart / bitstream through a CSR and generate kernels only from the
probe result. This is the first step in hooking up the matrix unit, and the only
trustworthy source of the specification: the shape parameters in the
configuration registers are guaranteed correct only within the native geometry.

## Precision capability description

* Matrix precision capabilities are no longer expressed by a read-only bitmap
  (`xmisa`); a per-hart device-tree property `riscv,matrix-isa-caps` describes
  the input / output type combinations instead;
* Rationale: a bitmap cannot express the combinatorial growth of
  "input × input → output" pairs.

TODO: capability string values and a parsing example.

## Extension enable and state

* Enable control for the matrix extension;
* Matrix state that must be saved / restored on a context switch.

TODO: save/restore flow and trap behaviour.

## Interfaces to the OS and firmware

* Firmware performs matrix-related initialisation during boot and fills in the
  capability description;
* The reported capabilities must match the actual RTL configuration, otherwise
  software will generate kernels for the wrong tile specification;
* OS-side requirements for matrix context handling.

TODO: interface conventions.
