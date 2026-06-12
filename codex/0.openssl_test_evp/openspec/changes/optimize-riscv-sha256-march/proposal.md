# Optimize RISC-V SHA-256 Build Target Selection

## Why

Agent3 diagnosed a performance regression in `openssl test_evp` on SOPHON SG2044 / T-Head C920v2. The SHA-256 compression hotspot `sha256_block_data_order_riscv64` executed as a scalar-only path even though the target hardware supports RVV 1.0 and Zbb. Perf evidence shows a compute-bound loop with no vector instructions and no Zbb rotate or byte-swap instructions.

The probable root cause is that the OpenSSL build configuration can use a generic `rv64gc`-style RISC-V target that does not advertise the `v` and `zbb` extensions to the compiler / assembler. That prevents optimized RISC-V SHA-256 code generation or selection for the affected build.

## What Changes

- Keep generic RISC-V targets compatible with baseline hardware and toolchains.
- Add or adjust an opt-in RISC-V configuration path that includes RVV and Zbb extension flags for SHA-256 performance validation.
- Preserve existing behavior for non-RISC-V targets and default RISC-V builds.
- Generate Agent4 handoff artifacts for PatchPlan, PatchCandidate, regression evidence, and VerifiedPatchPackage.

## Impact

- Users targeting RVV/Zbb-capable RISC-V systems get a documented build target that can enable the optimized instruction set.
- Generic RISC-V builds are not forced to require vector or bitmanip extensions.
- Hardware-level performance verification still requires a RISC-V target host because the current development host is macOS.
