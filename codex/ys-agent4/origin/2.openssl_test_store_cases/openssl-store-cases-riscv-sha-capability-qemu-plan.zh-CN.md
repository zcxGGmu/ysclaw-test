# 2.openssl_test_store_cases 开发与 QEMU 验证方案

## 结论

蓝图将 `SHA1_Final` 标量热点主要归因于构建 `-march` 缺少 `v/zbb`，这个方向不适合作为本次修复方案。当前 OpenSSL 的 RISC-V SHA 汇编/分发路径只覆盖 SHA-256 和 SHA-512：`crypto/sha/sha_riscv.c` 会根据 `Zbb`、`Zvkb+Zvknha/Zvknhb` 分发 SHA-2 实现，`crypto/sha/build.info` 中没有 RISC-V SHA-1 汇编实现，也不会因为设置 `OPENSSL_riscvcap` 或强制 `-march` 自动产生 RISC-V SHA-1 crypto dispatch。

因此本次采用行为中性的文档修复：补充 `OPENSSL_riscvcap` 中 RISC-V SHA-2 诊断覆盖示例，并明确这些覆盖只影响运行时分发，不改变编译器选择的基础 ISA、ELF `Tag_RISCV_arch`，也不会启用编译期 `__riscv_zbb` 代码生成；同时说明当前 RISC-V SHA crypto dispatch 不覆盖 SHA-1。

## 代码修改

修改文件：

- `doc/man3/OPENSSL_riscvcap.pod`

补丁文件：

- `openssl-store-cases-riscv-sha-capability-doc.patch`

主要内容：

- 增加 `rv64gc_zbb`、`rv64gc_v_zvkb_zvknha`、`rv64gc_v_zvkb_zvknhb` 示例，用于 SHA-2 dispatcher 诊断/benchmark。
- 说明 capability override 不会改变编译产物的基础 RISC-V ISA。
- 说明向量长度仍由硬件检测决定。
- 明确现有 RISC-V SHA crypto dispatch 覆盖 SHA-256/SHA-512，不新增 SHA-1 路径。

## 验证环境

宿主路径：

- `K:\ai-projs\yuan-sheng\openssl`

QEMU/Docker 环境：

- Docker 镜像：`ys-store-qemu-base:with-tools`
- 关键工具：`qemu-riscv64`、`riscv64-linux-gnu-gcc`、`riscv64-linux-gnu-readelf`、`podchecker`
- 构建配置：`./Configure linux64-riscv64 no-shared --cross-compile-prefix=riscv64-linux-gnu-`
- 已构建目标：`make -j8 apps/openssl`

## 验证步骤与结果

源码一致性检查：

```bash
rg -n "SHA1|sha1|zbb|zvknh|ZVKNH|RISCV_HAS" crypto/sha include/crypto include/arch doc/man3/OPENSSL_riscvcap.pod
```

结果确认：

- `crypto/sha/sha_riscv.c` 只分发 SHA-256/SHA-512 的 RISC-V Zbb/RVV 实现。
- `crypto/sha/build.info` 的 RISC-V 条目只包含 SHA-256/SHA-512 汇编和 `SHA256_ASM/SHA512_ASM` 定义。
- SHA-1 仍走通用 `sha1_block_data_order` 路径。

POD 语法检查：

```bash
podchecker doc/man3/OPENSSL_riscvcap.pod
```

结果：

```text
doc/man3/OPENSSL_riscvcap.pod pod syntax OK.
```

RISC-V ELF ISA 标签检查：

```bash
riscv64-linux-gnu-readelf -A apps/openssl | sed -n '/Tag_RISCV_arch/p'
```

结果：

```text
Tag_RISCV_arch: "rv64i2p1_m2p0_a2p1_f2p2_d2p2_c2p0_zicsr2p0_zifencei2p0_zmmul1p0"
```

该结果确认本次修复没有把 `zbb` 或 `v` 强行写进编译目标 ISA。

QEMU 直接验证 BER PKCS#12 store case：

```bash
qemu-riscv64 -L /usr/riscv64-linux-gnu ./apps/openssl storeutl \
  -passin pass:12345 \
  test/recipes/90-test_store_cases_data/test-BER.p12
```

结果包含：

```text
Total found: 4
```

QEMU 下运行 OpenSSL 目标 recipe：

```bash
SRCTOP=. BLDTOP=. \
EXE_SHELL='qemu-riscv64 -L /usr/riscv64-linux-gnu' \
HARNESS_VERBOSE=yes \
perl test/run_tests.pl test_store_cases
```

结果：

```text
90-test_store_cases.t ..
1..3
ok 1 - checking that storeutl fails when given a garbage pkcs12 file
ok 2 - checking that storeutl didn't ask for a passphrase
ok 3 - Checking that 'openssl storeutl' with test-BER.p12 returns 4 objects
All tests successful.
Files=1, Tests=3
Result: PASS
```

说明：`make V=1 TESTS=test_store_cases test` 在当前 checkout 的 Makefile 中会先进入 `tests: build_programs_nodep`，构建大量与本 recipe 无关的 test/fuzz 程序；为避免验证被无关长耗时构建阻塞，最终采用 `test/run_tests.pl test_store_cases` 直接执行同一个 OpenSSL recipe，并通过 `EXE_SHELL` 保证 `apps/openssl` 在 QEMU RISC-V 环境下运行。

## 回归结论

本次变更是文档修复，不改变运行时代码路径。QEMU 下 `storeutl` 直接验证和 `90-test_store_cases.t` recipe 均通过；`readelf` 结果也确认没有引入蓝图中不合适的强制 `-march=v/zbb` 构建行为。
