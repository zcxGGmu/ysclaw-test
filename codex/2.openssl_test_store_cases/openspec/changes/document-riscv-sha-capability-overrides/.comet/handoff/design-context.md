# Comet Design Handoff

- Change: document-riscv-sha-capability-overrides
- Phase: design
- Mode: full
- Context hash: dcc71dc13da76e1f54cb6056cf3ece976302fe89eb84790e936194e6e2faa54b

Generated-by: comet-handoff.sh

OpenSpec remains the canonical capability spec. This handoff is a deterministic, source-traceable context pack, not an agent-authored summary.

## openspec/changes/document-riscv-sha-capability-overrides/proposal.md

- Source: openspec/changes/document-riscv-sha-capability-overrides/proposal.md
- Lines: 1-26
- SHA256: 35ea90db9fe13d09268fe749a3febe170d1d0597b8383aefb84e2e02070aede3

```md
## Why

The Agent3 STORE CASES diagnosis reports a scalar `SHA1_Final` hot path on RISC-V, but current OpenSSL only has RISC-V SHA dispatch for SHA-256 and SHA-512. The `OPENSSL_riscvcap` manual should make SHA/Zbb capability override testing clearer and avoid implying that current RISC-V SHA crypto dispatch covers SHA-1.

## What Changes

- Add RISC-V SHA-focused `OPENSSL_riscvcap` examples for Zbb and vector SHA-2 capability combinations.
- Clarify that the existing RISC-V SHA crypto dispatch uses these capabilities for SHA-2 implementations.
- State that SHA-1 does not currently have a RISC-V SHA crypto dispatch path in OpenSSL.
- Preserve runtime behavior: no build, dispatch, digest, provider, or test harness code changes.

## Capabilities

### New Capabilities

- `riscv-sha-capability-overrides`: Documents how users can inspect and explicitly override RISC-V SHA-related capability settings for diagnostics and benchmarking.

### Modified Capabilities

- None.

## Impact

- Affected file: `doc/man3/OPENSSL_riscvcap.pod`.
- No ABI, API, build-system, digest implementation, provider, or test harness changes.
- The Agent4 package remains verification-blocked until `agent1 patch_regression --case openssl_test_store_cases` and SG2044 hardware capability checks can run.
```

## openspec/changes/document-riscv-sha-capability-overrides/design.md

- Source: openspec/changes/document-riscv-sha-capability-overrides/design.md
- Lines: 1-49
- SHA256: 6d503a73a0f92b0ab51854a190b82735d2056ef9927da7abfe70096e9489d88e

```md
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
```

## openspec/changes/document-riscv-sha-capability-overrides/tasks.md

- Source: openspec/changes/document-riscv-sha-capability-overrides/tasks.md
- Lines: 1-16
- SHA256: f84f438c60a3c92e8a48dd907ca6126cba0e68df8121e93974186fe344368e6c

```md
## 1. Agent4 Planning

- [ ] 1.1 Normalize the Agent3 Lite diagnosis into `.yuan-sheng/document-riscv-sha-capability-overrides/agent3/root-cause-blueprint.json`
- [ ] 1.2 Validate the canonical RootCauseBlueprint with the ysclaw-agent4 schema tool
- [ ] 1.3 Generate and validate `.yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch-plan.json`

## 2. Documentation Patch

- [ ] 2.1 Update `doc/man3/OPENSSL_riscvcap.pod` with SHA-specific RISC-V override examples
- [ ] 2.2 Keep the documentation clear that overrides are diagnostic, hwprobe is preferred for normal Linux execution, and current RISC-V SHA crypto dispatch is SHA-2 scoped

## 3. Verification and Packaging

- [ ] 3.1 Run available local documentation and source verification commands
- [ ] 3.2 Capture the true git diff as `.yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch.diff`
- [ ] 3.3 Generate and validate PatchCandidate, PatchRegressionResult, Agent5 patch, VerifiedPatchPackage, test report, and development summary
```

## openspec/changes/document-riscv-sha-capability-overrides/specs/riscv-sha-capability-overrides/spec.md

- Source: openspec/changes/document-riscv-sha-capability-overrides/specs/riscv-sha-capability-overrides/spec.md
- Lines: 1-30
- SHA256: 32f593dc54154802c52053bafc347540b11d667df1aee4e238057adb6ff90432

```md
## ADDED Requirements

### Requirement: SHA capability override examples
The `OPENSSL_riscvcap` manual page SHALL include examples for explicitly enabling RISC-V SHA-related Zbb and vector crypto capability combinations.

#### Scenario: Zbb SHA-2 override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for SHA diagnostics
- **THEN** the documentation identifies `rv64gc_zbb` as an override useful for testing RISC-V Zbb SHA-2 paths

#### Scenario: Vector SHA-256 override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for SHA diagnostics
- **THEN** the documentation identifies `rv64gc_v_zvkb_zvknha` as an override useful for testing RISC-V vector SHA-256 paths

#### Scenario: Vector SHA-512 override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for SHA diagnostics
- **THEN** the documentation identifies `rv64gc_v_zvkb_zvknhb` as an override useful for testing RISC-V vector SHA-512 paths

### Requirement: SHA-1 dispatch scope guidance
The `OPENSSL_riscvcap` manual page SHALL state that current RISC-V SHA crypto dispatch applies to SHA-2 implementations and does not add a SHA-1 crypto dispatch path.

#### Scenario: SHA-1 diagnosis
- **WHEN** a user is diagnosing a SHA-1 profile such as `SHA1_Final`
- **THEN** the documentation distinguishes SHA-1 from the RISC-V SHA-2 dispatch paths controlled by Zbb and ZVKNH capability bits

### Requirement: Override safety guidance
The `OPENSSL_riscvcap` manual page SHALL state that SHA-specific overrides are for explicit testing, diagnostics, and benchmarking, while normal Linux execution should prefer hwprobe-detected capabilities.

#### Scenario: Production guidance
- **WHEN** a user reads the SHA override examples
- **THEN** the documentation distinguishes diagnostic overrides from the preferred Linux hwprobe path for ordinary execution
```

