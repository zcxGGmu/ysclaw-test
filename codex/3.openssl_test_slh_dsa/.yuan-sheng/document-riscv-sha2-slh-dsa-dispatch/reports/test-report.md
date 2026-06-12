# 测试报告

Change: `document-riscv-sha2-slh-dsa-dispatch`

## 测试范围

- Documentation syntax for `doc/man3/OPENSSL_riscvcap.pod`.
- Source consistency between new SHA-specific `OPENSSL_riscvcap` examples, current RISC-V SHA-2 dispatch symbols, and the generic `linux64-riscv64` target context.
- Required Agent1 regression command for `openssl_test_slh_dsa`.
- Agent4 and Agent5 JSON artifact schema validation.

## 执行命令

```bash
node /Users/zq/.codex/plugins/cache/personal/ysclaw-agent4/0.1.0/tools/ysclaw-agent4-tools.js validate root-cause-blueprint .yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/agent3/root-cause-blueprint.json
node /Users/zq/.codex/plugins/cache/personal/ysclaw-agent4/0.1.0/tools/ysclaw-agent4-tools.js validate patch-plan .yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/agent4/patch-plan.json
node /Users/zq/.codex/plugins/cache/personal/ysclaw-agent4/0.1.0/tools/ysclaw-agent4-tools.js validate patch-candidate .yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/agent4/patch-candidate.json
node /Users/zq/.codex/plugins/cache/personal/ysclaw-agent4/0.1.0/tools/ysclaw-agent4-tools.js validate patch-regression-result .yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/agent4/patch-regression-result.json
node /Users/zq/.codex/plugins/cache/personal/ysclaw-agent4/0.1.0/tools/ysclaw-agent4-tools.js validate verified-patch-package .yuan-sheng/document-riscv-sha2-slh-dsa-dispatch/agent5/verified-patch-package.json
PATH=/Users/zq/Desktop/ai-projs/posp/yuan-sheng/opencode-agent4/node_modules/.bin:$PATH openspec validate document-riscv-sha2-slh-dsa-dispatch
podchecker doc/man3/OPENSSL_riscvcap.pod
git diff --check
rg -n "rv64gc_zbb|rv64gc_v_zvkb_zvknha|rv64gc_v_zvkb_zvknhb|sha256_block_data_order_zbb|sha256_block_data_order_zvkb_zvknha_or_zvknhb|sha512_block_data_order_zvkb_zvknhb|RISCV_HAS_ZVKB_AND_ZVKNHB|linux64-riscv64" crypto include doc/man3/OPENSSL_riscvcap.pod Configurations/10-main.conf
agent1 patch_regression --case openssl_test_slh_dsa
```

## 测试结果

- Agent4/Agent5 JSON schema validation: pass. Output: each validate command returned `{ "valid": true }`.
- `PATH=/Users/zq/Desktop/ai-projs/posp/yuan-sheng/opencode-agent4/node_modules/.bin:$PATH openspec validate document-riscv-sha2-slh-dsa-dispatch`: pass. Output: `Change 'document-riscv-sha2-slh-dsa-dispatch' is valid`.
- `podchecker doc/man3/OPENSSL_riscvcap.pod`: pass. Output: `doc/man3/OPENSSL_riscvcap.pod pod syntax OK.`
- `git diff --check`: pass. No whitespace errors were reported.
- RISC-V SHA-2 source consistency search: pass. Output confirmed the new documentation examples, existing SHA-2 dispatch symbols, and the generic `linux64-riscv64` target context.
- Local OpenSSL test binary check: no configured build tree is present in this sparse worktree. `Makefile`, `configdata.pm`, and `test/slh_dsa_test` are missing, so `make test TESTS=test_slh_dsa` was not runnable locally.
- `agent1 patch_regression --case openssl_test_slh_dsa`: error. Output: `zsh:1: command not found: agent1`.

## Agent1 回归证据

Agent1 is not installed or not available on PATH in this environment, so the required regression could not execute. The normalized `PatchRegressionResult` records `status: error`, and the generated `VerifiedPatchPackage` records `verification.status: failed`.

Required target-side follow-up:

```bash
agent1 patch_regression --case openssl_test_slh_dsa
make test TESTS=test_slh_dsa
make test TESTS=test_sha
readelf -A ./providers/fips.so | grep Tag_RISCV_arch
./apps/openssl info -cpusettings
grep -m1 '^isa' /proc/cpuinfo
perf annotate -s sha256_block_data_order_riscv64
perf annotate -s sha256_block_data_order_zbb
```
