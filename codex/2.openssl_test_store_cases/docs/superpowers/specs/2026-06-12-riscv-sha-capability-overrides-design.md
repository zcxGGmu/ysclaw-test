---
comet_change: document-riscv-sha-capability-overrides
role: technical-design
canonical_spec: openspec
---

# RISC-V SHA Capability Overrides Design

## Context

The source blueprint for `openssl_test_store_cases` reports scalar `SHA1_Final` as the RISC-V hotspot on SG2044. Current OpenSSL master includes RISC-V Zbb and vector crypto dispatch for SHA-256 and SHA-512, but not SHA-1. The safe Agent4 action is documentation that helps reviewers diagnose capability state without implying a source-level SHA-1 fix exists.

## Approach

The implementation changes only `doc/man3/OPENSSL_riscvcap.pod`. It adds SHA-focused examples under the existing `EXAMPLES` section:

- `rv64gc_zbb` for Zbb SHA-2 path override testing.
- `rv64gc_v_zvkb_zvknha` for vector SHA-256 override testing.
- `rv64gc_v_zvkb_zvknhb` for vector SHA-512 override testing.

The documentation will explicitly frame these examples as testing, diagnostics, and benchmarking aids. It will also state that the current RISC-V SHA crypto dispatch applies to SHA-2 implementations and does not add a SHA-1 crypto dispatch path.

## Boundaries

No runtime code, ABI, provider dispatch logic, assembler source, Configure target, or test recipe will change. The patch does not claim to fix the SG2044 `SHA1_Final` performance symptom; it makes the diagnostic path reproducible and reviewable.

## Validation

Local validation will cover POD syntax and source consistency checks. Required Agent1 regression remains `agent1 patch_regression --case openssl_test_store_cases`. If Agent1 or SG2044 hardware is unavailable, verification must be recorded as failed or errored rather than treated as passing.
