## ADDED Requirements

### Requirement: SHA capability override examples
The `OPENSSL_riscvcap` manual page SHALL include examples for explicitly enabling RISC-V SHA-related Zbb and vector crypto capability combinations.

#### Scenario: Zbb SHA-2 override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for SHA diagnostics
- **THEN** the documentation identifies `rv64gc_zbb` as an override useful for testing RISC-V Zbb SHA-2 paths

#### Scenario: Vector SHA-256 override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for SHA diagnostics
- **THEN** the documentation identifies `rv64gc_v_zvkb_zvknha` as an override useful for testing RISC-V vector SHA-256 paths

#### Scenario: Vector SHA-512 override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for SHA diagnostics
- **THEN** the documentation identifies `rv64gc_v_zvkb_zvknhb` as an override useful for testing RISC-V vector SHA-512 paths

### Requirement: SHA-1 dispatch scope guidance
The `OPENSSL_riscvcap` manual page SHALL state that current RISC-V SHA crypto dispatch applies to SHA-2 implementations and does not add a SHA-1 crypto dispatch path.

#### Scenario: SHA-1 diagnosis
- **WHEN** a user is diagnosing a SHA-1 profile such as `SHA1_Final`
- **THEN** the documentation distinguishes SHA-1 from the RISC-V SHA-2 dispatch paths controlled by Zbb and ZVKNH capability bits

### Requirement: Override safety guidance
The `OPENSSL_riscvcap` manual page SHALL state that SHA-specific overrides are for explicit testing, diagnostics, and benchmarking, while normal Linux execution should prefer hwprobe-detected capabilities.

#### Scenario: Production guidance
- **WHEN** a user reads the SHA override examples
- **THEN** the documentation distinguishes diagnostic overrides from the preferred Linux hwprobe path for ordinary execution
