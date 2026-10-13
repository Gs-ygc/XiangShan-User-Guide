# Exceptions and Interrupts {#sec:xsai-exception}

This chapter documents the exception and interrupt behaviour related to the
matrix extension; for the general mechanisms see the _XiangShan Open-Source
Processor User Guide_ (Exceptions and Interrupts chapter).

## New exception conditions

* Illegal instruction: matrix instructions when the extension is not enabled or
  the configuration is illegal;
* Alignment and memory-access faults: the address and stride constraints of
  matrix memory accesses;
* Undefined behaviour (UB): reading an unwritten matrix register region and
  similar cases, which functional simulators and difftest turn into detectable
  errors.

TODO: add the exception codes and the register content conventions.

## Interaction of long instructions and interrupts

* Matrix operations execute asynchronously, with commit separated from
  completion;
* Historical defects existed in scenarios such as an interrupt occurring during
  a configuration update; for the fixes see the ISA proposals and the RTL commit
  records.

TODO: add the precise exception and restartable semantics.

## Synchronization and the exit path

* Synchronization with scalar / vector is done uniformly with `mfence`;
* `mrelease` / `macquire` must cover **all** exit branches, including exception
  and early-return paths; when they are missed the symptom is intermittent
  errors that are hard to reproduce reliably;
* The consistency requirements between the in-flight state of the matrix unit
  and the software-visible state when an exception occurs.

TODO: add the state-cleanup conventions for exception paths.

## Debug-related

TODO: add the state consistency requirements related to checkpoint / restore.
