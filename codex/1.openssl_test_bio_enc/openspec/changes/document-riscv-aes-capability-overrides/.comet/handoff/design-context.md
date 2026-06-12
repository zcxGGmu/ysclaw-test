# Comet Design Handoff

- Change: document-riscv-aes-capability-overrides
- Phase: design
- Mode: full
- Context hash: 55de84d668752a2c8e7c045247df69d49de7c940bd1bbfcbc0c895fcfe0bf8e2

Generated-by: comet-handoff.sh

OpenSpec remains the canonical capability spec. This handoff is a deterministic, source-traceable context pack, not an agent-authored summary.

## openspec/changes/document-riscv-aes-capability-overrides/proposal.md

- Source: openspec/changes/document-riscv-aes-capability-overrides/proposal.md
- Lines: 1-26
- SHA256: 620a58efd06e12d94b7bc6893f2f895a3dd31081dc19d7aa60d13ad0f48ae379

```md
## Why

The Agent3 BIO ENC diagnosis reports that AES_encrypt runs as a scalar RISC-V hot path, but current OpenSSL already contains RISC-V AES crypto dispatch implementations. Users need precise documentation for verifying and explicitly overriding RISC-V AES capability bits before treating this as a source-level AES implementation defect.

## What Changes

- Add RISC-V AES-specific OPENSSL_riscvcap examples for scalar crypto AES, vector AES, and vector AES-GCM capability combinations.
- Clarify that these overrides are intended for explicit testing, diagnostics, and benchmarking.
- Preserve the existing behavior that Linux hwprobe-based runtime detection is preferred for normal execution.
- No runtime code, build target, or provider dispatch behavior changes.

## Capabilities

### New Capabilities

- `riscv-aes-capability-overrides`: Documents how users can inspect and explicitly override RISC-V AES capability settings for diagnostics and benchmarking.

### Modified Capabilities

- None.

## Impact

- Affected file: `doc/man3/OPENSSL_riscvcap.pod`.
- No ABI, API, build-system, provider, cipher, or test harness changes.
- The Agent4 package remains verification-blocked until `agent1 patch_regression --case openssl_test_bio_enc` and SG2044 hardware capability checks can run.
```

## openspec/changes/document-riscv-aes-capability-overrides/design.md

- Source: openspec/changes/document-riscv-aes-capability-overrides/design.md
- Lines: 1-47
- SHA256: 4aee085ac9687b056781070c93733e2186c0a2e350264c8618a26031b815cc65

```md
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
```

## openspec/changes/document-riscv-aes-capability-overrides/tasks.md

- Source: openspec/changes/document-riscv-aes-capability-overrides/tasks.md
- Lines: 1-16
- SHA256: 2c958df8a939028c1dc3f2d49fa845339551784a38ee0d19d5e45cf732c0c120

```md
## 1. Agent4 Planning

- [ ] 1.1 Normalize the Agent3 Lite diagnosis into `.yuan-sheng/document-riscv-aes-capability-overrides/agent3/root-cause-blueprint.json`
- [ ] 1.2 Validate the canonical RootCauseBlueprint with the ysclaw-agent4 schema tool
- [ ] 1.3 Generate and validate `.yuan-sheng/document-riscv-aes-capability-overrides/agent4/patch-plan.json`

## 2. Documentation Patch

- [ ] 2.1 Update `doc/man3/OPENSSL_riscvcap.pod` with AES-specific RISC-V override examples
- [ ] 2.2 Keep the documentation clear that overrides are diagnostic and hwprobe is preferred for normal Linux execution

## 3. Verification and Packaging

- [ ] 3.1 Run available local documentation and source verification commands
- [ ] 3.2 Capture the true git diff as `.yuan-sheng/document-riscv-aes-capability-overrides/agent4/patch.diff`
- [ ] 3.3 Generate and validate PatchCandidate, PatchRegressionResult, Agent5 patch, VerifiedPatchPackage, test report, and development summary
```

## openspec/changes/document-riscv-aes-capability-overrides/specs/riscv-aes-capability-overrides/spec.md

- Source: openspec/changes/document-riscv-aes-capability-overrides/specs/riscv-aes-capability-overrides/spec.md
- Lines: 1-23
- SHA256: 782f4e857236cf36f40930a5016133f39eb635e81ebe19c38a0faedd42d2e64a

```md
## ADDED Requirements

### Requirement: AES capability override examples
The `OPENSSL_riscvcap` manual page SHALL include examples for explicitly enabling RISC-V AES scalar crypto and vector crypto capability combinations.

#### Scenario: Scalar AES override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for AES diagnostics
- **THEN** the documentation identifies the `rv64gc_zknd_zkne` override as a scalar AES crypto test configuration

#### Scenario: Vector AES override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for AES diagnostics
- **THEN** the documentation identifies the `rv64gc_v_zvkb_zvkned` override as a vector AES test configuration for modes such as CBC, CTR, ECB, CFB, and OFB

#### Scenario: Vector AES-GCM override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for AES diagnostics
- **THEN** the documentation identifies the `rv64gc_v_zvkb_zvkg_zvkned` override as a vector AES-GCM test configuration

### Requirement: Override safety guidance
The `OPENSSL_riscvcap` manual page SHALL state that AES-specific overrides are for explicit testing, diagnostics, and benchmarking, while normal Linux execution should prefer hwprobe-detected capabilities.

#### Scenario: Production guidance
- **WHEN** a user reads the AES override examples
- **THEN** the documentation distinguishes diagnostic overrides from the preferred Linux hwprobe path for ordinary execution
```

