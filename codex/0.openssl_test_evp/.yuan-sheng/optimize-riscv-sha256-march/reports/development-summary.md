# Development Summary: optimize-riscv-sha256-march

## Input

Agent3 Lite diagnosis source:

`/Users/zq/Desktop/ai-projs/posp/yuan-sheng/ys/blueprint/0.openssl_test_evp/blueprint_openssl_diag-20260612-173745.json`

The Lite report attributed a scalar SHA-256 hotspot to missing RISC-V `v` and `zbb` build flags.

## Agent4 Findings

Current OpenSSL master already has the relevant optimized RISC-V SHA-256 pieces:

- `crypto/sha/build.info` includes `sha_riscv.c`, baseline SHA-256, Zbb SHA-256, and vector SHA-256 sources for `asm_arch => 'riscv64'`.
- `crypto/sha/sha_riscv.c` selects the vector crypto path when `RISCV_HAS_ZVKB()` and `RISCV_HAS_ZVKNHA()` or `RISCV_HAS_ZVKNHB()` are present and VLEN is at least 128.
- `crypto/sha/sha_riscv.c` selects the Zbb path when `RISCV_HAS_ZBB()` is present.
- `crypto/perlasm/riscv.pm` emits RISC-V Zbb/vector instruction encodings directly, so assembler mnemonic support for `-march=..._zbb` is not the only relevant condition.

Because of that, changing the generic RISC-V build target to force `rv64gcv_zbb` would be too broad and likely incorrect for upstream OpenSSL. The minimal safe patch is documentation that shows how to force and benchmark the relevant runtime capability paths using `OPENSSL_riscvcap`.

## Patch

Modified:

- `doc/man3/OPENSSL_riscvcap.pod`

The patch adds SHA-256 benchmark examples for:

- Zbb path: `OPENSSL_riscvcap="rv64gc_zbb"`
- Vector SHA-256 path: `OPENSSL_riscvcap="rv64gc_v_zvkb_zvknha"`

It also states that these overrides are for explicit testing and that normal Linux execution should prefer kernel hardware probing.

## Artifacts

- `.yuan-sheng/optimize-riscv-sha256-march/agent3/root-cause-blueprint.json`
- `.yuan-sheng/optimize-riscv-sha256-march/agent4/patch-plan.json`
- `.yuan-sheng/optimize-riscv-sha256-march/agent4/patch.diff`
- `.yuan-sheng/optimize-riscv-sha256-march/agent4/patch-candidate.json`
- `.yuan-sheng/optimize-riscv-sha256-march/agent4/patch-regression-result.json`
- `.yuan-sheng/optimize-riscv-sha256-march/agent5/patch.diff`
- `.yuan-sheng/optimize-riscv-sha256-march/agent5/verified-patch-package.json`
- `.yuan-sheng/optimize-riscv-sha256-march/reports/test-report.md`

## Verification State

The final package schema validates, but its verification status is failed because `agent1 patch_regression --case openssl_test_evp` and RISC-V hardware performance checks could not be run locally.
