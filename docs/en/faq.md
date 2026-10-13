# FAQ and Known Limitations {#sec:faq}

## Known limitations

* Minimal and full configurations differ in behaviour and performance; always
  fix the configuration when comparing;
* The matrix memory-miss path was not exercised by software for a long time, so
  verification is still being extended;
* Some low-precision formats and custom instructions are still in progress;
* A vector instruction hangs in a specific scenario under the minimal
  configuration but not under the full configuration.

## Simulation

* Linux boots extremely slowly under RTL → check `timebase-frequency` (see
  [Troubleshooting](debug-troubleshooting.md#sec:debug-troubleshooting));
* A cloud job stays `pending` for a long time → usually normal queueing; do not
  re-trigger;
* No waveform is produced → confirm `EMU_TRACE` was enabled at compile time.

## Build

* patchelf errors from mixing Nix and non-Nix rootfs artifacts → clean and
  rebuild;
* Kernel panic `No working init found` → rebuild the rootfs.

## Relation to the baseline manual

This manual documents the delta only. Anything not listed here matches
Kunminghu V2R2 by default; see the _XiangShan Open-Source Processor User Guide_.
