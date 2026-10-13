# Troubleshooting {#sec:debug-troubleshooting}

## Build issues

### Mixed Nix / non-Nix rootfs artifacts

**Log**: `error: patchelf is required to rewrite interpreter /nix/store/...`

**Cause**: artifacts from an old Nix build were reused in a non-Nix build.

**Recovery**:

```bash
source env.sh
make -C firmware/riscv-rootfs clean
make firmware
```

### Kernel panic: No working init found

**Log**: `Failed to execute /init (error -2)` followed by
`Kernel panic - not syncing`.

**Cause**: `/init` exists in the initramfs, but the loader path referenced by
busybox is missing from the image.

**Recovery**: rebuild the rootfs (clean stale artifacts if environments changed):

```bash
make -C firmware/riscv-rootfs clean
make firmware
```

## Runtime issues

### Linux boots extremely slowly under RTL

**Symptom**: OpenSBI and early Linux messages look normal, but later boot takes
minutes to hours, and guest time advances far slower than host time.

**Cause**: Linux derives scheduler time from `mtime` and the device-tree
`timebase-frequency`:

```text
RTL cycles per guest second = RTC divider * timebase-frequency
```

The simulation hardware divides the RTL clock by 100 by default, so a DTB
advertising 10 MHz needs 1e9 RTL cycles per guest second — roughly 46 hours at a
6 kHz emulation rate. The bundled XSAI Linux image and generated DTS use 1 MHz,
while QEMU's `nemu` machine uses 10 MHz, so **a DTB must not be moved between
QEMU and RTL without checking this value**. The current FPGA pack drives XSTop at
50 MHz with an RTC divider of 100, i.e. 500 kHz.

**Checks**:

```bash
fdtget firmware/nemu_board/dts/build/xiangshan_ai.dtb /cpus timebase-frequency
grep -E 'CONFIG_(HZ|NO_HZ|RISCV_TIMER|CPU_FREQ|RTC_CLASS)' firmware/riscv-linux/.config
```

The boot log line `Platform Timer Device : aclint-mtimer @ ...Hz` must agree.
Note that enabling an RTC driver or CPUFreq does **not** repair an incorrect
`mtime` scale.

### Hangs and stalls

* The generic stall watchdog counts only completion milestones (command
  accepted, release pulse, memory-side completion) with a threshold that adapts
  to the number of accepted commands, and can emit a deterministic signature for
  comparison;
* Diagnosis order: is the core still committing instructions → are matrix
  completion events produced → do memory responses return;
* In the cloud platform, 600 s without UART output marks a job as stalled and
  terminates it so the board is not held indefinitely.

### Results disagree

1. First exclude **undefined behaviour**: reading unwritten matrix registers,
   missing synchronization, out-of-range accesses;
2. Use difftest to localise the exact instruction and moment;
3. Verify that reference and DUT use the **same configuration** (a minimal DUT
   against a full reference produces systematic false positives).

### Matrix memory-access miss behaviour

Historically, software guaranteed that matrix load/store hit L2 (a workaround
for an early defect), so the usual test path **never exercised the miss path**,
leaving miss statistics at implausible values (e.g. all hits). Before evaluating
miss performance, validate the miss path itself on a small bare-metal case.

## Cloud platform and CI

| Symptom | Interpretation |
| --- | --- |
| `job pending` while another job is `running` | Normal queueing; do not re-trigger or cancel |
| `job running` with a long silent UART | Usually a compute phase; only a build/flash stage silent for >5 min is suspicious |
| Widespread CI failures/hangs | Check for concurrent usage first, then re-run in the queue |
