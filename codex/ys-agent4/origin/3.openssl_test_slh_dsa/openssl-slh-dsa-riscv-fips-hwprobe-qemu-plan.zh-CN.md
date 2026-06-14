# OpenSSL `3.openssl_test_slh_dsa` 修复与 QEMU 验证方案

## 结论

蓝图将 `test_slh_dsa` 的 RISC-V 热点归因于通用 `linux64-riscv64` 构建缺少 `-march=rv64gc_zba_zbb`。这个方向不宜直接采用：当前 OpenSSL 已经编译了 RISC-V SHA-256 的 RV64 baseline、Zbb、以及 `ZVKB+ZVKNHA/ZVKNHB` 向量实现，并由 `crypto/sha/sha_riscv.c` 在运行时按 capability 分发。

实际问题在 FIPS provider 路径：`providers/fips/fipsprov.c` 初始化时会调用 `OPENSSL_cpuid_setup()`，但 `include/arch/riscv_arch.h` 原先在 `FIPS_MODULE` 下排除了 Linux `hwprobe` 支持。结果是 `fips.so` 内部的 RISC-V capability 状态为空，除非手动设置 `OPENSSL_riscvcap`，否则 SLH-DSA SHA2 profile 会回落到 `sha256_block_data_order_riscv64`。

本次修复让 Linux RISC-V 的 FIPS module 同样编入 `hwprobe` capability detection，使 FIPS provider 与 libcrypto 的运行时分发行为一致。

## 代码改动

文件：

- `include/arch/riscv_arch.h`
- `doc/man3/OPENSSL_riscvcap.pod`

改动：

- 去掉 RISC-V Linux `hwprobe` 代码上的 `!defined(FIPS_MODULE)` 限制。
- 增加注释，说明 FIPS provider 也需要 provider-local capability detection。
- 在 `OPENSSL_riscvcap` 文档中补充 SHA-2 dispatcher diagnostic override 示例，强调这些 override 不改变 ELF `Tag_RISCV_arch` 或 compiler base ISA。

补丁文件：

- `openssl-slh-dsa-riscv-fips-hwprobe.patch`

## QEMU 验证环境

容器：

- Docker `ubuntu:24.04`

依赖：

- `gcc-riscv64-linux-gnu`
- `binutils-riscv64-linux-gnu`
- `libc6-dev-riscv64-cross`
- `linux-libc-dev-riscv64-cross`
- `qemu-user`
- `make`
- `perl`
- `python3`

工具版本：

```text
qemu-riscv64 version 8.2.2 (Debian 1:8.2.2+ds-0ubuntu1.16)
riscv64-linux-gnu-gcc (Ubuntu 13.3.0-6ubuntu2~24.04.1) 13.3.0
```

构建配置：

```bash
perl Configure linux64-riscv64 enable-fips no-shared no-ssl no-quic no-cmp no-cms --cross-compile-prefix=riscv64-linux-gnu-
make build_generated
make -j4 apps/openssl providers/fips.so providers/fipsmodule.cnf test/slh_dsa_test
```

说明：`no-ssl` 在当前版本中会提示 deprecated，但本次验证只覆盖 libcrypto/FIPS/SLH-DSA 路径。

## 验证结果

构建产物：

```text
/tmp/openssl-slh-crypto/apps/openssl
/tmp/openssl-slh-crypto/providers/fips.so
/tmp/openssl-slh-crypto/providers/fipsmodule.cnf
/tmp/openssl-slh-crypto/test/slh_dsa_test
```

`fips.so` ISA 属性保持通用 RISC-V baseline，未强制提高 generic target 要求：

```text
Tag_RISCV_arch: "rv64i2p1_m2p0_a2p1_f2p2_d2p2_c2p0_zicsr2p0_zifencei2p0_zmmul1p0"
```

`fips.so` 同时包含 SHA-256 baseline、Zbb、vector SHA-256 符号：

```text
sha256_block_data_order
sha256_block_data_order_riscv64
sha256_block_data_order_zbb
sha256_block_data_order_zvkb_zvknha_or_zvknhb
```

`fips.so` 中确认编入 RISC-V hwprobe 相关状态和系统接口：

```text
OPENSSL_cpuid_setup
OPENSSL_riscv_hwcap_P
OPENSSL_riscvcap_P
riscv_vlen
riscv_vlen_asm
U getauxval@GLIBC_2.27
U syscall@GLIBC_2.27
```

QEMU 暴露的 capability：

```bash
qemu-riscv64 -L /usr/riscv64-linux-gnu ./apps/openssl info -cpusettings
```

结果：

```text
OPENSSL_riscvcap=RV64GC_ZBA_ZBB_ZBS_V vlen:128
```

默认 provider SLH-DSA 测试：

```bash
qemu-riscv64 -L /usr/riscv64-linux-gnu ./test/slh_dsa_test
```

结果：`1..10` 全部通过。

FIPS provider SLH-DSA 测试：

```bash
OPENSSL_CONF_INCLUDE=providers OPENSSL_MODULES=providers \
qemu-riscv64 -L /usr/riscv64-linux-gnu ./test/slh_dsa_test -config test/fips-and-base.cnf
```

结果：`1..10` 全部通过。

FIPS provider 加载确认：

```bash
OPENSSL_CONF=test/fips-and-base.cnf OPENSSL_CONF_INCLUDE=providers OPENSSL_MODULES=providers \
qemu-riscv64 -L /usr/riscv64-linux-gnu ./apps/openssl list -providers
```

结果：

```text
base  status: active
fips  status: active
```

FIPS SHA-256 sanity check：

```bash
printf abc | OPENSSL_CONF=test/fips-and-base.cnf OPENSSL_CONF_INCLUDE=providers OPENSSL_MODULES=providers \
qemu-riscv64 -L /usr/riscv64-linux-gnu ./apps/openssl dgst -sha256
```

结果：

```text
SHA2-256(stdin)= ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad
```

## 风险与回归建议

- 本补丁不提高 `linux64-riscv64` 的默认 ISA baseline，因此不会要求所有 RISC-V Linux 用户具备 Zbb/RVV。
- 实际走 Zbb 还是 vector SHA-2 仍取决于内核 `hwprobe` 暴露的真实 capability 和 VLEN。
- 建议在 SG2044 实机上补跑 `test_slh_dsa`、`openssl speed -evp sha256`、以及 perf annotate，确认热点从 `sha256_block_data_order_riscv64` 切换到可用的加速路径。
