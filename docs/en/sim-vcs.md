# VCS Simulation and Testbench {#sec:sim-vcs}

## Scope

VCS is used mainly for module- and system-level verification of the matrix unit
and L2, for coverage collection, and for cross-checking against Verilator.

## Testbench overview

* **CUTE-Test**: matrix unit test cases with step-by-step comparison against the
  NEMU reference model; `make test` runs the full regression.
* **CUTE-CTRL**: control path cases and debug entry points.
* **TestTopWithL2**: the DUT with L2 and a coherence interface (no-snoop
  subset). A self-built SN functional model replaces the real SN and provides a
  protocol checker (transaction legality and timeout localisation).
* **CUTE-UVM**: hierarchical UVM environment organised in four layers.

| Layer | Typical modules |
| --- | --- |
| 1. Core compute | `FReducePE`, `MatrixTE` and submodules (MAC pipeline, compare tree, scale/vector decoders) |
| 2. Datapath control | `ADataController`, `BDataController`, `CDataController`, scale controllers |
| 3. Memory loading | `AMemoryLoader`, `BMemoryLoader`, `CMemoryLoader`, scale loaders |
| 4. Register files | `ABMatrixReg`, `CMatrixReg`, `ABScaleMatrixReg`, `SRAMTemplate` |

## Dual-engine alignment

The same test set is run on both Verilator and VCS, comparing the **pass/fail
sets** and the **timeout signatures**. Only when both match byte-for-byte can a
failure be attributed to the DUT. When the L2-enabled environment was first
built, both engines reported the same "25 pass / 32 fail" with identical
timeout signatures, which established behavioural equivalence.

## Build notes

1. Get Verilator green first with **ChiselDB / DRAMsim3 disabled**, then move to VCS:

   ```bash
   make -C XSAI emu -j8 CONFIG=DefaultMatrixConfig \
     WITH_DRAMSIM3=0 WITH_CHISELDB=0 WITH_CONSTANTIN=0 \
     EMU_THREADS=8 EMU_TRACE=fst
   ```

2. Update the L2 submodule to a revision containing the known bug fixes (it need
   not track the revision pinned by XSAI).
3. The difftest side needs the C++17 handling in `mma_backend`.

## Coverage

Line/branch coverage collected from Verilator serves as the daily baseline
(e.g. one run reported line 96% / branch 74%, where the shortfall came from
downstream paths of hung cases rather than from uncovered logic). VCS is used for
additional confirmation and timing-related checks.
