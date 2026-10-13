# Differential Testing {#sec:debug-difftest}

## Role

Difftest compares Xianglufeng execution against a functional reference model
step by step. It is the main guarantee of matrix extension correctness, and the
final arbiter of whether commit semantics follow the specification.

## Reference models

* **CPU reference** and **GPU reference** paths are both supported, allowing
  reference-side implementation differences to be ruled out;
* matrix instructions support batching, backpressure and ordering checks;
* several **undefined behaviours** are turned from "silently pass" into
  "detectable errors": reading unwritten matrix registers, missing `mfence`,
  memory access before `macquire`, and similar cases.

## Matrix register comparison

The earlier implementation transferred every element of the matrix register file
for comparison. It now computes a **hash on each side** and compares hashes:

| Item | Note |
| --- | --- |
| Benefit | Greatly reduces traffic between the emulator and the reference model |
| Applies to | Instructions with a unique result (`mzero`, arithmetic, ...) |
| Does not apply to | Multiplication-class instructions with error tolerance (e.g. `mma`), whose results are not unique and whose reference must keep progressing |
| Equivalence | Equal inputs always yield equal hashes; differing inputs normally yield different hashes |

## Overhead

Difftest is not free; its cost has been quantified on several workloads (same
binary, same seed, interleaved diff / no-diff runs):

| Workload | Configuration | Overhead |
| --- | --- | --- |
| AME-MMACC | CPU pinned (CPU 0–7) | +0.77% |
| CoreMark 5k-cycle smoke | — | +0.91% |
| CoreMark 50k-cycle smoke | — | +1.19% |
| CoreMark full workload | — | +0.30% |

Two methodological rules: (1) large variation measured without CPU pinning
(e.g. 17.10%) reflects scheduling/frequency drift and must not be recorded as
difftest overhead; (2) a comparison is valid only when the number of runs is
correct, both modes are stable and consistent, and instruction and cycle counts
are identical across modes.

## Recommended practice

1. Keep difftest enabled in daily minimal validation (`DIFF=1`) — it is the
   standard path.
2. On mismatch, first re-run with `DIFF=0` to exclude the diff interface itself,
   then decide between RTL and reference models.
3. For matrix registers, prefer instruction ranges already covered by hash
   comparison to avoid needless traffic.
4. Reference and DUT **configurations must match** (e.g. minimal vs full),
   otherwise systematic false positives appear.
