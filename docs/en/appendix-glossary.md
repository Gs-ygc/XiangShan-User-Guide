# Appendix B: Glossary {#sec:appendix-glossary}

| Term | Full name | Notes |
| --- | --- | --- |
| AME | Attached Matrix Extension | The attached matrix extension |
| AMU | AME Management Unit | Matrix control path handling memory access and command management for the matrix unit |
| CUTE | — | Matrix compute unit |
| HBL2 | High Bandwidth L2 | High-bandwidth second-level cache |
| LLC | Last Level Cache | Last-level cache |
| MSC | Matrix System Control | Base address for matrix system control; software must pin its base register |
| tile | — | Matrix tile; Xianglufeng implements only the default and mini native geometries |
| difftest | — | Differential testing |
| UB | Undefined Behaviour | Undefined behaviour |
| checkpoint | — | Program-state snapshot used for sliced simulation |
| GCPT | — | Boot image format used for fast startup |
| tail packet | — | Last packet holding fewer pixels than `DST_XUNIT`; stored in a packed layout |
| workload guard | — | `oc × K` threshold deciding whether a task is handed to the matrix unit |
| bit-level identical | — | Criterion that outputs are byte-identical with the matrix path on and off |

TODO: complete the glossary.
