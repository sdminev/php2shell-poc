# PHP2Shell 🔬

### Public-source research on two PHP vulnerabilities

**CVE-2026-109618 · CVE-2026-109619**  
![Status: source review](https://img.shields.io/badge/status-source%20review-2563eb) ![PoC: not claimed](https://img.shields.io/badge/PoC-not%20claimed-f59e0b)

This repository records a **read-only review of public information** as of 2026-10-21. We reviewed the two projects linked below and PHP's published advisories. We did not execute their code, test a server, or independently reproduce either vulnerability. There is no working PoC in this repository.

## What PHP has published

| CVE | PHP's description | Upstream advisory |
| --- | --- | --- |
| **CVE-2026-109618** | Out-of-memory handling can leave request state invalid for shutdown to consume. PHP reports demonstrated remote code execution with Apache mod_php. | [RCE via OOM corruption](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w) |
| **CVE-2026-109619** | A worker can retain stale request-owned state that a later request consumes. PHP reports demonstrated mod_php exploitation and a validated FPM session path. | [RCE via stale global state](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp) |

The first fixed upstream releases are **PHP 8.2.35, 8.3.36, 8.4.27, and 8.5.12**. PHP says upgrading to a build with the complete fix is the only reliable mitigation. Distribution backports require a package-specific check. [Upstream fix](https://github.com/php/php-src/commit/31c0ea7d7ff1d1fccb6492e52dab544011628065)

## What the two example repositories actually publish

This section is a **static source audit dated 2026-10-21**. We read the published code and lab files; we did not execute either project, send test requests, or verify their reported outcomes. The links below pin the reviewed revisions.

| Repository | Its published claim | What the code supports |
| --- | --- | --- |
| [ikow/php2shell-poc](https://github.com/ikow/php2shell-poc/tree/6d2168710509a2647545ba78d68afb1e3f745038) | “This is a pre-auth, zero-dependency RCE against default Apache mod_php installations.” ([README](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/README.md)) | Five probe scripts and a lab exist. Their response checks and published lab do not, by themselves, establish the README's RCE outcomes. |
| [eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/tree/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11) | “Confirmed on standard `php-src` builds (including versions 8.4.4 and 8.2.34).” ([README](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/README.md)) | One payload-building Python script, an unofficial patch, and mitigation files exist. The repository does not include a complete, independently checkable vulnerable-versus-patched result. |

### Findings that matter for a PoC

1. **Keep the CVEs separate.** PHP assigns [109618](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w) to out-of-memory bailout corruption and [109619](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp) to stale state consumed across requests. Eness's [single-request script](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/poc_109618.py) does not establish the documented 109619 path. Ikow's [session injection](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_session_injection.py), [deserialization](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_uaf_asan.php), and [handler mismatch](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_handler_mismatch.py) experiments have additional application or configuration conditions.
2. **Do not infer corruption from HTTP status.** Ikow's [OOM probe](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_oom_trigger.py) treats a 500 or missing response as evidence of the vulnerable condition; its [FPM probe](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_fpm_stale_state.py) treats a 502 as a use-after-free. These symptoms do not identify the underlying cause.
3. **Pin and complete the lab.** Ikow's [setup script](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/lab/setup.sh) says PHP 8.2.34 but selects a floating image, replaces the checked-in [Compose file](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/lab/docker-compose.yml) without its Apache service, and does not create the large session fixture or single-worker configuration required by its FPM probe.
4. **Account for build-specific inputs and reliable success checks.** Eness's [script](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/poc_109618.py) has no address-disclosure step; its default calculated function address is zero. It assumes TLS, infers SAPI from response headers, and treats marker text in the response as success without excluding an application echo.
5. **Do not present partial mitigations as fixes.** Eness's [unofficial patch](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/patch/session-upload-progress-counters.patch) changes upload-progress counters, whereas PHP's [upstream fix](https://github.com/php/php-src/commit/31c0ea7d7ff1d1fccb6492e52dab544011628065) also addresses bailout and request-state handling. Its [host-wide quick-fix script](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/quick-fix/run.sh) has an empty fixed-version list; its “read-only” check creates temporary files and queries live FPM pools.

## Path from research to a working PoC

| Audit gap | Required research result before making a claim |
| --- | --- |
| CVE attribution is mixed | Choose one CVE and one confirmed SAPI path; map the observation to the matching PHP advisory and upstream change. Treat auxiliary application-dependent chains separately. |
| HTTP errors stand in for memory evidence | Capture a direct trace of the relevant memory or lifecycle failure, with worker identity and full build details. A status code alone remains inconclusive. |
| Lab versions and prerequisites are unstable | Pin vulnerable and fixed source revisions or immutable images; include every required endpoint, fixture, configuration value, and worker setting in a reproducible local lab. |
| Build-specific address assumptions are undocumented | State exactly how each required input was obtained for that build. Do not claim portability or a default working exploit without that evidence. |
| Success checks can be ambiguous | Use an unmistakable local observation tied to the server-side effect, rule out echoed input, and make failures distinguishable from successes. |
| No patch comparison is published | Apply the same minimal stimulus to corresponding vulnerable and fixed builds and retain both results, including unsuccessful trials. |
| Mitigation advice is incomplete | Compare any proposed workaround with both PHP advisories; label partial exposure reduction as such and use the complete upstream fix for remediation claims. |

A future PoC entry should include its exact scope, pinned build and configuration, input fixture, observed trace, fixed-build comparison, and limitations. Label the result **crash**, **sanitizer-confirmed memory corruption**, or **code execution** according to the evidence actually collected. Until then, this repository remains research and audit documentation.
