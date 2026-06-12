---
comet_change: document-riscv-aes-capability-overrides
role: technical-design
canonical_spec: openspec
---

# RISC-V AES Capability Overrides Design

## Context

The source blueprint for `openssl_test_bio_enc` reports scalar `AES_encrypt` as the RISC-V hotspot on SG2044. Current OpenSSL master already contains RISC-V AES scalar crypto and vector crypto code paths, so the safest Agent4 action is to improve diagnostic documentation rather than alter runtime dispatch or build defaults.

## Approach

The implementation changes only `doc/man3/OPENSSL_riscvcap.pod`. It adds AES-focused examples under the existing `EXAMPLES` section:

- `rv64gc_zknd_zkne` for scalar AES crypto override testing.
- `rv64gc_v_zvkb_zvkned` for vector AES mode override testing.
- `rv64gc_v_zvkb_zvkg_zvkned` for vector AES-GCM override testing.

The documentation will explicitly frame these examples as testing, diagnostics, and benchmarking aids. It will also state that normal Linux execution should prefer runtime capability detection through hwprobe.

## Boundaries

No runtime code, ABI, provider dispatch logic, assembler source, or Configure target will change. The patch does not claim to fix the SG2044 performance symptom; it makes the diagnostic path reproducible and reviewable.

## Validation

Local validation will cover POD syntax and source consistency checks. Required Agent1 regression remains `agent1 patch_regression --case openssl_test_bio_enc`. If Agent1 or SG2044 hardware is unavailable, verification must be recorded as failed or errored rather than treated as passing.
