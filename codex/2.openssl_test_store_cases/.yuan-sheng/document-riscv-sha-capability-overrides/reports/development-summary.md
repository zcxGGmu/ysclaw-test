# 开发总结报告

Change: `document-riscv-sha-capability-overrides`

## 需求概述

Run the ysclaw-agent4 flow for Agent3 diagnosis `diag-20260612-180925`, which reported `openssl_test_store_cases` spending its RISC-V profile in scalar `SHA1_Final`. The original diagnosis blocked automatic optimization because build ISA, hardware extensions, and focused `sha1_block_data_order` evidence were missing.

## 实现变更

- Normalized the Agent3 Lite report into `.yuan-sheng/document-riscv-sha-capability-overrides/agent3/root-cause-blueprint.json`.
- Created OpenSpec and Superpowers design and plan artifacts for the change.
- Generated a schema-valid PatchPlan from the canonical RootCauseBlueprint.
- Updated `doc/man3/OPENSSL_riscvcap.pod` with SHA-specific RISC-V Zbb, vector SHA-256, and vector SHA-512 override examples.
- Documented that `OPENSSL_riscvcap` affects runtime-dispatched implementations and does not enable compile-time `__riscv_zbb` generation.
- Documented that current RISC-V SHA crypto dispatch covers SHA-2 paths and does not add a SHA-1 crypto dispatch path.
- Kept the implementation behavior-neutral: no digest implementation, provider dispatch, build configuration, assembler, or test recipe was changed.

## 产物路径

- RootCauseBlueprint: `.yuan-sheng/document-riscv-sha-capability-overrides/agent3/root-cause-blueprint.json`
- PatchPlan: `.yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch-plan.json`
- PatchCandidate: `.yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch-candidate.json`
- PatchRegressionResult: `.yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch-regression-result.json`
- Agent5 patch: `.yuan-sheng/document-riscv-sha-capability-overrides/agent5/patch.diff`
- VerifiedPatchPackage: `.yuan-sheng/document-riscv-sha-capability-overrides/agent5/verified-patch-package.json`
- Test report: `.yuan-sheng/document-riscv-sha-capability-overrides/reports/test-report.md`
- OpenSpec change: `openspec/changes/document-riscv-sha-capability-overrides/`
- Superpowers design: `docs/superpowers/specs/2026-06-12-riscv-sha-capability-overrides-design.md`
- Superpowers plan: `docs/superpowers/plans/2026-06-12-riscv-sha-capability-overrides.md`

## 验证结果

- JSON schema validation: all generated Agent4 and Agent5 JSON artifacts validate.
- OpenSpec validation: `PATH=/Users/zq/Desktop/ai-projs/posp/yuan-sheng/opencode-agent4/node_modules/.bin:$PATH openspec validate document-riscv-sha-capability-overrides` passes.
- Local documentation verification: `podchecker doc/man3/OPENSSL_riscvcap.pod` passes.
- Whitespace verification: `git diff --check` passes.
- Source consistency verification: RISC-V SHA-2 symbols and new documentation examples are present.
- Agent1 regression: failed to execute because `agent1` is not available on PATH.
- Final package verification state: `failed`.

## 风险与后续

- This package is not a verified performance fix. It is a behavior-neutral documentation patch plus a complete Agent4 artifact chain.
- The Comet design handoff is a design-phase frozen context package; the current implementation state is represented by the checked `tasks.md`, reports, PatchCandidate, PatchRegressionResult, and VerifiedPatchPackage.
- To promote the package to verified, rerun `agent1 patch_regression --case openssl_test_store_cases` in an environment with Agent1 and the required OpenSSL test target.
- On SG2044, confirm hardware and build state with `/proc/cpuinfo`, `riscv_hwprobe`, `readelf -A libcrypto.so.4`, and `./apps/openssl info -cpusettings`.
- Re-profile `SHA1_Final` and `sha1_block_data_order` before deciding whether a RISC-V SHA-1 implementation is justified.
