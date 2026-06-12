# 开发总结报告

**变更**: openssl-riscv64-aes-crypto  
**蓝图**: diag-20260610-002  
**日期**: 2026-06-12

## 需求概述

OpenSSL AES_encrypt 在 SG2044 (RVV 1.0 @ VLEN=128) 上以纯标量 T-table 方式执行，IPC 仅 0.878，test_bio_enc 耗时 5.3s，569 行汇编中零条向量或加密指令。需要启用 `zvkned`（RISC-V 向量加密 AES 扩展）以激活已有的 `aes-riscv64-zvkned.pl` 向量加密汇编路径。

## 实现变更

**文件**: `Configurations/10-main.conf` (2 行修改)

```diff
-        cflags           => add("-march=rv64gcv"),
-        cxxflags         => add("-march=rv64gcv"),
+        cflags           => add("-march=rv64gcv_zvkned"),
+        cxxflags         => add("-march=rv64gcv_zvkned"),
```

**分支**: `feature/riscv64-vector-enable`（与前一变更同分支）  
**提交**: `e5418d3`

## 产物路径

| 产物 | 路径 |
|------|------|
| RootCauseBlueprint | `.yuan-sheng/openssl-riscv64-aes-crypto/agent3/root-cause-blueprint.json` |
| PatchPlan | `.yuan-sheng/openssl-riscv64-aes-crypto/agent4/patch-plan.json` |
| PatchCandidate | `.yuan-sheng/openssl-riscv64-aes-crypto/agent4/patch-candidate.json` |
| PatchRegressionResult | `.yuan-sheng/openssl-riscv64-aes-crypto/agent4/patch-regression-result.json` |
| VerifiedPatchPackage | `.yuan-sheng/openssl-riscv64-aes-crypto/verified-patch-package.json` |

## 验证结果

- **Schema 校验**: ✅ PASS
- **代码审查**: ✅ PASS — 2 行修改，扩展已有 `-march` 标志
- **构建验证**: ⏳ 延迟至 SG2044 硬件
- **运行时测试**: ⏳ 延迟至 SG2044 (需确认 zvkned 硬件支持)

## 风险与后续

**风险**: MEDIUM — SG2044 C920 的 zvkned 支持未经 `/proc/cpuinfo` 或 `riscv_hwprobe` 确认。若不支持，运行时自动回退标量 T-table，无负面影响。

**后续步骤 (Agent5)**:
1. 在 SG2044 上确认 `zvkned` 支持：`grep -m1 '^isa' /proc/cpuinfo`
2. 若支持：构建并运行 `test_bio_enc`，`perf annotate AES_encrypt` 验证 `vaesem.vv`/`vaesef.vv`
3. `openssl speed aes-128-cbc` 记录性能提升（预期 ≥2×）
4. 若不支持：回退至 `rv64gcv`，`zvkned` 变更保持但无运行时效果
5. 与前一变更 (`openssl-riscv64-vector-enable`) 合并为一个 PR 提交至 upstream
