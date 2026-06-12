## ADDED Requirements

### Requirement: AES capability override examples
The `OPENSSL_riscvcap` manual page SHALL include examples for explicitly enabling RISC-V AES scalar crypto and vector crypto capability combinations.

#### Scenario: Scalar AES override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for AES diagnostics
- **THEN** the documentation identifies the `rv64gc_zknd_zkne` override as a scalar AES crypto test configuration

#### Scenario: Vector AES override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for AES diagnostics
- **THEN** the documentation identifies the `rv64gc_v_zvkb_zvkned` override as a vector AES test configuration for modes such as CBC, CTR, ECB, CFB, and OFB

#### Scenario: Vector AES-GCM override
- **WHEN** a user reads the `OPENSSL_riscvcap` examples for AES diagnostics
- **THEN** the documentation identifies the `rv64gc_v_zvkb_zvkg_zvkned` override as a vector AES-GCM test configuration

### Requirement: Override safety guidance
The `OPENSSL_riscvcap` manual page SHALL state that AES-specific overrides are for explicit testing, diagnostics, and benchmarking, while normal Linux execution should prefer hwprobe-detected capabilities.

#### Scenario: Production guidance
- **WHEN** a user reads the AES override examples
- **THEN** the documentation distinguishes diagnostic overrides from the preferred Linux hwprobe path for ordinary execution
