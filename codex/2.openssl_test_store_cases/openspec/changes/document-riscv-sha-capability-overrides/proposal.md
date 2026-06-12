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
