# PHP2Shell evidence tracker

**Last reviewed:** 21 October 2026  
**Scope:** CVE-2026-109618 and CVE-2026-109619, using public sources available on that date.

This tracker records **what each source establishes and what remains open**. We have not run exploit code, tested a server, or independently reproduced the vulnerabilities. “Confirmed” below means confirmed **by the cited publisher**, not by this repository. For the static audits, see the [README](../README.md#public-poc-repository-review) and the [expanded public PoC review](public-poc-review-2026-10-21.md).

## At a glance

| Topic | Status on 21 October 2026 |
| --- | --- |
| Vendor fixes | PHP released 8.2.35, 8.3.36, 8.4.27, and 8.5.12. [PHP announcement](https://www.php.net/archive/2026.php#2026-10-21-5) |
| Controlled RCE research | Reported by [PHP](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w) and [Wiz](https://www.wiz.io/blog/php2shell-php-rce), with conditions varying by CVE, variant, and SAPI. |
| Internet activity | [Cloudflare](https://blog.cloudflare.com/php-vulnerabilities-waf-protection) and [watchTowr](https://watchtowr.com/intelligence/php-remote-code-execution-php2shell/) report attempts or attack traffic. The cited material does not identify a confirmed successful intrusion. |
| This repository's PoC | None. Our contribution so far is source comparison and dated static audits. |

## Evidence labels

- **Vendor-confirmed:** PHP's published advisory or release statement.
- **Researcher-reported demonstration:** A research team's described controlled result; this repository has not reproduced it.
- **Observed attack traffic:** A telemetry publisher reports attempts. This does not alone establish successful execution or compromise.
- **Static audit:** A conclusion about published files at a pinned revision, without executing them.
- **Source analysis:** A plausible path identified in code or an advisory where the cited source does not claim a complete exploit.

## Claim matrix

| ID | CVE and PHP path | Source and date | Published evidence | Current assessment and open point |
| --- | --- | --- | --- | --- |
| **V1** | **109618 · Apache mod_php, request startup and shutdown** | [PHP advisory](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w), 21 Oct 2026 | **Vendor-confirmed:** out-of-memory bailout can leave incomplete request state that shutdown consumes; PHP reports an unauthenticated RCE demonstration before application code runs. | Confirmed in PHP's stated mod_php scope. The advisory is the authoritative source for conditions and remediation. |
| **V2** | **109618 · PHP-FPM after successful startup** | [PHP advisory](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w), 21 Oct 2026 | **Source analysis:** PHP describes FPM as susceptible in some post-startup contexts but says a complete FPM exploit for this CVE has not been demonstrated. | Do not present FPM RCE for 109618 as a completed public demonstration on this evidence. |
| **V3** | **109619 · Apache mod_php, fatal-error backtrace state** | [PHP advisory](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp), 21 Oct 2026 | **Vendor-confirmed:** a producer leaves request-owned state, and an early consumer in a later same-worker request uses it; PHP reports RCE for the PHP 8.5 fatal-backtrace case. | This is a specific confirmed path, not every possible stale-state variant. |
| **V4** | **109619 · session upload-progress lifecycle, mod_php and FPM** | [PHP advisory](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp), 21 Oct 2026 | **Vendor-confirmed:** PHP says session-lifecycle RCE was validated on mod_php and FPM. Its FPM path publishes state during application execution after startup and depends on a suitable application operation. | Keep the FPM prerequisite and same-worker request boundary attached to the claim. |
| **R1** | **Both CVEs · multiple variants and representative CMS deployments** | [Wiz Research](https://www.wiz.io/blog/php2shell-php-rce), 21 Oct 2026 | **Researcher-reported demonstration:** Wiz describes several variants grouped under the two CVEs, memory disclosure, and pre-authentication RCE demonstrations on representative WordPress, Drupal, and Joomla deployments. | Independent research supports serious impact. It does not verify the specific scripts in the two repos audited here. |
| **T1** | **Both CVEs · Internet attack traffic; SAPI not established in the report** | [Cloudflare](https://blog.cloudflare.com/php-vulnerabilities-waf-protection), 21 Oct 2026 | **Observed attack traffic:** Cloudflare says it saw attacks targeting customers and deployed WAF protections. | The report establishes observed traffic, not a named confirmed compromise. Cloudflare still recommends updating PHP. |
| **T2** | **Both CVEs · observed attempts; SAPI not established in the report** | [watchTowr](https://watchtowr.com/intelligence/php-remote-code-execution-php2shell/), 21 Oct 2026 | **Observed attack traffic:** watchTowr reports active exploitation attempts and marks the issue as exploited in the wild. | Treat “attempts observed” as the directly supported observation; the cited account does not document a successful intrusion. |
| **P1** | **Claims spanning both CVEs · ikow/php2shell-poc** | [Pinned repository](https://github.com/ikow/php2shell-poc/tree/6d2168710509a2647545ba78d68afb1e3f745038), audited 21 Oct 2026 | **Static audit:** scripts and a lab exist, but response-code checks and missing lab prerequisites do not establish the README's broad RCE outcomes. | Direct memory evidence, a pinned complete lab, and a fixed-build comparison are not published in the reviewed tree. [Detailed findings](../README.md#main-findings). |
| **P2** | **Claims spanning both CVEs · eness PoC repository** | [Pinned repository](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/tree/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11), audited 21 Oct 2026 | **Static audit:** one payload script, an unofficial patch, and mitigation files exist; the script depends on build-specific address input and an ambiguous response marker. | The reviewed tree does not provide an independently checkable vulnerable-versus-patched result or demonstrate the documented 109619 two-request path. [Detailed findings](../README.md#main-findings). |
| **P3** | **Reported remote chain · 9bie** | [Pinned repository](https://github.com/9bie/php2shell-bypass-aslr-exp/tree/4586f5cc120c82fd468b17469202bac629810ad9), audited 21 Oct 2026 | **Static audit:** driver checks a proof artifact and published logs report successful lab trials. | Conditions include a specific mod_php lab and local build offsets; no matching patched-build result was found. [Expanded review](public-poc-review-2026-10-21.md). |
| **P4** | **Trigger labs · jf0x3f** | [Pinned repository](https://github.com/jf0x3f/php2shell-pocs/tree/cc74c1897a274f28f0a65fee5847fdfa301dae7f), audited 21 Oct 2026 | **Static audit:** two bounded Docker labs and a configuration control are published. | The author scopes them to OOM and worker-abort triggers, not RCE; no patched-build comparison is included. [Expanded review](public-poc-review-2026-10-21.md). |
| **P5** | **Upload-progress stale-write claim · 0x4148** | [Pinned repository](https://github.com/0x4148/CVE-2026-109618-php2shell-lab/tree/bc40fb2c96d5e92121e6bf8b996f7ba3ad057413), audited 21 Oct 2026 | **Static audit:** split-upload script and session monitor exist. | Session data and HTTP statuses do not directly establish the claimed pointer write or worker crash. [Expanded review](public-poc-review-2026-10-21.md). |
| **P6** | **Remote-chain claim · xuehuolll** | [Pinned repository](https://github.com/xuehuolll/php2shell-CVE-2026-109618-109619/tree/63618bd7629dbfc2d5b505e9dd74e321d8345351), audited 21 Oct 2026 | **Static audit:** remote-chain script, analysis, and memory-error records exist. | The optional proof check means a sent final request may return success without server-side proof; a matched remote-chain receipt is absent from the reviewed tree. [Expanded review](public-poc-review-2026-10-21.md). |
| **P7** | **Lab-assisted and later remote claims · dinosn** | [Pinned repository](https://github.com/dinosn/php-cve-2026-10961x-lab-poc/tree/120b7b0eafee2ac5544b3092ff587032d3547ccf), audited 21 Oct 2026 | **Static audit:** compact negative and marker controls support a reported lab-assisted result; a later remote-chain driver is published. | The cited 12/12 end-to-end report is absent from this commit, and the driver does not check its final server-side marker. [Expanded review](public-poc-review-2026-10-21.md). |
| **F1** | **Both CVEs · remediation** | [PHP announcement](https://www.php.net/archive/2026.php#2026-10-21-5), 21 Oct 2026 | **Vendor-confirmed:** fixed upstream releases are 8.2.35, 8.3.36, 8.4.27, and 8.5.12. PHP says updating is the only complete solution. | For distribution builds, verify the vendor's backport status. A WAF or path-specific setting is exposure reduction, not a complete fix. |

## Questions to revisit when new evidence appears

1. Do reputable incident reports document a **successful compromise**, with enough detail to distinguish it from blocked or failed attempts?
2. Do the seven audited projects publish pinned, vulnerable-versus-fixed results with direct evidence of the failure they claim?
3. Has PHP or another primary source revised the affected scope, fixed versions, or assessment of a particular SAPI path?
4. Do later reports identify a new variant or a prerequisite that changes the attribution of an existing claim?

A new source should update the relevant row with its **publication date, exact claim, evidence type, and limitation**. A changed repository should be reviewed at a new pinned revision; the static conclusions above stay tied to the linked commits.

## Update log

- **21 October 2026:** Initial tracker compiled from PHP's advisories and release announcement, Wiz's research, Cloudflare and watchTowr reporting, and two pinned static audits.
- **21 October 2026:** Added five more pinned public-repository reviews. No local execution or exploit validation was performed.

Corrections with a primary source or a reproducible discrepancy are welcome through [GitHub Issues](https://github.com/sdminev/php2shell-poc/issues).