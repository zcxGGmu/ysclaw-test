# OpenSSL RISC-V FIPS hwprobe PR Draft

## Scope

This note captures the final upstream PR material for `0.openssl_test_evp`.

Actual code change in OpenSSL:

- `include/arch/riscv_arch.h`

Observed issue:

- In Linux RISC-V FIPS builds, `OPENSSL_cpuid_setup()` is called during FIPS provider init, but `hwprobe` support was compiled out for `FIPS_MODULE`.
- As a result, provider-local RISC-V capability state remained empty unless `OPENSSL_riscvcap` was set explicitly.
- This could cause SHA dispatch inside `fips.so` to fall back to `sha256_block_data_order_riscv64` even when the runtime supported accelerated extensions.

Fix direction:

- Enable Linux RISC-V `hwprobe` support for `FIPS_MODULE` builds as well, so capability detection inside the FIPS provider matches non-FIPS libcrypto behavior.

## PR Title

`riscv: enable hwprobe capability detection in the FIPS module`

## Commit Message

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

## PR Body

```markdown
This fixes RISC-V capability detection in FIPS provider builds on Linux.

Today the FIPS provider calls `OPENSSL_cpuid_setup()` during provider
initialization, but `include/arch/riscv_arch.h` only enables RISC-V
`hwprobe` support when `FIPS_MODULE` is not defined.

That means provider-local RISC-V capability state stays empty unless
`OPENSSL_riscvcap` is set explicitly. In practice this can cause FIPS-side
SHA dispatch to fall back to `sha256_block_data_order_riscv64` even when the
runtime supports accelerated extensions.

This patch enables Linux RISC-V `hwprobe` support for `FIPS_MODULE` builds too,
so capability detection inside the FIPS provider matches the non-FIPS libcrypto
behavior.

Validation:
- Minimal RISC-V/FIPS dispatch test under `qemu-riscv64`
  - before: `V=0 ZBB=0 ... dispatch=scalar`
  - after: `V=1 ZBB=1 ... dispatch=zbb`
- Full RISC-V cross build with `enable-fips`
  - `apps/openssl`, `libcrypto.so`, and `providers/fips.so` built successfully
  - FIPS provider loaded successfully
  - `openssl info -cpusettings` reported:
    `OPENSSL_riscvcap=RV64GC_ZBA_ZBB_ZBS_V vlen:128`
  - `openssl dgst -sha256` produced the expected digest
```

## Validation Summary

### Minimal FIPS dispatch check

- Patched build:
  - `V=1 ZBB=1 ZVKB=0 ZVKNHA=0 ZVKNHB=0 VLEN=128`
  - `dispatch=zbb`
- Baseline build:
  - `V=0 ZBB=0 ZVKB=0 ZVKNHA=0 ZVKNHB=0 VLEN=0`
  - `dispatch=scalar`

### Full OpenSSL cross-build check

Build configuration used:

```text
Configure linux64-riscv64 enable-fips no-tests no-demos --cross-compile-prefix=riscv64-linux-gnu-
```

Artifacts verified:

- `apps/openssl`
- `libcrypto.so.4`
- `libssl.so.4`
- `providers/fips.so`
- `providers/fipsmodule.cnf`

Runtime checks under `qemu-riscv64`:

- FIPS provider successfully loaded
- `openssl info -cpusettings` returned:

```text
OPENSSL_riscvcap=RV64GC_ZBA_ZBB_ZBS_V vlen:128
```

- `openssl dgst -sha256` over `abc` returned:

```text
ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad
```

## Notes

- This change fixes capability detection and dispatch selection.
- It does not itself guarantee a vector SHA-256 path on all targets.
- Whether the runtime can select `zbb` or `zvkb+zvknh*` still depends on what the actual hardware exposes.
