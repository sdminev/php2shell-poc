# Advisory analysis

Reviewed: 2026-10-21. Evidence level: **PHP advisory review**; the upstream patch has been identified but not analyzed line by line, and no local execution has occurred.

## CVE-2026-109618: out of memory corruption

PHP describes an out of memory error using `longjmp()` to leave a context while request memory is temporarily accessible but incompletely initialized. A later request shutdown can consume invalid state. Multipart parsing offers several routes to memory exhaustion within apparent input limits. The PHP advisory reports confirmed unauthenticated remote code execution over HTTP with Apache mod_php. It distinguishes that result from FPM source analysis, for which it reports no complete exploit for this CVE.

Source: [PHP advisory GHSA-w8hw-w9fx-j79w](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w).

## CVE-2026-109619: stale global state

PHP describes state owned by one request remaining published in worker globals after the request memory is freed. An operation in a subsequent request, before ordinary initialization resets those globals, can consume the stale pointer. The confirmed cases include PHP 8.5 fatal error backtrace and the session upload progress lifecycle. The producing and consuming requests must reach the same worker; their precise reachability depends on SAPI and configuration. PHP reports confirmed RCE with mod_php and a session lifecycle RCE path validated with FPM.

Source: [PHP advisory GHSA-6wmf-4rv5-p3rp](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp).

## Patch review target

The upstream fix is [31c0ea7, “Avoid state corruption via bailout”](https://github.com/php/php-src/commit/31c0ea7d7ff1d1fccb6492e52dab544011628065). PHP's OOM advisory says the fix replaces the abrupt OOM bailout with delayed soft OOM at a VM safe point and a hard exit at a higher threshold. The stale state advisory describes the complete fix as resetting the relevant worker state. These descriptions need to be checked against the pinned branch diff before attributing a particular changed line to either CVE.

## Scope and remediation

| Upstream branch | First fixed release |
| --- | --- |
| 8.2 | 8.2.35 |
| 8.3 | 8.3.36 |
| 8.4 | 8.4.27 |
| 8.5 | 8.5.12 |

This table is upstream release information, not a scanner verdict. Distribution packages may have backported fixes. Older unsupported branches are not covered by a blanket vulnerability assertion here.

Upgrade to a fully patched PHP build. Narrow configuration changes can reduce exposure to specific paths but cannot replace the patch, according to both PHP advisories.
