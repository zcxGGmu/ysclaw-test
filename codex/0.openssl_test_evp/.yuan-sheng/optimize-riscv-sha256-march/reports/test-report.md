# Test Report: optimize-riscv-sha256-march

## Summary

Verification status: failed

The local checks that can run on this macOS host passed, but the required Agent1 regression and target RISC-V performance checks could not be executed in this environment.

## Commands Run

- `node /Users/zq/.codex/plugins/cache/personal/ysclaw-agent4/0.1.0/tools/ysclaw-agent4-tools.js validate root-cause-blueprint .yuan-sheng/optimize-riscv-sha256-march/agent3/root-cause-blueprint.json`
  - Result: pass
- `node /Users/zq/.codex/plugins/cache/personal/ysclaw-agent4/0.1.0/tools/ysclaw-agent4-tools.js validate patch-plan .yuan-sheng/optimize-riscv-sha256-march/agent4/patch-plan.json`
  - Result: pass
- `podchecker doc/man3/OPENSSL_riscvcap.pod`
  - Result: pass
  - Output: `doc/man3/OPENSSL_riscvcap.pod pod syntax OK.`
- `perl Configure --help 2>&1 | rg 'linux64-riscv64|linux32-riscv32|BSD-riscv64|android-riscv64'`
  - Result: pass
  - Output includes the expected RISC-V targets.
- `rg -n 'sha256_block_data_order\\(|RISCV_HAS_ZVKB|RISCV_HAS_ZBB|sha256_block_data_order_zbb|sha256_block_data_order_zvkb' crypto/sha/sha_riscv.c crypto/sha/build.info`
  - Result: pass
  - Output confirms the dispatcher selects vector SHA-256 first, Zbb SHA-256 second, and the baseline implementation otherwise.
- `perl util/find-doc-nits doc/man3/OPENSSL_riscvcap.pod`
  - Result: error
  - Reason: requires `configdata.pm`.
- Out-of-tree `Configure` for doc-nits setup
  - Result: error
  - Output: unsupported inherited options `no-aes`, `no-hmac`; deprecated option `no-ripemd`.
- `agent1 patch_regression --case openssl_test_evp`
  - Result: error
  - Reason: `agent1` command is not installed in the current environment.

## Target Hardware Checks Not Run

- `openssl speed -evp sha256` before/after comparison on SOPHON SG2044.
- `perf annotate sha256_block_data_order*` on the rebuilt RISC-V binary.
- OpenSSL EVP test run on the target RISC-V host.

## Conclusion

The patch is locally schema-valid and POD-syntax-valid, and it matches the current OpenSSL RISC-V SHA-256 dispatcher model. It is not fully verified because Agent1 and target hardware regression evidence are unavailable in this environment.
