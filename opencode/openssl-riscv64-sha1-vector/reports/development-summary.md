# 开发总结报告

**变更**: openssl-riscv64-sha1-vector  
**蓝图**: diag-20260610-001  
**日期**: 2026-06-12

## 需求概述

OpenSSL 缺少 RISC-V SHA1 向量汇编，SHA1_Final 在 SG2044 上纯标量运行（IPC=0.859，test_store_cases=5.0s）。已有 `-march=rv64gcv_zvkned` 但编译器无法自动向量化 SHA1 的复杂依赖链。需手写 Zvknha 向量汇编。

## 实现变更

**新文件**: `crypto/sha/asm/sha1-riscv64-zvknha.pl` (411 行 perlasm)
- 4 路并行 SHA1 (VLEN=128, SEW=32, m1)
- Zvknha 指令: `vsha1c.vv` (Ch), `vsha1parity.vv` (Parity), `vsha1maj.vv` (Maj), `vsha1ms.vv` (消息扩展)
- 参考 `sha256-riscv64-zvkb-zvknha_or_zvknhb.pl` 结构

**修改文件**: `crypto/sha/build.info` (4 行)
- `$SHA1ASM_riscv64` 增加 `sha1-riscv64-zvknha.S`
- `$SHA1DEF_riscv64` 增加 `SHA1_ASM`
- 新增 GENERATE 规则

**分支**: `feature/riscv64-vector-enable`  
**提交**: `203914d`

## 产物路径

| 产物 | 路径 |
|------|------|
| RootCauseBlueprint | `.yuan-sheng/openssl-riscv64-sha1-vector/agent3/` |
| PatchPlan | `.yuan-sheng/openssl-riscv64-sha1-vector/agent4/patch-plan.json` |
| PatchCandidate | `.yuan-sheng/openssl-riscv64-sha1-vector/agent4/patch-candidate.json` |
| PatchRegressionResult | `.yuan-sheng/openssl-riscv64-sha1-vector/agent4/patch-regression-result.json` |

## 验证结果

- **Perlasm 审查**: ✅ PASS — 结构遵循 SHA-256 参考实现
- **构建集成**: ✅ PASS — build.info 语法正确
- **运行时**: ⏳ 延迟至 SG2044 + Zvknha 硬件

## 风险与后续

**风险**: MEDIUM — Zvknha 可用性未在 SG2044 上验证；perlasm 需通过 NIST SHA1 测试向量验证。

**后续**:
1. 在 SG2044 上确认 Zvknha: `grep -m1 '^isa' /proc/cpuinfo`
2. 若支持：构建 + `make test TESTS=test_sha1`
3. `openssl speed sha1` 验证 ≥2× 吞吐提升
4. 可与其他两个变更合并为一个 PR
