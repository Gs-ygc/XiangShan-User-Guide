# Exceptions and Interrupts {#sec:xsai-exception}

This chapter covers exception and interrupt behaviour related to the matrix
extension; general mechanisms are described in the _XiangShan User Guide_.

## New exception conditions

* Illegal instruction: matrix instructions when the extension is disabled or the
  configuration is invalid;
* Alignment and access faults: address and stride constraints of matrix accesses;
* Undefined behaviour: reading unwritten matrix registers, which functional
  simulators and difftest turn into detectable errors.

TODO: exception codes and register content conventions.

## Long instructions and interrupts

* Matrix operations execute asynchronously; commit and completion are decoupled;
* Historical defects existed around interrupts during configuration updates; see
  the ISA proposals and RTL commits for the fixes.

TODO: precise exception and restartable semantics.

## Debug-related

TODO: state consistency requirements for checkpoint/restore.
