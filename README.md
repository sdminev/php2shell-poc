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

## The two public repositories

### [ikow/php2shell-poc](https://github.com/ikow/php2shell-poc)

Its README calls the mod_php path “a pre-auth, zero-dependency RCE against default Apache mod_php installations.” The repository includes five [probe scripts](https://github.com/ikow/php2shell-poc/tree/main/poc) and a [lab setup](https://github.com/ikow/php2shell-poc/tree/main/lab). In our static review, several scripts classify HTTP errors or missing responses as signs of a vulnerability; those observations alone cannot establish memory corruption or code execution. The lab setup also uses a floating PHP image despite naming a specific vulnerable version, and its setup script replaces the checked-in service configuration. Some other proposed chains require a separate application flaw or configuration condition.

### [eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC)

Its README says “Confirmed on standard `php-src` builds (including versions 8.4.4 and 8.2.34).” The repository contains one [Python PoC script](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/main/poc_109618.py), an [unofficial source patch](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/main/patch/session-upload-progress-counters.patch), and mitigation files. The script relies on build-specific address information that it does not obtain itself, and it does not demonstrate the two-request state transition described for CVE-2026-109619. The proposed patch addresses upload-progress counters rather than the full set of conditions in PHP's official advisories.

## Our assessment

These are **published research attempts**, not results we have verified. Their code and documentation provide leads, but the broad claims about both CVEs and working RCE exceed what our read-only review can establish. This repository will distinguish PHP's confirmed advisory statements, the projects' claims, and any independently observed result before presenting a PoC.
