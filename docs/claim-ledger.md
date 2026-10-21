# Claim ledger

| Claim | Current evidence | Status |
| --- | --- | --- |
| PHP tracks two distinct issues as CVE-2026-109618 and CVE-2026-109619. | [PHP OOM advisory](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w), [PHP stale state advisory](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp) | Published by upstream; not independently reproduced here |
| Both issues have upstream fixes in PHP 8.2.35, 8.3.36, 8.4.27, and 8.5.12. | Same upstream advisories | Published by upstream; package backports require separate checks |
| The OOM case permits RCE on mod_php. | [PHP OOM advisory](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w) | Reported and validated by PHP; not independently reproduced here |
| The stale state case permits RCE on mod_php and has a validated FPM session path. | [PHP stale state advisory](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp) | Reported and validated by PHP; not independently reproduced here |
| This repository has a working PoC for either CVE. | No local execution environment or evidence | **Not established** |
| All PHP versions since 5.4 share the same exploitability. | Not established by the supported-branch advisories | **Do not claim** |
