# Lab plan and safe review guide

**Status:** planning and static review only. This guide describes how to assess a published lab and plan an independent comparison;

The goal is to answer a narrow question: **does the evidence published by a project support the outcome it claims, and what would an independent lab need to observe to confirm it?** Start with the [seven-repository source review](public-poc-review-2026-10-21.md) and PHP's [CVE-2026-109618](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w) and [CVE-2026-109619](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp) advisories.

## Plan an independent comparison

1. **Choose one claim.** Write down the CVE, PHP version, SAPI, claimed effect, and exact revision of the project being assessed. A crash, sanitizer report, disclosure, and code execution are separate outcomes. Do not combine auxiliary application flaws with a PHP-only claim.
2. **Inventory the published lab before using it.** Read its README, container definitions, setup scripts, target application, runner, dependency files, and evidence directory. Identify downloaded images, privileged settings, mounted host paths, persistent files, network exposure, and any script that changes host configuration. Do not treat a repository title or a successful process exit as a result.
3. **Design a disposable environment.** Use a local VM or equivalent isolated host with a snapshot. Keep services reachable only inside the lab, prevent access to unrelated systems, use synthetic data, and avoid real credentials. Give the lab a clear cleanup boundary. A public repository may contain code that has effects beyond its stated demonstration.
4. **Pin the comparison.** Record immutable image digests or exact PHP source commits, web-server and SAPI versions, relevant configuration, worker model, allocator or sanitizer settings, and hashes of the files under review. Prepare a corresponding PHP build containing the vendor fix; a feature-disabled configuration is a useful additional control, not the same comparison. PHP's [release announcement](https://www.php.net/archive/2026.php#2026-10-21-5) identifies fixed release lines.
5. **Decide what would count as evidence before any future run.** Specify a normal-request baseline, a negative control, the direct observation needed for the claimed outcome, and the expected result on the fixed build. Capture timestamps, worker identity, logs, and failures as well as successes. An HTTP error by itself does not establish memory corruption or execution.
6. **Report only the observed level.** Separate author-provided logs from independently obtained records. State the exact build and conditions, trial counts, unsuccessful attempts, and any missing evidence. The existing [reproduction protocol](reproduction-protocol.md) lists the record and publication fields.

This is a plan, not a claim that the vulnerable or fixed cases have been reproduced here.

## Which public labs help answer which question?

| Reviewed project | Useful starting point for source review | Check before trusting or reproducing its claim |
| --- | --- | --- |
| [ikow/php2shell-poc](https://github.com/ikow/php2shell-poc/tree/6d2168710509a2647545ba78d68afb1e3f745038) | Comparing several probe ideas and lab prerequisites. | Its setup uses a floating image and does not create every endpoint, fixture, or worker condition the probes need. HTTP 500/502 checks are inconclusive. |
| [eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/tree/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11) | Reviewing build-dependent assumptions and the difference between a proposed workaround and PHP's fix. | Its payload script lacks an address-disclosure stage and a robust default proof check. Review the host-wide quick-fix script's writes before considering any use. |
| [9bie/php2shell-bypass-aslr-exp](https://github.com/9bie/php2shell-bypass-aslr-exp/tree/4586f5cc120c82fd468b17469202bac629810ad9) | Studying a published multi-stage lab claim with a proof check and retained successes and misses. | Pin the image and offsets for the specific build; distinguish local proof-file access from the HTTP request path. A matched patched-build result is absent. |
| [jf0x3f/php2shell-pocs](https://github.com/jf0x3f/php2shell-pocs/tree/cc74c1897a274f28f0a65fee5847fdfa301dae7f) | Studying explicitly scoped trigger claims and an internal-network Compose layout. | The source shows intended stimuli, while worker failure is author-reported. Its configuration control is not a patched PHP build. |
| [0x4148/CVE-2026-109618-php2shell-lab](https://github.com/0x4148/CVE-2026-109618-php2shell-lab/tree/bc40fb2c96d5e92121e6bf8b996f7ba3ad057413) | Studying the timing of an upload-progress observation in an FPM lab. | The session check occurs before the alleged later stale write. A response failure cannot identify a freed-pointer write. |
| [xuehuolll/php2shell-CVE-2026-109618-109619](https://github.com/xuehuolll/php2shell-CVE-2026-109618-109619/tree/63618bd7629dbfc2d5b505e9dd74e321d8345351) | Comparing a remote-chain implementation, provenance notes, and an instrumented memory trace. | Its ASan trace covers a different environment from the remote mod_php claim; the proof check is optional in the remote script. Auxiliary application paths have separate prerequisites. |
| [dinosn/php-cve-2026-10961x-lab-poc](https://github.com/dinosn/php-cve-2026-10961x-lab-poc/tree/120b7b0eafee2ac5544b3092ff587032d3547ccf) | Reading retained lab-assisted negative controls and marker verdicts. | Keep the lab-assisted record separate from the later no-debugger claim. The latter cites absent reports, and its driver does not check the final server-side marker. |

## Short guide to reviewing a public lab safely

- **Read first:** inspect all files that could execute or modify the host, including setup and cleanup scripts. Review container mounts, ports, privileges, and network settings.
- **Keep the boundary local:** any later independent check belongs on systems you own or are authorized to test, inside a disposable environment. Public example targets are not test targets.
- **Use a matched control:** keep PHP build, SAPI, configuration, and application behavior comparable between vulnerable and fixed cases. Record exceptions rather than hiding them.
- **Inspect the result, not the label:** a sent request, HTTP error, or zero exit status is only that observation. Ask for a direct trace or unambiguous server-side effect suited to the claim.
- **Publish the limit:** say whether your conclusion comes from static source review, an author's retained record, or your own independent observation.

For the source-level findings behind these checks, see the [full review](public-poc-review-2026-10-21.md). This guide is intentionally non-operational; anyone independently evaluating public code must review it and the applicable authorization and environment before deciding whether to run it.
