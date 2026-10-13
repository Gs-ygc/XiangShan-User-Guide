# Accelerated Simulation (Palladium) {#sec:sim-acceleration}

## Flow overview

RTL synthesis → netlist mapping onto board resources → on-board run. Synthesis
plus gate-level netlist, and netlist mapping/place-and-route, dominate the wall
time (typically 20–30 minutes each), so incremental builds and caching matter.

## Packaging and deployment

```bash
make pldm                 # build and package inside a disposable git worktree
# Optional:
#   PLDM_SKIP_BUILD=1     reuse an existing build/
#   PLDM_KEEP_WORKTREE=1  keep the worktree
#   PLDM_COMPRESS=0       write a plain .tar
```

Deployment and execution happen inside the closed area:

1. Compile the RTL on the open side (with `PLDM=1` and related options, and
   **without difftest** — diff-related comparisons must be done on the open side
   first).
2. Transfer into the closed area over SFTP (use the small-machine gateway when
   crossing the firewall).
3. Activate the PLDM environment inside the closed area and build:

   ```bash
   make pldm-build CONFIG=xxx LLC=OpenLLC FPGA=1 PLDM=1 \
     WITH_CHISELDB=0 WITH_CONSTANTIN=0 DISABLE_DIFFTEST_RAM_DPIC=1 SYNTHESIS=1
   ```

   A successful build prints `FLOW SUCCESS`.
4. Run:

   ```bash
   make pldm-run PLDM_EXTRA_ARGS="+no-diff +workload=/path/to/bin"
   ```

## Access

All access goes through the `open10` gateway:

```bash
pldm doctor                                  # check the link
pldm run --target login01 --follow -- CMD    # run a command
pldm jobs | pldm logs JOB --follow | pldm cancel JOB
```

Note: the home directory on `login01` is `noexec`; invoke scripts as
`/bin/sh <path>`.

## Common issues

| Symptom | Cause and handling |
| --- | --- |
| RTL syntax errors during build | Left-over fragments from Chisel-generated RTL; locate the offending line and clean it up, then retry |
| Run appears stuck | First check whether it still makes progress (see [Troubleshooting](debug-troubleshooting.md#sec:debug-troubleshooting)) |
| Need diff comparison | Diffs must be done on the open side; closed-area builds have difftest disabled |

## Host communication

The link between Palladium and the host can become the bottleneck (measured on
the order of 200 MB/s), which limits the evaluation frequency of differential
testing. Estimate this ceiling before running instruction-by-instruction
comparisons at scale.
