---
change: document-riscv-sha-capability-overrides
design-doc: docs/superpowers/specs/2026-06-12-riscv-sha-capability-overrides-design.md
base-ref: afd25111f12ac3f8c7009ea84d827718e5620e31
---

# RISC-V SHA Capability Overrides Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add behavior-neutral RISC-V SHA capability override examples to the `OPENSSL_riscvcap` manual and package the Agent4 evidence chain.

**Architecture:** Keep the implementation to a single manual-page patch. Use ysclaw-agent4 tools as the source of truth for RootCauseBlueprint, PatchPlan, PatchCandidate, PatchRegressionResult, and VerifiedPatchPackage.

**Tech Stack:** OpenSSL POD documentation, OpenSpec, ysclaw-agent4 JSON schemas, shell verification commands.

---

## Files

- Modify: `doc/man3/OPENSSL_riscvcap.pod`
- Create/Update: `.yuan-sheng/document-riscv-sha-capability-overrides/agent3/root-cause-blueprint.json`
- Create/Update: `.yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch-plan.json`
- Create/Update: `.yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch.diff`
- Create/Update: `.yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch-candidate.json`
- Create/Update: `.yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch-regression-result.json`
- Create/Update: `.yuan-sheng/document-riscv-sha-capability-overrides/agent5/patch.diff`
- Create/Update: `.yuan-sheng/document-riscv-sha-capability-overrides/agent5/verified-patch-package.json`
- Create/Update: `.yuan-sheng/document-riscv-sha-capability-overrides/reports/test-report.md`
- Create/Update: `.yuan-sheng/document-riscv-sha-capability-overrides/reports/development-summary.md`

### Task 1: Validate Blueprint and PatchPlan

**Files:**
- Validate: `.yuan-sheng/document-riscv-sha-capability-overrides/agent3/root-cause-blueprint.json`
- Create: `.yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch-plan.json`

- [x] **Step 1: Validate RootCauseBlueprint**

Run:

```bash
node /Users/zq/.codex/plugins/cache/personal/ysclaw-agent4/0.1.0/tools/ysclaw-agent4-tools.js validate root-cause-blueprint .yuan-sheng/document-riscv-sha-capability-overrides/agent3/root-cause-blueprint.json
```

Expected: JSON output contains `"valid": true`.

- [x] **Step 2: Record canonical blueprint in Comet state**

Run:

```bash
bash /Users/zq/.codex/plugins/cache/personal/ysclaw-agent4/0.1.0/skills/comet/scripts/comet-state.sh set document-riscv-sha-capability-overrides root_cause_blueprint .yuan-sheng/document-riscv-sha-capability-overrides/agent3/root-cause-blueprint.json
```

Expected: state update succeeds.

- [x] **Step 3: Generate PatchPlan**

Run:

```bash
node /Users/zq/.codex/plugins/cache/personal/ysclaw-agent4/0.1.0/tools/ysclaw-agent4-tools.js plan .yuan-sheng/document-riscv-sha-capability-overrides/agent3/root-cause-blueprint.json .yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch-plan.json
```

Expected: patch plan JSON is created.

- [x] **Step 4: Validate PatchPlan**

Run:

```bash
node /Users/zq/.codex/plugins/cache/personal/ysclaw-agent4/0.1.0/tools/ysclaw-agent4-tools.js validate patch-plan .yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch-plan.json
```

Expected: JSON output contains `"valid": true`.

### Task 2: Apply Documentation Patch

**Files:**
- Modify: `doc/man3/OPENSSL_riscvcap.pod`

- [x] **Step 1: Add SHA override examples**

Edit the `EXAMPLES` section after the existing vector-extension example. Add Zbb, vector SHA-256, and vector SHA-512 override examples.

- [x] **Step 2: Add scope and safety wording**

State that the SHA override examples are intended for explicit testing, diagnostics, and benchmarking, that normal Linux execution should prefer hwprobe-detected capabilities, and that current RISC-V SHA crypto dispatch is SHA-2 scoped.

- [x] **Step 3: Review the diff**

Run:

```bash
git diff -- doc/man3/OPENSSL_riscvcap.pod
```

Expected: only the intended documentation block changed.

### Task 3: Verify, Package, and Report

**Files:**
- Create: `.yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch.diff`
- Create: `.yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch-candidate.json`
- Create: `.yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch-regression-result.json`
- Create: `.yuan-sheng/document-riscv-sha-capability-overrides/agent5/patch.diff`
- Create: `.yuan-sheng/document-riscv-sha-capability-overrides/agent5/verified-patch-package.json`
- Create: `.yuan-sheng/document-riscv-sha-capability-overrides/reports/test-report.md`
- Create: `.yuan-sheng/document-riscv-sha-capability-overrides/reports/development-summary.md`

- [x] **Step 1: Run local syntax verification**

Run:

```bash
podchecker doc/man3/OPENSSL_riscvcap.pod
```

Expected: no POD syntax errors.

- [x] **Step 2: Run source consistency checks**

Run:

```bash
rg -n "rv64gc_zbb|rv64gc_v_zvkb_zvknha|rv64gc_v_zvkb_zvknhb|sha256_block_data_order_zbb|sha256_block_data_order_zvkb_zvknha_or_zvknhb|sha512_block_data_order_zvkb_zvknhb|RISCV_HAS_ZVKB_AND_ZVKNHB" crypto include doc/man3/OPENSSL_riscvcap.pod
```

Expected: output confirms RISC-V SHA-2 symbols and documentation examples.

- [x] **Step 3: Try Agent1 regression**

Run:

```bash
agent1 patch_regression --case openssl_test_store_cases
```

Expected: pass on an environment with Agent1 and the required test target. If unavailable, record an error result.

- [x] **Step 4: Generate PatchCandidate and regression package**

Use the ysclaw-agent4 tool sequence to generate `patch.diff`, `patch-candidate.json`, `patch-regression-result.json`, `agent5/patch.diff`, and `verified-patch-package.json`.

- [x] **Step 5: Validate final artifacts**

Run the ysclaw-agent4 `validate` command for all JSON artifacts and check that test and development reports contain the required headings.
