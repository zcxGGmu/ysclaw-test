# 开发总结报告

Change: `document-riscv-sha2-slh-dsa-dispatch`

## 需求概述

Run the ysclaw-agent4 flow for Agent3 diagnosis `diag-20260612-180518`, which reported `openssl_test_slh_dsa` spending its RISC-V profile in scalar `sha256_block_data_order_riscv64`. The original diagnosis recommended forcing `-march=rv64gc_zba_zbb` for `linux64-riscv64`, but blocked automatic optimization because ARM baseline, direct binary ISA evidence, and target hwprobe evidence were missing.

## 实现变更

- Normalized the Agent3 Lite report into `.yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/agent3/root-cause-blueprint.json`.
- Created OpenSpec and Superpowers design and plan artifacts for the change.
- Generated a schema-valid PatchPlan from the canonical RootCauseBlueprint.
- Updated `doc/man3/OPENSSL_riscvcap.pod` with SHA-specific RISC-V Zbb, vector SHA-256, and vector SHA-512 override examples.
- Documented that `OPENSSL_riscvcap` affects runtime-dispatched implementations and does not change compiler-selected architecture flags, the default `linux64-riscv64` target baseline, or compile-time `__riscv_zbb` generation.
- Kept the implementation behavior-neutral: no Configure target, digest implementation, generated assembly, provider, FIPS, SLH-DSA, or test recipe was changed.

## 产物路径

- RootCauseBlueprint: `.yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/agent3/root-cause-blueprint.json`
- PatchPlan: `.yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/agent4/patch-plan.json`
- PatchCandidate: `.yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/agent4/patch-candidate.json`
- PatchRegressionResult: `.yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/agent4/patch-regression-result.json`
- Agent5 patch: `.yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/agent5/patch.diff`
- VerifiedPatchPackage: `.yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/agent5/verified-patch-package.json`
- Test report: `.yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/reports/test-report.md`
- OpenSpec change: `openspec/changes/document-riscv-sha2-slh-dsa-dispatch/`
- Superpowers design: `docs/superpowers/specs/2026-06-12-riscv-sha2-slh-dsa-dispatch-design.md`
- Superpowers plan: `docs/superpowers/plans/2026-06-12-riscv-sha2-slh-dsa-dispatch.md`

## 验证结果

- JSON schema validation: all generated Agent4 and Agent5 JSON artifacts validate.
- OpenSpec validation: `PATH=/Users/zq/Desktop/ai-projs/posp/yuan-sheng/opencode-agent4/node_modules/.bin:$PATH openspec validate document-riscv-sha2-slh-dsa-dispatch` passes.
- Local documentation verification: `podchecker doc/man3/OPENSSL_riscvcap.pod` passes.
- Whitespace verification: `git diff --check` passes.
- Source consistency verification: RISC-V SHA-2 symbols, new documentation examples, and `linux64-riscv64` target context are present.
- Local OpenSSL tests were not run because this sparse worktree has no configured build tree, `Makefile`, `configdata.pm`, or `test/slh_dsa_test` binary.
- Agent1 regression: failed to execute because `agent1` is not available on PATH.
- Final package verification state: `failed`.

## 风险与后续

- This package is not a verified performance fix. It is a behavior-neutral documentation patch plus a complete Agent4 artifact chain.
- To promote the package to verified, rerun `agent1 patch_regression --case openssl_test_slh_dsa` in an environment with Agent1 and the required OpenSSL test target.
- On SG2044, confirm hardware and build state with `/proc/cpuinfo`, `riscv_hwprobe`, `readelf -A providers/fips.so`, and `./apps/openssl info -cpusettings`.
- Re-profile `sha256_block_data_order_riscv64`, `sha256_block_data_order_zbb`, and `sha256_block_data_order_zvkb_zvknha_or_zvknhb` before considering any RISC-V SHA-256 source or Configure target change.
