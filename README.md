# PHP2Shell 🔬

### Public-source research on two critical PHP vulnerabilities

**CVE-2026-109618 · CVE-2026-109619**  
![Status: source review](https://img.shields.io/badge/status-source%20review-2563eb) ![PoC: not claimed](https://img.shields.io/badge/PoC-not%20claimed-f59e0b)

> ⭐ **Follow the evidence as it develops.** Star this repo to bookmark and support the research; use **Watch** for GitHub notifications. Expect source-linked findings, test results, and any work-in-progress code or research utilities we publish, with verified results clearly separated from open questions.

This repository collects **public-source research and a static audit** of two third-party projects. The audit is dated **21 October 2026**. We did not execute either project's code, send test requests, or independently reproduce either vulnerability. There is no working PoC in this repository.

**Start here:** [PHP2Shell evidence tracker](docs/evidence-tracker.md) — a dated, source-linked map of vendor findings, research demonstrations, observed attack traffic, and the open questions behind each claim.

## Contents

- [Evidence tracker](docs/evidence-tracker.md)
- [Current status](#current-status)
- [What PHP has published](#what-php-has-published)
- [Independent research and observed activity](#independent-research-and-observed-activity)
- [What the two example repositories actually publish](#what-the-two-example-repositories-actually-publish)
  - [Main findings](#main-findings)
- [How to read the evidence](#how-to-read-the-evidence)
- [Path from research to a working PoC](#path-from-research-to-a-working-poc)
- [Sources and update policy](#sources-and-update-policy)

## Current status

**Snapshot: 21 October 2026.** This is a dated summary, not a live feed.

| Question | What the published evidence says |
| --- | --- |
| Are fixes available? | **Yes.** PHP released **8.2.35, 8.3.36, 8.4.27, and 8.5.12** with fixes for both CVEs. See [PHP's security announcement](https://www.php.net/archive/2026.php#2026-10-21-5). |
| Is remote code execution demonstrated? | **Yes, in controlled research.** PHP reports a confirmed pre-application mod_php path for 109618 and mod_php exploitation plus a validated FPM session path for 109619. [Wiz](https://www.wiz.io/blog/php2shell-php-rce) reports pre-authentication RCE demonstrations against representative WordPress, Drupal, and Joomla deployments. These reports do not validate the two third-party repos audited below. |
| Has attack activity been seen? | **Yes, attempts have been reported.** [Cloudflare](https://blog.cloudflare.com/php-vulnerabilities-waf-protection) says it saw attack traffic targeting customers; [watchTowr](https://watchtowr.com/intelligence/php-remote-code-execution-php2shell/) reports exploitation attempts. The cited reports do not establish a confirmed successful intrusion. |
| Is a WAF a complete fix? | **No.** [Cloudflare](https://blog.cloudflare.com/php-vulnerabilities-waf-protection) deployed protections and still recommends updating PHP. PHP says an updated runtime is the only complete remediation. |

PHP also says **8.1 and older receive no regular PHP-project releases**. Some distribution or LTS vendors may backport fixes, so package status must be checked with the vendor rather than inferred from a PHP version string alone. [PHP security announcement](https://www.php.net/archive/2026.php#2026-10-21-5)

## What PHP has published

| CVE | PHP's description | Demonstrated scope in PHP's advisory |
| --- | --- | --- |
| **CVE-2026-109618** | An out-of-memory bailout can interrupt request setup while internal state is incomplete; shutdown may then use invalid state. | [The advisory](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w) reports RCE on Apache mod_php. It describes FPM susceptibility from source analysis but says a complete FPM exploit for this CVE has not been demonstrated. |
| **CVE-2026-109619** | Request-owned state can remain in a worker after its backing memory is freed; a later request on that worker can consume it. | [The advisory](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp) describes a producer in one request and a consumer in another, and reports mod_php exploitation plus a validated FPM session path. |

The advisories describe **classes of failure with multiple variants**, not a single interchangeable payload. Conditions differ by variant and PHP SAPI. The [upstream fix](https://github.com/php/php-src/commit/31c0ea7d7ff1d1fccb6492e52dab544011628065) and patched releases address the underlying handling. Configuration changes may reduce exposure to particular paths, but PHP does not present them as a complete substitute for updating.

## Independent research and observed activity

| Source | Published finding | Evidence boundary |
| --- | --- | --- |
| [Wiz Research](https://www.wiz.io/blog/php2shell-php-rce) | Describes several variants grouped under the two CVEs, including memory disclosure and controlled RCE demonstrations on representative CMS deployments. | A research-team demonstration; it does not independently verify either repository audited here. |
| [Cloudflare](https://blog.cloudflare.com/php-vulnerabilities-waf-protection) | Reports attack traffic against customers and deployment of default WAF protections on 21 October. | Observed attempts and protection deployment; the post does not identify a confirmed successful intrusion. |
| [watchTowr](https://watchtowr.com/intelligence/php-remote-code-execution-php2shell/) | Reports active exploitation attempts and marks the issue as exploited in the wild. | Its published account supports the existence of attempts; it does not supply a confirmed compromise case in the material cited here. |

The distinction matters: **a vulnerability demonstration**, **attack traffic**, and **a confirmed compromise** are different findings. This page records each at the level its cited source supports.

## What the two example repositories actually publish

This section is a **static source audit dated 2026-10-21**. We read the published code and lab files; we did not execute either project, send test requests, or verify their reported outcomes. The links below pin the reviewed revisions.

| Repository | Its published claim | What the code supports |
| --- | --- | --- |
| [ikow/php2shell-poc](https://github.com/ikow/php2shell-poc/tree/6d2168710509a2647545ba78d68afb1e3f745038) | “This is a pre-auth, zero-dependency RCE against default Apache mod_php installations.” ([README](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/README.md)) | Five probe scripts and a lab exist. Their response checks and published lab do not, by themselves, establish the README's RCE outcomes. |
| [eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/tree/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11) | “Confirmed on standard `php-src` builds (including versions 8.4.4 and 8.2.34).” ([README](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/README.md)) | One payload-building Python script, an unofficial patch, and mitigation files exist. The repository does not include a complete, independently checkable vulnerable-versus-patched result. |

### Main findings

1. **Repository CVE attribution needs narrower evidence.** PHP's advisories describe [109618](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w) as out-of-memory bailout corruption and [109619](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp) as stale state crossing a request boundary. Each CVE covers more than one possible variant, as [Wiz's research](https://www.wiz.io/blog/php2shell-php-rce) makes clear. Eness's sole [PoC script](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/poc_109618.py) proposes an upload-progress pointer issue in one request; the published script does not establish the producer-and-consumer sequence PHP documents for 109619. Several of ikow's labeled chains require a separate application flaw or configuration condition. These are limits of those published scripts, not limits of the CVEs themselves.
2. **Ikow's probes overinterpret responses.** Its [OOM script](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_oom_trigger.py) treats an HTTP 500 or no response as evidence of the vulnerable condition without checking PHP's memory state, a sanitizer trace, or code execution. Its [FPM script](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_fpm_stale_state.py) similarly labels a 502 as a triggered use-after-free. Those responses have many possible causes. The [session-injection script](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_session_injection.py) asks the reader to inspect session files; the [handler-mismatch script](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_handler_mismatch.py) uses an object without the destructor needed to demonstrate its claimed execution path.
3. **Ikow's published lab is internally inconsistent.** The [setup script](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/lab/setup.sh) says it prepares PHP 8.2.34 but selects a floating 8.2 image. It replaces the checked-in [Compose file](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/lab/docker-compose.yml) with one that omits the Apache service used for the mod_php claim. The [FPM probe](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_fpm_stale_state.py) requires a large prepared session, a particular endpoint, and a single worker; the setup script does not create those prerequisites. As published, the repo is not a pinned, self-contained reproduction of those claims.
4. **Eness's script depends on missing build-specific information.** The [script](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/poc_109618.py) has no address-disclosure step. With its defaults, the calculated function address is zero; a caller must supply an address or suitable override. It assumes TLS even for an HTTP URL, infers PHP SAPI from response headers, and reports a hit by finding marker text in the response body. It contains no vulnerable-versus-patched comparison, and a miss does not produce a failing process status. These limit portability and the strength of its success check.
5. **Eness's mitigation files are not a substitute for PHP's fix.** Its [proposed patch](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/patch/session-upload-progress-counters.patch) changes upload-progress counter handling, while PHP's [upstream fix](https://github.com/php/php-src/commit/31c0ea7d7ff1d1fccb6492e52dab544011628065) addresses the broader out-of-memory and stale-state conditions. The host-wide [quick-fix script](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/quick-fix/run.sh) leaves its fixed-version list empty, so its version check cannot reliably recognize current upstream fixed releases. Its supposedly read-only check also creates temporary files and queries live FPM pools.

**Audit conclusion:** Both repos contain substantive research attempts, but neither published tree supplies enough reliable, reproducible evidence for this static audit to endorse a broad “both CVEs demonstrated as RCE” claim. That limits what we can conclude from the published code; it does not prove their underlying observations false. Independent demonstrations reported by PHP and Wiz are separate evidence.

## How to read the evidence

| Evidence | What it can support |
| --- | --- |
| HTTP 500, 502, timeout, or worker exit | An observable failure worth investigating; it does not identify a memory bug or prove execution. |
| Reproducible sanitizer trace or direct lifecycle evidence on a pinned vulnerable build | A specific memory or state failure, if the trace is tied to the relevant CVE path. |
| Same input on a corresponding patched build | Evidence that the observation depends on the repaired behavior, subject to matching configuration and other changes. |
| Unambiguous server-side effect with input echo ruled out | A stronger basis for a code-execution claim, when the full build and test conditions are disclosed. |

Our audit assesses **what the published repository trees establish**. It is not a judgment on unpublished results or on the independent demonstrations above.

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

## Sources and update policy

- **Primary remediation source:** [PHP security announcement](https://www.php.net/archive/2026.php#2026-10-21-5), [109618 advisory](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w), [109619 advisory](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp), and [upstream fix](https://github.com/php/php-src/commit/31c0ea7d7ff1d1fccb6492e52dab544011628065).
- **Independent research and field reporting:** [Wiz Research](https://www.wiz.io/blog/php2shell-php-rce), [Cloudflare](https://blog.cloudflare.com/php-vulnerabilities-waf-protection), and [watchTowr](https://watchtowr.com/intelligence/php-remote-code-execution-php2shell/).
- **Audited projects:** [ikow/php2shell-poc at the reviewed revision](https://github.com/ikow/php2shell-poc/tree/6d2168710509a2647545ba78d68afb1e3f745038) and [eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC at the reviewed revision](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/tree/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11).

Status statements are dated. Future updates should link the exact source and date, distinguish a report from our own verification, and identify whether an observation is a crash, memory error, information disclosure, or code execution. The pinned source audit remains tied to the revisions above; material changes to either project require a new review.
