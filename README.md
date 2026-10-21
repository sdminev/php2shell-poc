# PHP2Shell 🔬

### Public-source research on two PHP vulnerabilities

**CVE-2026-109618 · CVE-2026-109619**  
![Status: source review](https://img.shields.io/badge/status-source%20review-2563eb) ![PoC: not claimed](https://img.shields.io/badge/PoC-not%20claimed-f59e0b)

> ⭐ **Follow the trail from theory to proof.** Star this repo for PoC progress, test results, and any work-in-progress code or research utilities we share. Every update will distinguish verified results from open questions.

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

### Main findings

1. **The CVE labels are too broad.** PHP describes [109618](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w) as out-of-memory bailout corruption and [109619](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp) as stale state crossing a request boundary. Eness's sole [PoC script](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/poc_109618.py) targets a proposed upload-progress pointer issue in one request; it does not demonstrate the documented two-request 109619 path. Several of ikow's labeled chains require a separate application flaw or configuration condition.
2. **Ikow's probes overinterpret responses.** Its [OOM script](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_oom_trigger.py) treats an HTTP 500 or no response as evidence of the vulnerable condition without checking PHP's memory state, a sanitizer trace, or code execution. Its [FPM script](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_fpm_stale_state.py) similarly labels a 502 as a triggered use-after-free. Those responses have many possible causes. The [session-injection script](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_session_injection.py) asks the reader to inspect session files; the [handler-mismatch script](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_handler_mismatch.py) uses an object without the destructor needed to demonstrate its claimed execution path.
3. **Ikow's published lab is internally inconsistent.** The [setup script](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/lab/setup.sh) says it prepares PHP 8.2.34 but selects a floating 8.2 image. It replaces the checked-in [Compose file](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/lab/docker-compose.yml) with one that omits the Apache service used for the mod_php claim. The [FPM probe](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_fpm_stale_state.py) requires a large prepared session, a particular endpoint, and a single worker; the setup script does not create those prerequisites. As published, the repo is not a pinned, self-contained reproduction of those claims.
4. **Eness's script depends on missing build-specific information.** The [script](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/poc_109618.py) has no address-disclosure step. With its defaults, the calculated function address is zero; a caller must supply an address or suitable override. It assumes TLS even for an HTTP URL, infers PHP SAPI from response headers, and reports a hit by finding marker text in the response body. It contains no vulnerable-versus-patched comparison, and a miss does not produce a failing process status. These limit portability and the strength of its success check.
5. **Eness's mitigation files are not a substitute for PHP's fix.** Its [proposed patch](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/patch/session-upload-progress-counters.patch) changes upload-progress counter handling, while PHP's [upstream fix](https://github.com/php/php-src/commit/31c0ea7d7ff1d1fccb6492e52dab544011628065) addresses the broader out-of-memory and stale-state conditions. The host-wide [quick-fix script](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/quick-fix/run.sh) leaves its fixed-version list empty, so its version check cannot reliably recognize current upstream fixed releases. Its supposedly read-only check also creates temporary files and queries live FPM pools.

**Conclusion:** Both repos contain substantive research attempts, but neither published tree supplies enough reliable, reproducible evidence for this static audit to endorse a broad “both CVEs demonstrated as RCE” claim. That limits what we can conclude from the published code; it does not prove their underlying observations false.

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
