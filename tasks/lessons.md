# Lessons

## 2026-06-12: Agent4 diagnostic tasks require code-level optimization unless explicitly scoped otherwise

- When the user asks to complete root-cause-blueprint development tasks, do not satisfy the request with behavior-neutral documentation patches alone.
- If the validated diagnosis points at a performance/code path problem, produce a code-level optimization or explicitly stop and get confirmation before downgrading scope.
- Agent4 artifacts must reflect the actual implementation scope; `patch.diff` should contain source changes when the requested task is code optimization.
- If hardware or Agent1 verification is unavailable, keep verification status failed, but still distinguish "unverified code optimization" from "documentation-only fallback".
