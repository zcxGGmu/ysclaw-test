## Context

Agent3 Lite reported `openssl_test_bio_enc` as spending all profiled CPU samples in scalar `AES_encrypt` on SG2044. The diagnosis suggested a missing `-march` capability mismatch, but also explicitly blocked automatic Agent4 forwarding because the SG2044 crypto extension availability was not confirmed.

Current OpenSSL master already includes RISC-V AES acceleration sources in `crypto/aes/build.info`, RISC-V AES prototypes in `include/crypto/aes_platform.h`, and provider dispatch through `providers/implementations/ciphers/cipher_aes_hw_rv64i.inc`. This makes a default build-configuration or provider-code patch unsafe without target hardware evidence.

## Goals / Non-Goals

**Goals:**

- Add documented AES-focused `OPENSSL_riscvcap` override examples for diagnosing RISC-V AES dispatch.
- Keep the change behavior-neutral and suitable for upstream review.
- Capture the Agent4 evidence chain and verification limitations honestly.

**Non-Goals:**

- Do not change `Configurations/10-main.conf` or force `-march` extensions.
- Do not alter RISC-V AES provider dispatch, perlasm output, or BIO cipher code.
- Do not claim the performance issue is fixed without SG2044 and Agent1 regression evidence.

## Decisions

1. Documentation-only patch over runtime patch.
   - Rationale: The current source already contains scalar and vector RISC-V AES accelerated paths. The observed scalar `AES_encrypt` path can be caused by runtime capability detection, build provenance, or unavailable hardware extensions.
   - Alternative rejected: Add default `-march=rv64gcv_zvkned`. This risks increasing assembler/compiler requirements and is not justified because OpenSSL's RISC-V assembly can encode instructions independently of the compiler ISA string.

2. Use `doc/man3/OPENSSL_riscvcap.pod` as the only changed source file.
   - Rationale: The file already documents RISC-V capability names, Linux hwprobe behavior, and generic override examples. AES-specific examples fit naturally there.
   - Alternative rejected: Add extra `test/bio_enc_test.c` coverage. That would not address the diagnostic gap or prove performance dispatch behavior.

3. Treat unavailable Agent1 and SG2044 verification as a failed verification state.
   - Rationale: Agent4 packaging can include failed or error regression evidence, but it must not represent the result as verified.

## Risks / Trade-offs

- Users may misuse overrides as production configuration -> Mitigation: state that overrides are for explicit testing, diagnostics, and benchmarking, and that normal Linux operation should use hwprobe detection.
- Documentation may imply a capability combination works on all hardware -> Mitigation: examples are framed as capability overrides, not hardware guarantees.
- The original performance symptom remains unproven locally -> Mitigation: record required SG2044 commands and mark final verification as failed when Agent1 and target hardware are unavailable.

## Migration Plan

No migration is required. The change only updates a manual page. Rollback is a normal revert of the documentation diff.

## Open Questions

- Does the reported SG2044 system expose `zvkned`, `zkne`, and `zknd` through `/proc/cpuinfo` or `riscv_hwprobe`?
- Was the profiled `libcrypto.so.4` built from current OpenSSL master or an older tree without the same RISC-V AES dispatch coverage?
