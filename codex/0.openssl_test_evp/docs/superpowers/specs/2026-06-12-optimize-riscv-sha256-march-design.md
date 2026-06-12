# Optimize RISC-V SHA-256 March Design

## Goal

Provide an OpenSSL path for RVV/Zbb-capable RISC-V SHA-256 performance investigations to verify that optimized implementations are built and selected, without changing the generic RISC-V baseline.

## Root Cause

Agent3 observed that `sha256_block_data_order_riscv64` ran as a scalar-only hot path on SG2044. Current OpenSSL master already builds separate RISC-V SHA-256 baseline, Zbb, and vector crypto implementations when `asm_arch => 'riscv64'` is active, and `sha_riscv.c` selects among them at runtime.

The corrected implementation hypothesis is narrower than the original Lite blueprint wording: a scalar-only profile usually means the optimized capability bits were not visible to OpenSSL at runtime, or the benchmark did not verify that the binary contained and selected the optimized symbols. `-march` can describe the compiler's base ISA metadata, but the SHA-256 dispatcher uses `hwprobe` or `OPENSSL_riscvcap` capability data.

## Design Choice

Keep OpenSSL's generic RISC-V build target unchanged. Add focused verification guidance and artifact evidence around the existing dispatcher and capability override path instead of forcing all RISC-V builds to advertise RVV/Zbb.

## Implementation Constraints

- Keep the changed file set minimal.
- Do not edit generated assembly output.
- Preserve current default target behavior.
- Prefer OpenSSL's existing RISC-V runtime capability model.
- Make local verification possible with `perl Configure LIST`, source checks, and generated Agent4 artifacts.

## Verification Strategy

Local verification:

- Schema-validate Agent4 JSON artifacts.
- Confirm RISC-V targets appear in `perl Configure LIST`.
- Confirm source checks show the optimized SHA-256 dispatcher and symbols are present.

Target-hardware verification:

- Run `openssl speed -evp sha256`.
- Run `perf annotate sha256_block_data_order_riscv64` and confirm RVV/Zbb instructions appear where supported.
- Run OpenSSL tests including EVP coverage.
