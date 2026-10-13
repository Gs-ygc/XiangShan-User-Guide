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

**Log**: `Failed to execute /init (error -2)` followed by `Kernel panic - not syncing`.

**Cause**: `/init` exists in the initramfs, but the loader path referenced by busybox is missing from the image.

**Recovery**: rebuild the rootfs (clean stale artifacts if the environment was switched):

```bash
make -C firmware/riscv-rootfs clean
make firmware
```

## Runtime issues

### Linux boots extremely slowly under RTL

**Symptom**: OpenSBI and early Linux messages look normal, but later boot takes minutes to hours, and guest time advances far slower than host time.

**Cause**: Linux derives scheduler time from `mtime` and the device-tree `timebase-frequency`:

```text
RTL cycles per guest second = RTC divider * timebase-frequency
```

The simulation hardware divides the RTL clock by 100 by default, so a DTB advertising 10 MHz needs one billion cycles per guest second — roughly 46 hours at a 6 kHz emulation rate. The bundled XSAI Linux image and the generated DTS use 1 MHz, while QEMU's `nemu` machine uses 10 MHz, so **a DTB must not be moved directly between QEMU and RTL**. The current FPGA pack flow drives XSTop at 50 MHz with an RTC divider of 100, i.e. 500 kHz.

**Checks**:

```bash
fdtget firmware/nemu_board/dts/build/xiangshan_ai.dtb /cpus timebase-frequency
grep -E 'CONFIG_(HZ|NO_HZ|RISCV_TIMER|CPU_FREQ|RTC_CLASS)' firmware/riscv-linux/.config
```

The kernel boot log line `Platform Timer Device : aclint-mtimer @ ...Hz` must agree with it. Note that enabling an RTC driver or CPUFreq does **not** repair an incorrect `mtime` scale.

### Hangs and stalls

* The generic stall watchdog counts only "completion milestones" (command accepted, release pulse, memory-side completion); its threshold adapts to the number of accepted commands, and on timeout it can emit a deterministic signature that makes reproduction easier to compare;
* Diagnosis order: are instructions still committing → are matrix completion events produced → do memory responses return;
* In the cloud platform, 600 s without UART output marks a job as stalled and terminates it, so the board is not held for a long time.

### Results disagree

1. First rule out **undefined behaviour**: reading matrix register regions that were never written, missing required synchronization, out-of-range reads and writes;
2. Use differential verification to converge on the exact instruction and moment;
3. Confirm that the reference model and the DUT use the **same configuration** (for example, a trimmed configuration against a full one produces systematic false positives).

### Matrix memory-access miss behaviour

Historically, software guaranteed by default that matrix load/store hit L2 (a workaround for an early defect), so the regular test path **never triggers the miss path**, which left miss-related statistics at anomalous values for a long time (for example, all hits). To evaluate miss performance, first validate the miss path itself on a small bare-metal case.

## Cloud platform and CI

| Symptom | Interpretation |
| --- | --- |
| job `pending`, with another job `running` | Normal queueing; no need to re-trigger or cancel |
| job `running`, UART silent for a long time | Usually a compute phase; only more than 5 minutes without output during build/flash is suspicious |
| Widespread CI failures/hangs | Check first whether concurrent usage is the cause; re-run in the queue |

## Frequent failures in software porting (inference frameworks)

Problems that appear after connecting the matrix unit fall into two categories, wrong results and crashes or hangs; look up your symptom in the tables below.

### Wrong results

| Symptom | Root cause | Recovery |
| --- | --- | --- |
| A layer outputs all zeros, the test reports `error=0` | The kernel does `memset(dst,0); return` when `K_pad` exceeds the limit, silently writing zeros | Raise the capacity limit, and let the shape guard fall back to the framework's own kernel |
| Pixel rows are misaligned (row 1 equals row 3) | The accumulator row stride is hard-coded to 64, while `tile_n=128` in practice | Always read the accumulator with `tile_n` |
| Tail packet (`real_count < DST_XUNIT`) gives wrong results | The per-sz stride of A uses a fixed `DST_XUNIT`, while a tail packet actually has a packed layout | Use the pixel count inside the packet, `min(real_count, EX)` |
| A whole layer is wrong on the board but fully correct in QEMU | That bitstream used a non-native tile shape | Validate only the shape selected by the probe; do not run the other one |
| bf16 results differ by 1 ulp and boundary tests hang | Results are not truncated to the bf16 grid | Truncate consistently in the reference implementation and the epilogue |
| attention output is 0 or inf | Q packing and the KV cache are stored as fp32, while the bf16 kernel reads in 2-byte units | Fix Q packing, KV storage, and the byte units of pointers and strides at the same time |
| GEMV / depthwise paths disagree with bf16 semantics | These bypass paths do not follow the low-precision convention | Add the bf16 semantics, or fall back explicitly |

### Crashes and hangs

| Symptom | Root cause | Recovery |
| --- | --- | --- |
| `rc=134` (`SIGABRT`) on the board | Anonymous pool mode is hard-coded into the artifact, and the `/dev/mem` initialisation failure path aborts | Pass pool mode in only from the external environment |
| `rc=255` on the board | A wrong result under the anonymous pool (not physically contiguous) is intercepted by the in-process difftest guard, which returns -1 | Same as above; the board must use the physical pool |
| Results fail occasionally and do not reproduce reliably | `mrelease` / `macquire` do not cover all exit branches, or the MSC base register is reclaimed by the compiler | Pin `s11`, and cover all exit paths with synchronization |
| A framework path raises illegal instruction on that simulator | The framework's built-in fp16 RVV GEMM is incompatible with the simulator version, unrelated to the matrix unit | Turn off that feature switch to work around it |

### Verification false alarms

| Symptom | Root cause | Recovery |
| --- | --- | --- |
| Parsed output is empty | An uninitialised awk variable propagates an empty value into the index (an empty string is printed when argmax lands on 0) | `print idx + 0`; delete stale output files before each run |
| "The model was added but never reached the image" | Only the build log was checked, not the image contents | Unpack and check the image after the build, or verify that the image size changed |
| All tests are green but the conclusion is wrong | The tests exercised a different shape or a different path | Confirm that the selected shape and the path actually exercised agree (print the shape and the call counts) |
| Difftest dump comparison fails spuriously | The comparison read 16 B-aligned padding (uninitialised memory) | Compare valid elements only |
| The self-check is "consistently wrong" | The self-check reference inherited the kernel's wrong layout assumption | Index derivation in the reference implementation must be independent of the kernel |
| The pre-packaged B tile holds stale data | `create_pipeline` runs before the quantization stage | Confirm the order is `load_param → load_model` (quantization) `→ create_pipeline` |

### Build and integration

| Symptom | Root cause | Recovery |
| --- | --- | --- |
| Cross-compiled CMake configure fails | Host x86 options are still enabled, and `CMAKE_MAKE_PROGRAM` is not set | Disable the host ISA options, and set `CMAKE_MAKE_PROGRAM` explicitly |
| Compilation reports that `platform.h` cannot be found | The include path of the build directory is missing | Add `-I<build>/src` |
| `sysroot` becomes `/sysroot` | In the one-liner `export A=x B=$A/y`, `$A` is not yet in effect | Put each `export` on its own line |
| The simulator reports `/lib/ld-linux-*.so.1 not found` | The prefix environment variable was not set | Set `QEMU_LD_PREFIX` explicitly |
| The environment variable is passed `"0"` but is treated as an enabled switch | The check uses `getenv() != NULL`, testing presence but not value | Compare the value instead |
