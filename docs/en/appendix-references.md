# Appendix C: References and Repositories {#sec:appendix-references}

## Manuals

* _XiangShan Open-Source Processor User Guide_ — the baseline behaviour
* Xianglufeng ISA specifications and proposals (ZAMES / ZMASYNC / low-precision
  convert)

## Repositories

TODO: RTL, compiler, simulators, software stack, kernel library and docs.

## Tools and material

* `xsai-env` documentation and workflows (`docs/`, `AGENTS.md`)
* Environment checks: `make test-smoke` / `make test`
* Reporting: `scripts/bug-report.sh`, `scripts/create-issue.sh`

## Build artifacts and payloads

| Path | Contents |
| --- | --- |
| `XSAI/build/emu` | Verilator emulator |
| `XSAI/build/rtl/SimTop.sv` | Simulation-only RTL top |
| `XSAI/build/rtl/XSTop.sv` | Synthesis / FPGA-style RTL top |
| `XSAI/ready-to-run/*.bin` | Pre-built bare-metal images |
| `firmware/gcpt_restore/build-*/build/gcpt.bin` | Checkpoint restore boot image |
| `firmware/checkpoints/**/*.zstd` | Checkpoint slices |
