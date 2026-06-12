# 测试报告

Change: `document-riscv-aes-capability-overrides`

## 测试范围

- Documentation syntax for `doc/man3/OPENSSL_riscvcap.pod`.
- Source consistency between new AES-specific `OPENSSL_riscvcap` examples and existing RISC-V AES symbols and provider dispatch.
- Required Agent1 regression command for `openssl_test_bio_enc`.

## 执行命令

```bash
podchecker doc/man3/OPENSSL_riscvcap.pod
rg -n "rv64gc_zknd_zkne|rv64gc_v_zvkb_zvkned|rv64gc_v_zvkb_zvkg_zvkned|rv64i_zkne_encrypt|rv64i_zvkned_encrypt|RISCV_HAS_ZVKNED|cipher_aes_hw_rv64i.inc" crypto include providers doc/man3/OPENSSL_riscvcap.pod
agent1 patch_regression --case openssl_test_bio_enc
```

## 测试结果

- `podchecker doc/man3/OPENSSL_riscvcap.pod`: pass. Output: `doc/man3/OPENSSL_riscvcap.pod pod syntax OK.`
- RISC-V AES source consistency search: pass. Output confirmed the new documentation examples and existing `rv64i_zkne_encrypt`, `rv64i_zvkned_encrypt`, `RISCV_HAS_ZVKNED`, and `cipher_aes_hw_rv64i.inc` references.
- `agent1 patch_regression --case openssl_test_bio_enc`: error. Output: `zsh:1: command not found: agent1`.

## Agent1 回归证据

Agent1 is not installed or not available on PATH in this environment, so the required regression could not execute. The normalized `PatchRegressionResult` records `status: error`, and the generated `VerifiedPatchPackage` records `verification.status: failed`.

Required target-side follow-up:

```bash
agent1 patch_regression --case openssl_test_bio_enc
./apps/openssl info -cpusettings
OPENSSL_riscvcap="rv64gc_v_zvkb_zvkned" make test TESTS=test_bio_enc
OPENSSL_riscvcap="rv64gc_zknd_zkne" make test TESTS=test_bio_enc
```
