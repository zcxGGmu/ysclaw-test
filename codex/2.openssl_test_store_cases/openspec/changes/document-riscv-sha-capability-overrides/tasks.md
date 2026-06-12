## 1. Agent4 Planning

- [x] 1.1 Normalize the Agent3 Lite diagnosis into `.yuan-sheng/document-riscv-sha-capability-overrides/agent3/root-cause-blueprint.json`
- [x] 1.2 Validate the canonical RootCauseBlueprint with the ysclaw-agent4 schema tool
- [x] 1.3 Generate and validate `.yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch-plan.json`

## 2. Documentation Patch

- [x] 2.1 Update `doc/man3/OPENSSL_riscvcap.pod` with SHA-specific RISC-V override examples
- [x] 2.2 Keep the documentation clear that overrides are diagnostic, hwprobe is preferred for normal Linux execution, and current RISC-V SHA crypto dispatch is SHA-2 scoped

## 3. Verification and Packaging

- [x] 3.1 Run available local documentation and source verification commands
- [x] 3.2 Capture the true git diff as `.yuan-sheng/document-riscv-sha-capability-overrides/agent4/patch.diff`
- [x] 3.3 Generate and validate PatchCandidate, PatchRegressionResult, Agent5 patch, VerifiedPatchPackage, test report, and development summary
