# 开发总结报告

Change: `document-riscv-aes-capability-overrides`

## 需求概述

Run the ysclaw-agent4 flow for Agent3 diagnosis `diag-20260612-174210`, which reported `openssl_test_bio_enc` spending its RISC-V profile in scalar `AES_encrypt`. The original diagnosis blocked automatic optimization because SG2044 crypto extension availability was not confirmed.

## 实现变更

- Normalized the Agent3 Lite report into `.yuan-sheng/document-riscv-aes-capability-overrides/agent3/root-cause-blueprint.json`.
- Created OpenSpec and Superpowers design and plan artifacts for the change.
- Generated a schema-valid PatchPlan from the canonical RootCauseBlueprint.
- Updated `doc/man3/OPENSSL_riscvcap.pod` with AES-specific RISC-V scalar crypto, vector AES, and vector AES-GCM override examples.
- Kept the implementation behavior-neutral: no provider dispatch, build configuration, assembler, BIO, or cipher code was changed.

## 产物路径

- RootCauseBlueprint: `.yuan-sheng/document-riscv-aes-capability-overrides/agent3/root-cause-blueprint.json`
- PatchPlan: `.yuan-sheng/document-riscv-aes-capability-overrides/agent4/patch-plan.json`
- PatchCandidate: `.yuan-sheng/document-riscv-aes-capability-overrides/agent4/patch-candidate.json`
- PatchRegressionResult: `.yuan-sheng/document-riscv-aes-capability-overrides/agent4/patch-regression-result.json`
- Agent5 patch: `.yuan-sheng/document-riscv-aes-capability-overrides/agent5/patch.diff`
- VerifiedPatchPackage: `.yuan-sheng/document-riscv-aes-capability-overrides/agent5/verified-patch-package.json`
- Test report: `.yuan-sheng/document-riscv-aes-capability-overrides/reports/test-report.md`
- OpenSpec change: `openspec/changes/document-riscv-aes-capability-overrides/`
- Superpowers design: `docs/superpowers/specs/2026-06-12-riscv-aes-capability-overrides-design.md`
- Superpowers plan: `docs/superpowers/plans/2026-06-12-riscv-aes-capability-overrides.md`

## 验证结果

- JSON schema validation: all generated Agent4 and Agent5 JSON artifacts validate.
- OpenSpec validation: `openspec validate document-riscv-aes-capability-overrides` passes.
- Local documentation verification: `podchecker doc/man3/OPENSSL_riscvcap.pod` passes.
- Source consistency verification: RISC-V AES symbols and new documentation examples are present.
- Agent1 regression: failed to execute because `agent1` is not available on PATH.
- Final package verification state: `failed`.

## 风险与后续

- This package is not a verified performance fix. It is a behavior-neutral documentation patch plus a complete Agent4 artifact chain.
- To promote the package to verified, rerun `agent1 patch_regression --case openssl_test_bio_enc` in an environment with Agent1 and the required OpenSSL test target.
- On SG2044, confirm hardware and runtime capability state with `/proc/cpuinfo`, `riscv_hwprobe`, `readelf -A libcrypto.so.4`, and `./apps/openssl info -cpusettings`.
- Re-profile `test_bio_enc` to confirm whether execution reaches `rv64i_zkne_*` or `rv64i_zvkned_*` instead of scalar `AES_encrypt`.
