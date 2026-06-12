## Context

Agent3 Lite identified `SHA1_Final` in `openssl_test_store_cases` as a scalar RISC-V hotspot and proposed adding `v` and `zbb` build capabilities. That diagnosis is incomplete for current OpenSSL master: `crypto/sha/sha_riscv.c` dispatches RISC-V Zbb and vector crypto paths for SHA-256 and SHA-512, while SHA-1 remains on the generic SHA-1 code path unless another architecture-specific SHA-1 implementation applies.

The original diagnosis also blocked automatic forwarding because build ISA, Zbb/Zvknh hardware availability, and focused `sha1_block_data_order` evidence were not supplied.

## Goals / Non-Goals

**Goals:**

- Add documented SHA/Zbb `OPENSSL_riscvcap` override examples for RISC-V diagnostics.
- Make the current SHA-2-only scope of RISC-V SHA crypto dispatch explicit.
- Keep the patch upstream-friendly and behavior-neutral.
- Package the full Agent4 evidence chain with honest failed verification status when Agent1 is unavailable.

**Non-Goals:**

- Do not add RISC-V SHA-1 assembly or dispatch logic.
- Do not alter `Configurations/` default RISC-V compiler flags.
- Do not claim the `test_store_cases` performance symptom is fixed locally.

## Decisions

1. Documentation-only patch over SHA-1 implementation work.
   - Rationale: Implementing a RISC-V SHA-1 fast path would require new assembly, new dispatch decisions, and target validation. The blueprint lacks the required hardware and build evidence.
   - Alternative rejected: Add default `-march=rv64gc_v_zbb`. This would not create a SHA-1 vector crypto path and may raise toolchain expectations.

2. Use `doc/man3/OPENSSL_riscvcap.pod` as the only source change.
   - Rationale: The file already documents RISC-V capability names and override syntax. SHA/Zbb examples belong in the same examples section.
   - Alternative rejected: Add `test_store_cases` coverage. The current test already triggers store cases; added coverage would not validate the performance diagnosis.

3. Keep final verification failed unless Agent1 regression runs.
   - Rationale: The Agent4 package format supports failed/error regression evidence. Treating unavailable Agent1 as pass would be misleading.

## Risks / Trade-offs

- Users may expect SHA-1 to be accelerated by ZVKNH overrides -> Mitigation: explicitly say current RISC-V SHA crypto dispatch covers SHA-2, not SHA-1.
- Documentation-only patch does not reduce `SHA1_Final` runtime -> Mitigation: record the remaining SG2044 follow-up commands and failed verification state.
- ZVKNHA/ZVKNHB hardware may be absent on SG2044 -> Mitigation: frame examples as explicit override diagnostics and prefer hwprobe for normal Linux execution.

## Migration Plan

No migration is required. The change only updates a manual page. Rollback is a normal documentation revert.

## Open Questions

- Does the target SG2044 expose Zbb, ZVKB, ZVKNHA, or ZVKNHB through Linux hwprobe or `/proc/cpuinfo`?
- Was the profiled `libcrypto.so.4` built from current OpenSSL master or an older source tree?
- Would a future RISC-V SHA-1 implementation be worth adding, given SHA-1 deprecation and expected upstream review cost?
