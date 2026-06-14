# OpenSSL RISC-V Code Optimization Redo Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the prior documentation-only Agent4 output with code-level OpenSSL RISC-V optimization work for the four root-cause blueprints.

**Architecture:** Implement one low-risk shared OpenSSL patch set that makes existing RISC-V SHA/AES accelerated code paths more likely to be selected at runtime without changing the generic `linux64-riscv64` compile-time ISA baseline. Keep SHA-1 as an explicit gap unless a separate RISC-V SHA-1 implementation is approved.

**Tech Stack:** OpenSSL C, OpenSSL build.info, RISC-V capability detection, provider cipher dispatch, SHA assembly dispatch, Perl-generated assembly checks.

---

## File Structure

- Modify: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4/include/arch/riscv_arch.h`
  - Allow Linux RISC-V hwprobe capability detection for the FIPS module, not only libcrypto/default-provider builds.
  - Preserve generic RISC-V baseline; do not add `-march=rv64gc_zbb` to `linux64-riscv64`.
- Modify: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4/crypto/riscvcap.c`
  - Keep runtime capability setup centralized.
  - If needed, factor capability bit application into a small helper so tests can validate behavior without real hardware.
- Modify: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4/crypto/sha/sha_riscv.c`
  - Add a small RISC-V SHA-256/SHA-512 dispatch helper so existing Zbb/vector implementations are selected through a single documented branch.
  - Avoid adding new SHA-1 code in this patch.
- Modify or create: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4/test/rdcpu_sanitytest.c` or `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4/test/riscvcap_internal_test.c`
  - Add a focused RISC-V-only internal test for `OPENSSL_riscvcap` override parsing and dispatch preconditions if it can be made deterministic.
- Modify: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4/test/build.info`
  - Register the RISC-V internal test only if a new file is required.
- Create/update: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/ysclaw-test/codex/*`
  - Regenerate Agent4/Agent5 artifacts so `patch.diff` contains code changes, not only documentation.

## Task 1: Prepare Clean Worktree

**Files:**
- Create worktree: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4`

- [ ] **Step 1: Create a clean branch from upstream**

Run:

```bash
git -C /Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl fetch upstream master
git -C /Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl worktree add -b codex/openssl-riscv-code-optimization /Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4 upstream/master
```

Expected: new worktree on `codex/openssl-riscv-code-optimization`.

- [ ] **Step 2: Expand required sparse paths**

Run:

```bash
git -C /Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4 sparse-checkout add \
  crypto/aes crypto/modes crypto/sha crypto/riscvcap.c include/arch include/crypto \
  providers test Configurations doc/man3
```

Expected: files needed for code, tests, and artifacts are present.

## Task 2: Prove The FIPS RISC-V Capability Detection Gap

**Files:**
- Read: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4/include/arch/riscv_arch.h`
- Read: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4/crypto/riscvcap.c`
- Read: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4/providers/fips/fipsprov.c`

- [ ] **Step 1: Verify current guard**

Run:

```bash
rg -n "OPENSSL_SYS_LINUX|FIPS_MODULE|OSSL_RISCV_HWPROBE|OPENSSL_cpuid_setup" \
  include/arch/riscv_arch.h crypto/riscvcap.c providers/fips/fipsprov.c
```

Expected: `include/arch/riscv_arch.h` gates `OSSL_RISCV_HWPROBE` with `!defined(FIPS_MODULE)` while `providers/fips/fipsprov.c` still calls `OPENSSL_cpuid_setup()`.

- [ ] **Step 2: Record root-cause note**

Add a short debugging note to the new Agent4 development summary: FIPS provider contains RISC-V SHA/AES accelerated sources but the FIPS build cannot auto-populate RISC-V caps from hwprobe due to the header guard.

## Task 3: Write Failing Capability Test

**Files:**
- Modify or create: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4/test/riscvcap_internal_test.c`
- Modify: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4/test/build.info`

- [ ] **Step 1: Add a RISC-V-only test skeleton**

Create a test that is compiled only for `defined(__riscv) && defined(OPENSSL_CPUID_OBJ)`.

```c
#include "testutil.h"
#include "arch/riscv_arch.h"

static int test_riscv_cap_override_parses_crypto_extensions(void)
{
    return TEST_true(RISCV_HAS_ZBB())
        && TEST_true(RISCV_HAS_ZKND())
        && TEST_true(RISCV_HAS_ZKNE())
        && TEST_true(RISCV_HAS_ZVKNED());
}
```

Use `OPENSSL_riscvcap=rv64gc_zbb_zknd_zkne_v_zvkned` when running this test.

- [ ] **Step 2: Run test before implementation**

Run on a RISC-V build host:

```bash
OPENSSL_riscvcap=rv64gc_zbb_zknd_zkne_v_zvkned make test TESTS=riscvcap_internal_test
```

