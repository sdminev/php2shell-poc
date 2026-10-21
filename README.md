# PHP2Shell: CVE-2026-109618 and CVE-2026-109619

An evidence-first workspace for independent analysis of the PHP2Shell disclosures.

**Status (2026-10-21): advisory review only.** This repository does not yet contain a validated vulnerability PoC or independent exploit result. A PoC will be added only with recorded vulnerable and patched build results. Do not interpret the advisory summary or a version number as proof that a particular service is exploitable.

## What PHP disclosed

| ID | PHP's description | Confirmed scope in the advisory |
| --- | --- | --- |
| [CVE-2026-109618](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w) | Out of memory bailout can leave partially initialized request state for shutdown to consume. | Remote code execution over HTTP was demonstrated with Apache mod_php. PHP describes FPM susceptibility from source analysis but says a complete FPM exploit for this CVE was not demonstrated. |
| [CVE-2026-109619](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp) | A worker can retain a pointer to freed request memory; an early operation in another request can consume it. | Remote code execution was demonstrated with Apache mod_php, and a session lifecycle path was validated with FPM. The producing and consuming requests must reach the same worker. |

Both advisories assign **CVSS v4.0 9.2**. The supported upstream branches were fixed in **8.2.35, 8.3.36, 8.4.27, and 8.5.12**. Package vendors may backport fixes while retaining older version strings, so check the vendor's security notice or the actual patch before drawing a conclusion from `php -v`. Older, unsupported PHP branches need separate source and package verification; this repo makes no blanket claim about every PHP 5–8.1 build.

PHP says upgrading to a release containing the complete fix is the only reliable mitigation for these issues. Disabling `session.upload_progress` is not a complete fix for both CVEs.

## Research workflow

1. Review the two PHP advisories and [the upstream fix](https://github.com/php/php-src/commit/31c0ea7d7ff1d1fccb6492e52dab544011628065).
2. Build pinned vulnerable and patched PHP releases with the same SAPI, extensions, compiler, and configuration.
3. Run a minimal local trigger against an isolated service bound to loopback. Record a sanitizer trace or other direct memory-safety evidence, not just an HTTP error or worker restart.
4. Repeat on the patched build. Record unsuccessful attempts and environmental differences.
5. Publish a PoC only after the evidence supports the precise CVE, SAPI, preconditions, and observed impact.

See [advisory analysis](docs/advisory-analysis.md), [reproduction protocol](docs/reproduction-protocol.md), and [claim ledger](docs/claim-ledger.md).

## Source policy

PHP's advisories and source history are the authority for vulnerability identity and remediation. Third-party projects are leads for experiments, not proof of the claims in this repository. Research results must distinguish **source review**, **local observation**, **sanitizer finding**, and **demonstrated code execution**.

## Responsible use

Run experiments only on systems you own or are authorized to test. The proposed lab is local and intentionally has no remote-target mode.
