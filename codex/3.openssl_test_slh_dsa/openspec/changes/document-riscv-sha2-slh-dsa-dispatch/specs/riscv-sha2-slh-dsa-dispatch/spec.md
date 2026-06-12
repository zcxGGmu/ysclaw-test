## ADDED Requirements

### Requirement: SHA-2 capability override examples
The `OPENSSL_riscvcap` manual page SHALL include examples for explicitly enabling RISC-V SHA-2-related Zbb and vector crypto capability combinations.

#### Scenario: Zbb SHA-2 override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for SHA-2 diagnostics
- **THEN** the documentation identifies `rv64gc_zbb` as an override useful for testing RISC-V Zbb SHA-2 paths

#### Scenario: Vector SHA-256 override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for SHA-256 diagnostics
- **THEN** the documentation identifies `rv64gc_v_zvkb_zvknha` as an override useful for testing RISC-V vector SHA-256 paths

#### Scenario: Vector SHA-512 override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for SHA-512 diagnostics
- **THEN** the documentation identifies `rv64gc_v_zvkb_zvknhb` as an override useful for testing RISC-V vector SHA-512 paths

### Requirement: Runtime dispatch scope guidance
The `OPENSSL_riscvcap` manual page SHALL state that SHA-specific overrides affect runtime-dispatched implementations and do not change compiler-selected architecture flags or the generic `linux64-riscv64` target baseline.

#### Scenario: Diagnosing scalar SLH-DSA SHA-256
- **WHEN** a user is diagnosing a scalar `sha256_block_data_order_riscv64` profile from `openssl_test_slh_dsa`
- **THEN** the documentation distinguishes runtime capability selection from compile-time `-march` policy

### Requirement: Override safety guidance
The `OPENSSL_riscvcap` manual page SHALL state that SHA-specific overrides are for explicit testing, diagnostics, and benchmarking, while normal Linux execution should prefer hwprobe-detected capabilities.

#### Scenario: Production guidance
- **WHEN** a user reads the SHA override examples
- **THEN** the documentation distinguishes diagnostic overrides from the preferred Linux hwprobe path for ordinary execution
