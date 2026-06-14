# OpenSSL Partial Clone Setup

- [x] Confirm target path and existing directories
- [x] Clone fork with `--depth 1 --filter=blob:none`
- [x] Add official OpenSSL repository as `upstream`
- [x] Create a topic branch from `upstream/master`
- [x] Verify remotes, branch, and partial clone filter

## Review

- Created `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl` with shallow sparse partial clone.
- Current branch is `codex/openssl-pr-work`, tracking `upstream/master`.
- `origin` points to `git@github.com:zcxGGmu/openssl.git`; `upstream` points to `git@github.com:openssl/openssl.git`.
- Both fetch remotes use `blob:none`; repository is shallow and sparse checkout is enabled.
- Final checkout size is approximately 2.8 MB.

# YSClaw Agent4 OpenSSL EVP Flow

- [x] Normalize Agent3 Lite diagnosis into canonical RootCauseBlueprint
- [x] Validate RootCauseBlueprint against Agent4 schema
- [x] Create OpenSpec change and Comet state for the OpenSSL performance fix
- [x] Expand sparse checkout for relevant OpenSSL build/config files
- [x] Generate PatchPlan from validated blueprint
- [x] Implement the minimal OpenSSL patch
- [x] Capture git diff and generate PatchCandidate
- [x] Run available verification and record Agent1-compatible regression evidence
- [x] Generate Agent5 patch, VerifiedPatchPackage, test report, and development summary
- [x] Validate final artifacts and report output paths

## Review: YSClaw Agent4 OpenSSL EVP Flow

- Generated Agent4 artifacts under `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl/.yuan-sheng/optimize-riscv-sha256-march/`.
- Generated OpenSpec and Comet artifacts under `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl/openspec/changes/optimize-riscv-sha256-march/`.
- Implemented a minimal documentation patch in `doc/man3/OPENSSL_riscvcap.pod`.
- Final package schema validates, but `verification.status` is `failed` because `agent1 patch_regression --case openssl_test_evp` and target RISC-V performance checks could not run in this environment.
- Comet state remains `phase: verify`, `verify_result: fail`, `archived: false`.

# YSClaw Agent4 OpenSSL BIO ENC Flow

- [x] Create isolated OpenSSL worktree for `openssl_test_bio_enc`
- [x] Normalize Agent3 Lite diagnosis into canonical RootCauseBlueprint
- [x] Validate RootCauseBlueprint against Agent4 schema
- [x] Create OpenSpec change and Comet state
- [x] Expand sparse checkout for relevant OpenSSL files
- [x] Generate PatchPlan from validated blueprint
- [x] Implement the minimal OpenSSL patch
- [x] Capture git diff and generate PatchCandidate
- [x] Run available verification and record Agent1-compatible regression evidence
- [x] Generate Agent5 patch, VerifiedPatchPackage, test report, and development summary
- [x] Validate final artifacts and report output paths

## Review: YSClaw Agent4 OpenSSL BIO ENC Flow

- Generated Agent4 artifacts under `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-bio-enc-agent4/.yuan-sheng/document-riscv-aes-capability-overrides/`.
- Generated OpenSpec and Comet artifacts under `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-bio-enc-agent4/openspec/changes/document-riscv-aes-capability-overrides/`.
- Implemented a minimal documentation patch in `doc/man3/OPENSSL_riscvcap.pod`.
- Final package schema validates, but `verification.status` is `failed` because `agent1 patch_regression --case openssl_test_bio_enc` could not run in this environment.
- Comet state remains `phase: verify`, `verify_result: fail`, `archived: false`.

# YSClaw Agent4 OpenSSL STORE CASES Flow

