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
