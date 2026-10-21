# Reproduction protocol

**Status:** planned, not executed. No PoC is claimed.

## Environment record

For every experiment, record:

- Exact PHP source tag and commit, compiler and sanitizer flags, binary hash, and SAPI.
- Web server or FastCGI frontend version, PHP modules, `php.ini`, worker count, memory and input limits.
- Exact request fixture hash, test endpoint, timestamps, worker process IDs, and exit status.
- Raw sanitizer output and a redacted service log. Keep both failing and passing runs.

## Validation gates

1. Establish a normal request baseline and a deterministic worker identity in a loopback-only lab.
2. Demonstrate the claimed memory-safety failure on a pinned vulnerable build. A timeout, HTTP 500, or process exit alone is inconclusive.
3. Use the same fixture and comparable configuration on the corresponding patched build.
4. Map the observed stack and lifecycle to one advisory. Do not use a session upload progress symptom alone to label both CVEs.
5. If showing code execution, use only a harmless local marker and record the worker privilege and complete evidence chain. Do not infer RCE from a crash.

## Publication gate

For each result, state whether it is source-level reasoning, an observed crash, sanitizer-confirmed memory corruption, or code execution. Include full reproducibility details and failed trials. Do not generalize from one SAPI, version, or configuration to all PHP deployments.
