# FPGA and Cloud Platform {#sec:sim-fpga}

## Local flow

```bash
make fpga-rtl        # generate FPGA-oriented RTL
make fpga-synth      # Vivado project/synthesis
make fpga-bitstream  # generate the bitstream
make fpga-program    # program the board
make fpga-run-bitstream  # program, then run a workload
```

The default FPGA RTL is a single-core `FpgaMinimalMatrix256KConfig`: ZhuJiang
LLC, CHI issue E.b, a 256 KiB eight-bank L2 and 256 KiB LLC, with the
synthesis-oriented VipArbiter patch applied by default.

Frequently used knobs:

| Group | Variables |
| --- | --- |
| RTL generation | `FPGA_RTL_CONFIG`, `FPGA_RTL_NUM_CORES`, `FPGA_RTL_LLC`, `FPGA_RTL_L2_CACHE_SIZE`, `FPGA_RTL_OUT_ROOT` |
| Synthesis | `FPGA_SYNTH_PROJECT`, `FPGA_SYNTH_JOBS`, `FPGA_SYNTH_RUN_TO` |
| ILA | `FPGA_ILA_DEPTH`, `FPGA_ILA_PROFILE(full\|compact\|minimal\|axi\|axi-valid)`, `FPGA_ILA_CAPTURE_TIMEOUT_MS` |
| Run | `FPGA_HOST`, `PAYLOAD`, `FPGA_XDMA_*`, `FPGA_UART_CMD`, `FPGA_TIMEOUT` |

Capturing one ILA window:

```bash
make fpga-insert-zhujiang-ila
make fpga-capture-ila
```

## Cloud platform

The platform provides a GitLab pipeline, board queue and UART console. Prefer the
wrapper scripts over hand-rolled polling:

```bash
# Trigger a run (default bitstream; UART output collected automatically)
bash run-payload.sh <payload.bin> --bitstream default

# Bitstream: keep = reuse what is on the board (fastest) | default | mini (minimalMatrix)
bash run-payload.sh <payload.bin> --bitstream keep

# Interactive: board shell with read/write
bash run-payload.sh <payload.bin> --interactive
```

Watching an existing job:

```bash
python3 watch.py job --job <id> --mode uart|stages|full   # --once for one sample
python3 watch.py status                                   # board/runner status
```

Interpretation:

* `job pending` while another job runs → **normal queueing**, just wait;
* `job running` with a silent UART → usually a quiet compute phase; only a
  build/flash stage with no output for more than 5 minutes is suspicious;
* completion is detected by sentinel strings, so there is no need to wait for
  the window to expire; 600 s without UART output triggers a **stall watchdog**
  that kills the job to free the board.

## Payload preparation

Maintain a set of "verified on board" payloads, each with:

* an explicit completion marker (used by the watchdog and the verdict);
* the expected runtime order of magnitude;
* the required bitstream (`default` / `mini`) and whether interrupts or UART
  input are needed.

## Known limitations

* Minimal-matrix configurations differ in behaviour and performance from full
  configurations; always fix the configuration when comparing.
* Upload bandwidth to the cloud platform is limited; allow transfer time for
  large firmware images.
