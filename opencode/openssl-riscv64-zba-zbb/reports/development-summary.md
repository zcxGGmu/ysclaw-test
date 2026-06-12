# 开发总结报告
**变更**: openssl-riscv64-zba-zbb **蓝图**: diag-20260611-153337

## 需求概述
SHA-256 在 test_slh_dsa 中 IPC=2.047（compute-bound），Sigma/Maj 旋转用 sllw+srlw+or（3 指令）而非 roriw（1 指令，zbb）。需追加 zba_zbb。

## 实现变更
`-march=rv64gcv_zvkned` → `-march=rv64gcv_zba_zbb_zvkned`（2 行），提交 `e4ff81c`。

## 产物路径
- agent3/root-cause-blueprint.json, agent4/{patch-plan,patch-candidate,patch-regression-result}.json
- verified-patch-package.json

## 验证结果
✅ Schema 校验通过，运行时 ⏳ SG2044。

## 风险与后续
LOW — zba_zbb 为 RVA22 强制扩展。SG2044 上验证 roriw 替换 sllw+srlw+or。
