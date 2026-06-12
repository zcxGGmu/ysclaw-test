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
