# Modes and CSRs {#sec:xsai-csr}

This chapter lists only the CSR-related content that Xianglufeng adds to or
changes over Kunminghu V2R2.

## Matrix extension CSRs

TODO: CSR names, addresses, bit fields and access permissions.

## Precision capability description

* Read-only capability bitmaps are no longer used for matrix precisions; a
  per-hart device-tree attribute describes the input/output type combinations;
* Rationale: a bitmap cannot express the combinatorial growth of
  "input × input → output" pairs.

TODO: capability string values and parsing example.

## Extension enable and state

* Enable controls for the matrix extension;
* Matrix state that must be saved/restored on context switch.

TODO: save/restore flow and trap behaviour.

## Interfaces to firmware and OS

* Firmware performs matrix-related initialisation during boot;
* OS-side requirements for matrix context handling.

TODO: interface conventions.
