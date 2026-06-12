# Comet Design Handoff

- Change: document-riscv-sha2-slh-dsa-dispatch
- Phase: design
- Mode: compact
- Context hash: 7df34b9eeaee9e01dbc1acd6e84330f477916b98494954eb30cb980d00130fb1

Generated-by: comet-handoff.sh

OpenSpec remains the canonical capability spec. This handoff is a deterministic, source-traceable context pack, not an agent-authored summary.

## openspec/changes/document-riscv-sha2-slh-dsa-dispatch/proposal.md

- Source: openspec/changes/document-riscv-sha2-slh-dsa-dispatch/proposal.md
- Lines: 1-28
- SHA256: 0c4c150558e43771675375c07030971a2926081592a420448071cb694caa276d

```md
## Why

The Agent3 SLH-DSA diagnosis reports `sha256_block_data_order_riscv64` as a scalar RISC-V hotspot on SG2044 and recommends adding `-march=rv64gc_zba_zbb` to the default `linux64-riscv64` target. Current OpenSSL master already contains RISC-V SHA-256 Zbb and vector crypto implementations with runtime dispatch, so forcing a higher global ISA baseline would be unsafe for generic RISC-V Linux builds.

The documentation should make the intended diagnostic path clear: inspect or override RISC-V capabilities when validating SHA-2 dispatch, while preserving hwprobe-based runtime selection for normal execution.

## What Changes

- Add RISC-V SHA-2-focused `OPENSSL_riscvcap` examples for Zbb and vector SHA-256 capability combinations.
- Clarify that SHA capability overrides affect runtime-dispatched SHA-2 implementations and do not change the compiler-selected base architecture or default `linux64-riscv64` target baseline.
- State that normal Linux execution should prefer hwprobe-detected capabilities over manual overrides.
- Preserve runtime behavior: no Configure, dispatch, assembly, provider, FIPS, SLH-DSA, or test harness code changes.

## Capabilities

### New Capabilities

- `riscv-sha2-slh-dsa-dispatch`: Documents how users diagnosing SLH-DSA SHA-2 profiles can inspect and override RISC-V SHA-2 capability bits without raising the generic build ISA baseline.

### Modified Capabilities

- None.

## Impact

- Affected file: `doc/man3/OPENSSL_riscvcap.pod`.
- No ABI, API, build-system, digest implementation, provider, FIPS, or test harness changes.
- The Agent4 package remains verification-blocked until `agent1 patch_regression --case openssl_test_slh_dsa` and SG2044 capability/build checks can run.
```

## openspec/changes/document-riscv-sha2-slh-dsa-dispatch/design.md

- Source: openspec/changes/document-riscv-sha2-slh-dsa-dispatch/design.md
- Lines: 1-52
- SHA256: f983b4425e3b3f24264f9ab5f8ab451ed85d57d114abb863737773901cd748b5

```md
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
```

## openspec/changes/document-riscv-sha2-slh-dsa-dispatch/tasks.md

- Source: openspec/changes/document-riscv-sha2-slh-dsa-dispatch/tasks.md
- Lines: 1-16
- SHA256: 429528bfed4d575aa6e4115ba022b430722ce000234c0bc70d61c6f2bf28ee86

```md
## 1. Agent4 Planning

- [ ] 1.1 Normalize the Agent3 Lite diagnosis into `.yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/agent3/root-cause-blueprint.json`
- [ ] 1.2 Validate the canonical RootCauseBlueprint with the ysclaw-agent4 schema tool
- [ ] 1.3 Generate and validate `.yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/agent4/patch-plan.json`

## 2. Documentation Patch

- [ ] 2.1 Update `doc/man3/OPENSSL_riscvcap.pod` with SHA-2-specific RISC-V override examples
- [ ] 2.2 Keep the documentation clear that overrides are diagnostic, hwprobe is preferred for normal Linux execution, and global `linux64-riscv64` `-march` defaults are not changed

## 3. Verification and Packaging

- [ ] 3.1 Run available local documentation and source verification commands
- [ ] 3.2 Capture the true git diff as `.yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/agent4/patch.diff`
- [ ] 3.3 Generate and validate PatchCandidate, PatchRegressionResult, Agent5 patch, VerifiedPatchPackage, test report, and development summary
```

## openspec/changes/document-riscv-sha2-slh-dsa-dispatch/specs/riscv-sha2-slh-dsa-dispatch/spec.md

- Source: openspec/changes/document-riscv-sha2-slh-dsa-dispatch/specs/riscv-sha2-slh-dsa-dispatch/spec.md
- Lines: 1-30
- SHA256: 4eb20f3113c61051804e7a83fe7319e468fd382d7fb34fb34455efa43b0ed874

```md
## ADDED Requirements

### Requirement: SHA-2 capability override examples
The `OPENSSL_riscvcap` manual page SHALL include examples for explicitly enabling RISC-V SHA-2-related Zbb and vector crypto capability combinations.

#### Scenario: Zbb SHA-2 override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for SHA-2 diagnostics
- **THEN** the documentation identifies `rv64gc_zbb` as an override useful for testing RISC-V Zbb SHA-2 paths

#### Scenario: Vector SHA-256 override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for SHA-256 diagnostics
- **THEN** the documentation identifies `rv64gc_v_zvkb_zvknha` as an override useful for testing RISC-V vector SHA-256 paths

#### Scenario: Vector SHA-512 override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for SHA-512 diagnostics
- **THEN** the documentation identifies `rv64gc_v_zvkb_zvknhb` as an override useful for testing RISC-V vector SHA-512 paths

### Requirement: Runtime dispatch scope guidance
The `OPENSSL_riscvcap` manual page SHALL state that SHA-specific overrides affect runtime-dispatched implementations and do not change compiler-selected architecture flags or the generic `linux64-riscv64` target baseline.

#### Scenario: Diagnosing scalar SLH-DSA SHA-256
- **WHEN** a user is diagnosing a scalar `sha256_block_data_order_riscv64` profile from `openssl_test_slh_dsa`
- **THEN** the documentation distinguishes runtime capability selection from compile-time `-march` policy

### Requirement: Override safety guidance
The `OPENSSL_riscvcap` manual page SHALL state that SHA-specific overrides are for explicit testing, diagnostics, and benchmarking, while normal Linux execution should prefer hwprobe-detected capabilities.

#### Scenario: Production guidance
- **WHEN** a user reads the SHA override examples
- **THEN** the documentation distinguishes diagnostic overrides from the preferred Linux hwprobe path for ordinary execution
```

