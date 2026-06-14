# OpenSSL RISC-V FIPS hwprobe PR 草稿

## 范围

这是 `0.openssl_test_evp` 的最终 PR 材料。

实际代码改动：

- `include/arch/riscv_arch.h`

发现的问题：

- 在 Linux RISC-V 的 FIPS 构建里，`OPENSSL_cpuid_setup()` 会在 FIPS provider 初始化时被调用，但 `FIPS_MODULE` 下把 `hwprobe` 能力探测编译掉了。
- 结果是，provider 内部的 RISC-V capability 状态一直是空的，除非手动设置 `OPENSSL_riscvcap`。
- 这样会让 `fips.so` 里的 SHA 调度回退到 `sha256_block_data_order_riscv64`，即使运行时其实支持加速扩展。

修复方向：

- 让 Linux 下的 RISC-V `FIPS_MODULE` 构建也保留 `hwprobe`，使 FIPS provider 内部的能力探测和非 FIPS 的 `libcrypto` 一致。

## PR 标题

`riscv: enable hwprobe capability detection in the FIPS module`

## 提交说明

```text
riscv: enable hwprobe capability detection in the FIPS module

The FIPS provider calls OPENSSL_cpuid_setup() during provider init,
but RISC-V hwprobe capability detection was previously compiled out
for FIPS_MODULE builds.

As a result, provider-local RISC-V capability state remained empty
unless OPENSSL_riscvcap was set explicitly, which caused RISC-V SHA
dispatch in the FIPS provider to fall back to the scalar path even
when the runtime supported accelerated extensions such as Zbb/V.

Allow hwprobe support for Linux FIPS_MODULE builds as well so that
provider-local capability detection matches libcrypto behavior.

This was validated with a RISC-V cross build under qemu-riscv64:
- before the change, FIPS-module capability detection reported no
  V/ZBB support and selected the scalar dispatch path
- after the change, capability detection reported RV64GC_ZBA_ZBB_ZBS_V
  with vlen:128 and selected the Zbb path
- a full OpenSSL RISC-V build with enable-fips successfully loaded
  the FIPS provider and executed SHA-256 correctly
```

## PR 正文

```markdown
这个补丁修复的是 Linux RISC-V FIPS 构建里的能力探测问题。

当前 FIPS provider 在初始化时会调用 `OPENSSL_cpuid_setup()`，
但 `include/arch/riscv_arch.h` 只在 `FIPS_MODULE` 未定义时才启用
RISC-V `hwprobe` 支持。

这会导致 provider 内部的 RISC-V capability 状态一直为空，除非显式
设置 `OPENSSL_riscvcap`。实际结果是，FIPS 侧 SHA 调度可能回退到
`sha256_block_data_order_riscv64`，即使运行时支持加速扩展。

这个补丁让 Linux 下的 `FIPS_MODULE` 构建也保留 `hwprobe` 支持，
使 FIPS provider 内的能力探测与非 FIPS 的 libcrypto 行为一致。

验证：
- 在 `qemu-riscv64` 下做最小 FIPS 分发测试
  - 修改前：`V=0 ZBB=0 ... dispatch=scalar`
  - 修改后：`V=1 ZBB=1 ... dispatch=zbb`
- 做完整的 RISC-V 交叉构建并开启 `enable-fips`
  - `apps/openssl`、`libcrypto.so`、`providers/fips.so` 均成功生成
  - FIPS provider 成功加载
  - `openssl info -cpusettings` 输出：
    `OPENSSL_riscvcap=RV64GC_ZBA_ZBB_ZBS_V vlen:128`
  - `openssl dgst -sha256` 输出正确摘要
```

## 验证摘要

### 最小 FIPS 分发验证

- 补丁版：
  - `V=1 ZBB=1 ZVKB=0 ZVKNHA=0 ZVKNHB=0 VLEN=128`
  - `dispatch=zbb`
- 基线版：
  - `V=0 ZBB=0 ZVKB=0 ZVKNHA=0 ZVKNHB=0 VLEN=0`
  - `dispatch=scalar`

### 完整 OpenSSL 交叉构建验证

构建命令：

```text
Configure linux64-riscv64 enable-fips no-tests no-demos --cross-compile-prefix=riscv64-linux-gnu-
```

确认的产物：

- `apps/openssl`
- `libcrypto.so.4`
- `libssl.so.4`
- `providers/fips.so`
- `providers/fipsmodule.cnf`

在 `qemu-riscv64` 下的运行验证：

- FIPS provider 成功加载
- `openssl info -cpusettings` 输出：

```text
OPENSSL_riscvcap=RV64GC_ZBA_ZBB_ZBS_V vlen:128
```

- `openssl dgst -sha256` 对 `abc` 输出：

```text
ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad
```

## 备注

- 这个修改修复的是能力探测和分发选择。
- 它不保证所有目标都一定能走向量 SHA-256。
- 具体能走 `zbb` 还是 `zvkb+zvknh*`，仍然取决于真实硬件暴露的能力。
