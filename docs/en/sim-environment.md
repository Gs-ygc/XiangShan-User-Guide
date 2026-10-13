# Environment Setup {#sec:sim-environment}

## Overview of xsai-env

`xsai-env` is the integrated development workspace for Xianglufeng. The top-level
`Makefile` orchestrates everything; the actual RTL project lives in `XSAI/`, the
functional simulators in `NEMU/` and `QEMU/`, and firmware/rootfs in `firmware/`.

```bash
git clone <xsai-env> ~/xsai-env
cd ~/xsai-env
```

## First-time setup

```bash
make deps         # install system dependencies (requires sudo)
make init         # initialize submodules and environment
make init-force   # recommended before the first heavy build
source env.sh     # or: direnv allow
```

Notes:

* `RISCV`, `RISCV_SYSROOT` and `QEMU_LD_PREFIX` are auto-detected; put machine
  specific overrides in `.envrc.local`.
* The top level intentionally does **not** export `CROSS_COMPILE`; firmware and
  software flows set their own toolchain prefixes.
* Nix is optional. Use `make nix-shell` / `nix-init` / `nix-smoke` only to
  reproduce Nix-specific issues.

## Generated directories

`local/`, `build/`, `log/` and `firmware/checkpoints/` are generated artifacts
and are not tracked.

## Common targets

| Target | Purpose |
| --- | --- |
| `make xsai` | Build the Verilator emulator `XSAI/build/emu` |
| `make run-emu` | Run Verilator simulation (see [Verilator Simulation](sim-verilator.md#sec:sim-verilator)) |
| `make run-nemu` | Run NEMU |
| `make run-qemu` | Run QEMU (optional `MODEL_IMG`) |
| `make firmware` | Build firmware and rootfs |
| `make pldm` | Package a Palladium build (see [Accelerated Simulation](sim-acceleration.md#sec:sim-acceleration)) |
| `make run-fpga` | Deploy and run on FPGA (see [FPGA and Cloud Platform](sim-fpga.md#sec:sim-fpga)) |
| `make test-smoke` | Fast environment sanity check |
| `make test` | Full environment sanity check (slower) |

Run `make help` for the complete list.

## Sanity checks

1. Start with `make help` and `make -n <target>` to confirm the command path.
   Note that `run-*` targets contain recursive `$(MAKE)`, so `-n` is not
   guaranteed to be side-effect free.
2. Use `make test-smoke` for fast validation.
3. Avoid full RTL rebuilds unless required.

## Compute resources

RTL simulation is CPU heavy. Run it on an online, low-load server reported by
the monitoring API, and size concurrency by **physical cores** (one emulation
worker occupies 8 physical cores; the upper bound is physical cores / 8). Keep
watching the monitor while long simulations run.

## See also

* Simulation: [Verilator](sim-verilator.md#sec:sim-verilator), [VCS](sim-vcs.md#sec:sim-vcs),
  [Accelerated](sim-acceleration.md#sec:sim-acceleration), [FPGA](sim-fpga.md#sec:sim-fpga)
* Troubleshooting: [Troubleshooting](debug-troubleshooting.md#sec:debug-troubleshooting)
