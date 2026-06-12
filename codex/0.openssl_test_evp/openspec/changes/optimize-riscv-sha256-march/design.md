# Design

## Context

The Agent3 blueprint points at OpenSSL build configuration rather than SHA-256 algorithm logic. The desired fix is to make the correct RISC-V ISA extension selection explicit for an RVV/Zbb-capable performance target.

## Decision

Use an opt-in target/configuration change instead of altering the generic RISC-V baseline. The implementation must avoid making `rv64gc` users require RVV or Zbb.

The expected implementation shape is:

- Inspect `Configurations/10-main.conf` and RISC-V SHA-256 generator sources.
- Prefer adding a named target derived from the existing Linux RISC-V configuration if no suitable target already exists.
- Set the target's architecture flags to include `v` and `zbb` for RVV/Zbb-capable systems.
- Keep the change small enough that configuration generation can be checked locally.

## Risks

- Some toolchains may not support all RISC-V extension spellings. The target must be opt-in so unsupported toolchains can keep using the generic target.
- The local macOS environment cannot prove RISC-V instruction emission or performance. The final report must distinguish local verification from required target-hardware verification.
- OpenSSL generated assembly paths are sensitive; do not edit generated assembly without updating the generator source.

## Verification

- Confirm the new/changed configuration is visible through `perl Configure LIST`.
- Confirm local `perl Configure <target>` accepts the target far enough to generate configuration metadata, or record the blocker if cross-toolchain support is unavailable.
- Validate Agent4 JSON artifacts with the bundled schema tool.
- Record Agent1-compatible regression evidence using `agent1 patch_regression --case openssl_test_evp`.
