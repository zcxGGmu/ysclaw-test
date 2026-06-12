# 测试报告

**变更**: openssl-riscv64-aes-crypto  
**蓝图**: diag-20260610-002  
**日期**: 2026-06-12

## 测试范围

本变更将 `linux64-riscv64` 目标的 `-march` 从 `rv64gcv` 扩展为 `rv64gcv_zvkned`，在已有的基础向量扩展上增加 `zvkned` 向量加密 AES 扩展（2 行配置修改）。

- 代码审查（diff 检查）
- 构建验证（需 SG2044 RISC-V 交叉编译环境）
- 运行时正确性验证（需 SG2044 硬件 + zvkned 支持）

## 执行命令

```bash
cd openssl
git checkout feature/riscv64-vector-enable
./Configure linux64-riscv64
make -j$(nproc)
make test TESTS=test_bio_enc
openssl speed aes-128-cbc
readelf -A libcrypto.so.4 | grep Tag_RISCV_arch
perf annotate AES_encrypt
```

## 测试结果

| 测试 | 状态 | 说明 |
|------|------|------|
| 代码审查（diff 检查） | ✅ PASS | 2 行变更 (rv64gcv → rv64gcv_zvkned) |
| 构建验证 | ⏳ DEFERRED | 需 RISC-V 交叉编译器 + zvkned 支持 |
| `readelf -A libcrypto.so.4` | ⏳ DEFERRED | 需目标硬件构建 |
| `test_bio_enc` | ⏳ DEFERRED | 需 SG2044 构建环境 |
| `openssl speed aes-128-cbc` | ⏳ DEFERRED | 需 SG2044 硬件 |
| `perf annotate AES_encrypt` | ⏳ DEFERRED | 需 SG2044 硬件 + perf |
| 标量 T-table 回退 | ⏳ DEFERRED | 需非 zvkned RISC-V 硬件 |

### 风险评估

- **风险等级**: MEDIUM（SG2044 zvkned 支持未确认）
- **影响代码**: 单个构建配置文件，2 行
- **向后兼容**: 通过 `crypto/riscvcap.c` 运行时调度完全保留
- **前序变更**: 依赖已应用的 `rv64gcv` 基础向量扩展

## Agent1 回归证据

```
agent1 patch_regression --case test_bio_enc
agent1 patch_regression --case openssl_speed_aes128cbc
agent1 patch_regression --case readelf_check_libcrypto_arch
agent1 patch_regression --case perf_annotate_AES_encrypt
```