Expected before implementation: the test either is not available yet or fails until the helper/reset strategy is implemented.

## Task 4: Implement Runtime Capability Fix

**Files:**
- Modify: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4/include/arch/riscv_arch.h`
- Modify: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4/crypto/riscvcap.c`

- [ ] **Step 1: Enable hwprobe for FIPS module**

Change:

```c
#if defined(OPENSSL_SYS_LINUX) && !defined(FIPS_MODULE)
```

to:

```c
#if defined(OPENSSL_SYS_LINUX)
```

Keep the existing `__has_include(<asm/hwprobe.h>)` and `__NR_riscv_hwprobe` guards.

- [ ] **Step 2: Keep vector-dependent capability gating**

Preserve:

```c
if (!IS_IN_DEPEND_VECTOR(RISCV_capabilities[i].bit_offset) || VECTOR_CAPABLE)
    OPENSSL_riscvcap_P[RISCV_capabilities[i].index] |= ...
```

Expected: vector crypto caps are still accepted only when base V is present.

## Task 5: Implement SHA Dispatch Cleanup

**Files:**
- Modify: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4/crypto/sha/sha_riscv.c`

- [ ] **Step 1: Add helper predicates**

Add helpers:

```c
static int riscv_has_sha256_vector(void)
{
    return RISCV_HAS_ZVKB()
        && (RISCV_HAS_ZVKNHA() || RISCV_HAS_ZVKNHB())
        && riscv_vlen() >= 128;
}

static int riscv_has_sha512_vector(void)
{
    return RISCV_HAS_ZVKB_AND_ZVKNHB() && riscv_vlen() >= 128;
}
```

- [ ] **Step 2: Use helper predicates in dispatch wrappers**

Replace inline compound conditions in `sha256_block_data_order()` and `sha512_block_data_order()` with the helpers.

Expected: same behavior, clearer dispatch boundary, easier verification. No SHA-1 change in this task.

## Task 6: Verify Locally Available Checks

**Files:**
- Worktree: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/openssl-codeopt-agent4`

- [ ] **Step 1: Static source verification**

Run:

```bash
git diff --check
perl crypto/sha/asm/sha256-riscv64-zbb.pl linux64 /tmp/sha256-riscv64-zbb.S >/dev/null
perl crypto/sha/asm/sha256-riscv64-zvkb-zvknha_or_zvknhb.pl linux64 /tmp/sha256-riscv64-zv.S >/dev/null
perl crypto/aes/asm/aes-riscv64-zkn.pl linux64 /tmp/aes-riscv64-zkn.S >/dev/null
perl crypto/aes/asm/aes-riscv64-zvkned.pl linux64 /tmp/aes-riscv64-zvkned.S >/dev/null
```

Expected: all commands exit 0.

- [ ] **Step 2: Build/test if toolchain is available**

Run:

```bash
./Configure linux64-riscv64 no-shared enable-fips
make -j4
make test TESTS="test_evp test_bio_enc test_store test_slh_dsa"
```

Expected: run only on a RISC-V or cross-capable environment. If unavailable locally, record as blocked and keep verification failed.

## Task 7: Regenerate Agent4 Artifacts

**Files:**
- Update: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/ysclaw-test/codex/0.openssl_test_evp`
- Update: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/ysclaw-test/codex/1.openssl_test_bio_enc`
- Update: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/ysclaw-test/codex/2.openssl_test_store_cases`
- Update: `/Users/zq/Desktop/ai-projs/posp/yuan-sheng/ysclaw-test/codex/3.openssl_test_slh_dsa`

- [ ] **Step 1: Capture code diff**

Run:

```bash
git diff --no-ext-diff > /tmp/openssl-riscv-code-optimization.patch
```

Expected: patch includes C/header/test changes, not only `doc/man3/OPENSSL_riscvcap.pod`.

- [ ] **Step 2: Update per-blueprint packages**

For each blueprint, regenerate `PatchCandidate`, `PatchRegressionResult`, `VerifiedPatchPackage`, test report, and development summary using the ysclaw-agent4 tools.

Expected: artifact schema validates; verification remains failed if Agent1/hardware regression is unavailable.

## Scope Boundaries

- Do not force `-march=rv64gc_zbb`, `rv64gcv`, or crypto extensions on the generic `linux64-riscv64` target.
- Do not claim performance is fixed without SG2044 runtime evidence.
- Do not add a new RISC-V SHA-1 implementation in this patch. The `2.openssl_test_store_cases` blueprint remains partially addressed by capability plumbing and must record SHA-1 as a follow-up if the hotspot persists.
- Do not delete or revert the previous artifact bundles until replacement code-level artifacts are generated and validated.
