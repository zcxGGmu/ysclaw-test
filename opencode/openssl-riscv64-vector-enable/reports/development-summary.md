# 开发总结报告

**变更**: openssl-riscv64-vector-enable  
**蓝图**: diag-20250609-001  
**日期**: 2026-06-12  
**工作流**: ysclaw-agent4 (full)

## 需求概述

OpenSSL 在 SG2044 (T-Head C920v2, RVV 1.0 @ VLEN=128) 上的 `sha256_block_data_order_riscv64` 完全以标量模式运行，未使用向量硬件。`perf` 分析确认零条 `v*` 向量指令，test_evp 耗时 210s，IPC=1.88（计算受限）。

**根因**: `linux64-riscv64` 构建目标未设置 `v` 扩展标志，编译器默认生成 `rv64gc`（纯标量）代码。

## 实现变更

在 `Configurations/10-main.conf` 的 `linux64-riscv64` 目标中添加 `-march=rv64gcv` 编译标志（2 行修改）。

```diff
 "linux64-riscv64" => {
     inherit_from     => [ "linux-generic64"],
+    cflags           => add("-march=rv64gcv"),
+    cxxflags         => add("-march=rv64gcv"),
     perlasm_scheme   => "linux64",
     asm_arch         => 'riscv64',
 },
```

## 产物路径

| 阶段 | 产物 | 路径 |
|------|------|------|
| Agent3 → | RootCauseBlueprint | `.yuan-sheng/openssl-riscv64-vector-enable/agent3/root-cause-blueprint.json` |
| PatchPlan | PatchPlan | `.yuan-sheng/openssl-riscv64-vector-enable/agent4/patch-plan.json` |
| PatchCandidate | git diff + metadata | `.yuan-sheng/openssl-riscv64-vector-enable/agent4/patch-candidate.json` |
| PatchRegressionResult | 测试计划 + 状态 | `.yuan-sheng/openssl-riscv64-vector-enable/agent4/patch-regression-result.json` |
| VerifiedPatchPackage | 完整补丁包 | `.yuan-sheng/openssl-riscv64-vector-enable/verified-patch-package.json` |

## 验证结果

- **代码审查**: ✅ PASS — 2 行修改，遵循现有 ARM/PPC64 配置模式
- **Schema 校验**: ✅ PASS — 所有产物通过 `ysclaw-agent4-tools.js validate`
- **运行时测试**: ⏳ 延迟至 SG2044 硬件执行

## 预期性能提升

| 指标 | 修改前 (标量) | 修改后预期 (RVV VLEN=128) |
|------|-------------|--------------------------|
| SHA-256 吞吐量 | 基线 | ~2-4x |
| test_evp 耗时 | 210s | 估计 50-100s |
| IPC | 1.88 | ~2.5-3.0 |

## 风险与后续

1. 在 SG2044 上构建 OpenSSL：`./Configure linux64-riscv64 && make`
2. 验证 `readelf -A fips.so | grep Tag_RISCV_arch` 包含 `v`
3. 运行 `make test TESTS=test_evp` 确认所有测试通过
4. 运行 `openssl speed sha256` 记录性能提升
5. 使用 `perf annotate` 确认向量指令路径被选中
6. 向 OpenSSL upstream 提交 PR
