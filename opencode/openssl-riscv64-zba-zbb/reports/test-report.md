# 测试报告
**变更**: openssl-riscv64-zba-zbb **日期**: 2026-06-12

## 测试范围
追加 `zba_zbb` 到 `-march=rv64gcv_zvkned` → `rv64gcv_zba_zbb_zvkned`（2 行配置修改）。

## 执行命令
```bash
./Configure linux64-riscv64 && make
readelf -A libcrypto.so | grep Tag_RISCV_arch
openssl speed sha256
perf annotate sha256_block_data_order_riscv64
```

## 测试结果
| 测试 | 状态 |
|------|------|
| 代码审查 | ✅ PASS |
| 构建 + readelf | ⏳ SG2044 |
| openssl speed | ⏳ SG2044 |
| perf annotate | ⏳ SG2044 |

## Agent1 回归证据
```
agent1 patch_regression --case test_slh_dsa
agent1 patch_regression --case test_evp
agent1 patch_regression --case openssl_speed_sha256
agent1 patch_regression --case perf_annotate_sha256_riscv64
```
