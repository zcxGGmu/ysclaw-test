## Context

Agent3 Lite identified `sha256_block_data_order_riscv64` in `openssl_test_slh_dsa` as a scalar RISC-V hotspot and proposed adding Zbb/Zba build flags to `linux64-riscv64`. Current OpenSSL master already builds three SHA-256 RISC-V paths: baseline RV64, Zbb, and vector SHA-256. `crypto/sha/sha_riscv.c` chooses vector SHA-256 when ZVKB plus ZVKNHA or ZVKNHB and VLEN >= 128 are detected, otherwise Zbb when Zbb is detected, otherwise the baseline `sha256_block_data_order_riscv64` path.

The Agent3 diagnosis also blocked automatic forwarding because key evidence was missing: direct `readelf -A` build ISA output, SG2044 hwprobe capability confirmation, and an ARM comparison baseline. That makes a source or Configure change too risky for Agent4.

## Goals / Non-Goals

**Goals:**

- Document SHA-2 RISC-V capability override examples relevant to SLH-DSA SHA2 profiles.
- Make clear that capability overrides exercise runtime dispatch and do not change the compiler-selected base architecture.
- Keep the patch upstream-friendly and behavior-neutral.
- Package the full Agent4 evidence chain with honest failed verification status when Agent1 is unavailable.

**Non-Goals:**

- Do not force `-march=rv64gc_zba_zbb` or any other extension into `linux64-riscv64`.
- Do not alter RISC-V SHA-256 dispatch logic, generated assembly, FIPS provider code, or SLH-DSA implementation code.
- Do not claim the SG2044 performance symptom is fixed locally.

## Decisions

1. Documentation-only patch over Configure target changes.
   - Rationale: Raising the generic RISC-V Linux ISA baseline would allow compilers to emit Zbb instructions in ordinary C code and could break systems without Zbb.
   - Alternative rejected: Add default `-march=rv64gc_zba_zbb`. This weakens runtime dispatch and is not safe for OpenSSL's generic target.

2. Use `doc/man3/OPENSSL_riscvcap.pod` as the only source change.
   - Rationale: The file already documents RISC-V capability names and override syntax. SHA-2 diagnostic examples belong in the examples section.
   - Alternative rejected: Add SLH-DSA or provider tests. The symptom is performance/capability selection, not a functional SLH-DSA correctness gap.

3. Keep final verification failed unless Agent1 regression runs.
   - Rationale: The Agent4 package format supports failed/error regression evidence. Treating unavailable Agent1 as pass would be misleading.

## Risks / Trade-offs

- Documentation-only patch does not reduce SG2044 SLH-DSA runtime.
  - Mitigation: Record the required target-side checks and failed verification state.
- Users may overuse `OPENSSL_riscvcap` overrides in production.
  - Mitigation: explicitly prefer Linux hwprobe-detected capabilities for normal execution.
- SG2044 may not expose the required vector SHA-2 extensions.
  - Mitigation: frame vector examples as diagnostics and require `/proc/cpuinfo`, `riscv_hwprobe`, and `openssl info -cpusettings` confirmation.

## Migration Plan

No migration is required. The change only updates a manual page. Rollback is a normal documentation revert.

## Open Questions

- Does the target SG2044 expose Zbb, ZVKB, ZVKNHA, or ZVKNHB through Linux hwprobe?
- Was the profiled FIPS module built from current OpenSSL master with the current RISC-V SHA-256 dispatch code?
- Does target-side `OPENSSL_riscvcap=rv64gc_zbb` select `sha256_block_data_order_zbb` for the SLH-DSA SHA2 profile?
