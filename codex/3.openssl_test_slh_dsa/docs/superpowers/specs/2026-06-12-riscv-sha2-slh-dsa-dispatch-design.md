---
comet_change: document-riscv-sha2-slh-dsa-dispatch
role: technical-design
canonical_spec: openspec
---

# RISC-V SHA-2 SLH-DSA Dispatch Design

## Context

The source blueprint for `openssl_test_slh_dsa` reports scalar `sha256_block_data_order_riscv64` as the RISC-V SG2044 hotspot. The codebase now has three RISC-V SHA-256 implementations: baseline RV64, Zbb, and vector SHA-256. Runtime selection is in `crypto/sha/sha_riscv.c`, not in SLH-DSA provider code.

The Agent3 recommendation to force `-march=rv64gc_zba_zbb` into `linux64-riscv64` is not safe for upstream OpenSSL. That would raise the generic target baseline and could allow compilers to emit Zbb instructions in unrelated C code. It would also weaken the value of `OPENSSL_riscvcap` and hwprobe-based runtime dispatch.

## Approach

The implementation changes only `doc/man3/OPENSSL_riscvcap.pod`. It adds SHA-2-focused examples under the existing `EXAMPLES` section:

- `rv64gc_zbb` for Zbb SHA-2 override testing.
- `rv64gc_v_zvkb_zvknha` for vector SHA-256 override testing.
- `rv64gc_v_zvkb_zvknhb` for vector SHA-512 override testing.

The wording frames these examples as diagnostics and benchmarking aids. It states that the overrides affect runtime-dispatched implementations, do not change compiler-selected architecture flags, do not change the generic `linux64-riscv64` target baseline, and should not replace Linux hwprobe capability detection during normal execution.

## Boundaries

No runtime code, ABI, provider dispatch logic, FIPS source list, assembler generator, Configure target, or test recipe changes are part of this patch. The change does not claim to fix the SG2044 `openssl_test_slh_dsa` performance symptom; it records the safe diagnostic surface for target-side validation.

## Validation

Local validation covers:

- `ysclaw-agent4-tools.js validate` for RootCauseBlueprint, PatchPlan, PatchCandidate, PatchRegressionResult, and VerifiedPatchPackage.
- `openspec validate document-riscv-sha2-slh-dsa-dispatch`.
- `podchecker doc/man3/OPENSSL_riscvcap.pod`.
- `git diff --check`.
- `rg` checks tying the new examples to existing SHA-256/SHA-512 RISC-V dispatch symbols.

Required Agent1 regression remains `agent1 patch_regression --case openssl_test_slh_dsa`. If Agent1 is unavailable, the package must record `PatchRegressionResult.status: error` and `VerifiedPatchPackage.verification.status: failed`.
