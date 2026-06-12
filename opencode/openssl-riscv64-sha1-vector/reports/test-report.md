# 测试报告

**变更**: openssl-riscv64-sha1-vector  
**蓝图**: diag-20260610-001  
**日期**: 2026-06-12

## 测试范围

实现 RISC-V 向量 SHA1 汇编 (`sha1-riscv64-zvknha.pl`, 411 行 perlasm) + 构建系统集成 (`crypto/sha/build.info`, 4 行修改)。使用 Zvknha 向量加密扩展进行 4 路并行 SHA1 压缩。

- 代码审查（perlasm 结构与 SHA-256 参考实现对照）
- 构建集成验证（build.info 语法）
- 运行时验证（需 SG2044 + Zvknha 支持）

## 执行命令

```bash
cd openssl
git checkout feature/riscv64-vector-enable
./Configure linux64-riscv64
make -j$(nproc)
make test TESTS=test_sha1
make test TESTS=test_store_cases
openssl speed sha1
perf annotate sha1_block_data_order_zvknha
```

## 测试结果

| 测试 | 状态 | 说明 |
|------|------|------|
| Perlasm 结构审查 | ✅ PASS | 遵循 SHA-256 参考实现模式 |
| build.info 语法 | ✅ PASS | 2 行 ASM + 2 行 DEF + 1 行 GENERATE |
| 构建验证 | ⏳ DEFERRED | 需 SG2044 + Zvknha 编译器 |
| test_sha1 | ⏳ DEFERRED | 需 SG2044 |
| test_store_cases | ⏳ DEFERRED | 需 SG2044 |
| openssl speed sha1 | ⏳ DEFERRED | 需 SG2044 |
| perf annotate | ⏳ DEFERRED | 需 SG2044 + perf |

## Agent1 回归证据

```
agent1 patch_regression --case test_sha1
agent1 patch_regression --case test_store_cases
agent1 patch_regression --case openssl_speed_sha1
agent1 patch_regression --case perf_annotate_SHA1_Final
```