- [x] Create isolated OpenSSL worktree for `openssl_test_store_cases`
- [x] Normalize Agent3 Lite diagnosis into canonical RootCauseBlueprint
- [x] Validate RootCauseBlueprint against Agent4 schema
- [x] Create OpenSpec change and Comet state
- [x] Expand sparse checkout for relevant OpenSSL files
- [x] Generate PatchPlan from validated blueprint
- [x] Implement the minimal OpenSSL patch
- [x] Capture git diff and generate PatchCandidate
- [x] Run available verification and record Agent1-compatible regression evidence
- [x] Generate Agent5 patch, VerifiedPatchPackage, test report, and development summary
- [x] Validate final artifacts and report output paths

## Review: YSClaw Agent4 OpenSSL STORE CASES Flow

- Generated Agent4 artifacts under `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-store-cases-agent4/.yuan-sheng/document-riscv-sha-capability-overrides/`.
- Generated OpenSpec and Comet artifacts under `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-store-cases-agent4/openspec/changes/document-riscv-sha-capability-overrides/`.
- Implemented a behavior-neutral documentation patch in `doc/man3/OPENSSL_riscvcap.pod`.
- Final package schema validates, but `verification.status` is `failed` because `agent1 patch_regression --case openssl_test_store_cases` could not run in this environment.
- Comet state remains `phase: verify`, `verify_result: fail`, `archived: false`.

# YSClaw Agent4 OpenSSL SLH DSA Flow

- [x] Create isolated OpenSSL worktree for `openssl_test_slh_dsa`
- [x] Normalize Agent3 Lite diagnosis into canonical RootCauseBlueprint
- [x] Validate RootCauseBlueprint against Agent4 schema
- [x] Create OpenSpec change and Comet state
- [x] Inspect relevant OpenSSL RISC-V SHA-256/FIPS code paths
- [x] Generate PatchPlan from validated blueprint
- [x] Implement the minimal OpenSSL patch
- [x] Capture git diff and generate PatchCandidate
- [x] Run available verification and record Agent1-compatible regression evidence
- [x] Generate Agent5 patch, VerifiedPatchPackage, test report, and development summary
- [x] Validate final artifacts and report output paths

## Review: YSClaw Agent4 OpenSSL SLH DSA Flow

- Generated Agent4 artifacts under `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-slh-dsa-agent4/.yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/`.
- Generated OpenSpec and Comet artifacts under `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-slh-dsa-agent4/openspec/changes/document-riscv-sha2-slh-dsa-dispatch/`.
- Implemented a behavior-neutral documentation patch in `doc/man3/OPENSSL_riscvcap.pod`.
- Final package schema validates, but `verification.status` is `failed` because `agent1 patch_regression --case openssl_test_slh_dsa` could not run in this environment.
- Comet state remains `phase: verify`, `verify_result: fail`, `archived: false`.

# Move Agent4 Artifacts To Codex Directory

- [x] Confirm the four source worktrees and artifact directories
- [x] Move only generated `.yuan-sheng`, `openspec`, and `docs/superpowers` artifacts into `codex`
- [x] Verify the destination contains the four artifact bundles and no OpenSSL source tree
- [x] Record relocation result

## Review: Move Agent4 Artifacts To Codex Directory

- Moved four artifact bundles into `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/ysclaw-test/codex/`.
- Destination groups are `0.openssl_test_evp`, `1.openssl_test_bio_enc`, `2.openssl_test_store_cases`, and `3.openssl_test_slh_dsa`.
- Each group preserves `.yuan-sheng`, `openspec`, and `docs/superpowers` artifact structure.
- Verified key Agent4/Agent5 files exist in each group and no OpenSSL source tree directories were moved into `codex`.

# Redo Agent4 Work With Code-Level OpenSSL Optimizations

- [x] Re-read four root-cause blueprints and relevant OpenSSL RISC-V code paths
- [ ] Write and confirm a code-level implementation plan before source edits
- [ ] Create or switch to a clean optimization worktree
- [ ] Add focused regression tests for RISC-V capability parsing/dispatch behavior
- [ ] Implement minimal code-level RISC-V optimization changes
- [ ] Run available build/test/documentation/schema verification
- [ ] Regenerate Agent4/Agent5 artifacts under `codex`
- [ ] Record final verification status honestly
