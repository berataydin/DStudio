# Qwen3.8 metadata inspection without PLE preloading

Base: [`ivanfioravanti/ds4-metal`](https://github.com/ivanfioravanti/ds4-metal/tree/bd9cfbccc03a709a3f00b50e0ac1cc41c3fcf02d),
revision `bd9cfbccc03a709a3f00b50e0ac1cc41c3fcf02d`, branch `qwen3.8-flash-next`.
The original source is MIT-licensed; retain its notices. Model terms are separate.
The same patch applies to the new install pin `66b0e3fc3bf0f548db1ec0c0dd19f4e43567a7f8`;
its apply/repeat/check/restore, unrelated-edit, drift, symlink and ABI cases were
rerun on a private copy of the actual fresh installation. The historical native
prefetch measurements below belong to the original base, not a new inference run.

The native `--inspect` path suppresses prefetch of the main weights but previously
requested `POSIX_MADV_WILLNEED` for the entire separately stored PLE. This could
block an inspection on tens of gigabytes of SSD reads. The one-line patch applies
the existing `inspect_only` decision to the PLE mapping too. Metadata parsing,
shape validation and normal inference prefetch are unchanged. The PLE remains
required for this split checkpoint; this does not enable expert streaming.

The installer applies this patch before building Qwen3.8. There are no preceding
DStudio patches on this native branch. For an explicitly selected checkout:

```sh
DS4_DIR=/path/to/ds4-qwen38 sh scripts/apply-ds4-qwen38-inspect.sh check
DS4_DIR=/path/to/ds4-qwen38 sh scripts/apply-ds4-qwen38-inspect.sh apply
DS4_DIR=/path/to/ds4-qwen38 sh scripts/apply-ds4-qwen38-inspect.sh restore
```

Verification:

```sh
node tests/integration/qwen38_inspect_patch_test.mjs ds4-qwen38
node tests/integration/qwen38_inspect_patch_test.mjs ds4-qwen38 --native ds4/gguf
```

The first command uses a private source copy for patch lifecycle tests. The second
also builds the native CLI, uses existing real model/PLE metadata, and on macOS
interposes the OS prefetch call. It proves the original inspection asks to
prefetch the full PLE and the patched inspection does not, with identical output.
It then runs without interposition and checks all reported tensor counts. A
separate controlled startup exits at the first normal CPU prefetch, before
inference. No weights are downloaded and no GPU or numerical quality is tested.
An existing engine holding the instance lock is not stopped by this test.
