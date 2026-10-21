# Public PHP2Shell PoC repositories: a static source review

**Review date:** 21 October 2026  
**Scope:** Seven public repositories, at the pinned commits below.  
**Method:** Read published READMEs, scripts, lab configuration, and selected evidence files. We did **not** run code, send requests, or reproduce results.

This is a review of **published evidence**, not a verdict on unpublished work. A repo can contain a useful trigger lab without proving code execution; a lab report can support a narrower result than its headline; and a source file that sends a final request does not, by itself, prove what happened on the server. The [README's main findings](../README.md#main-findings) give more detail on ikow and eness.

## How to use this review

Start with the **claim** in a project's README, then inspect the linked implementation and success check. Ask whether the code confirms the claimed effect or only submits a request, receives an HTTP error, or reads an application-controlled response. Next, compare the claimed scope with the published lab: PHP version, SAPI, configuration, worker model, build-specific inputs, and any application endpoint. Finally, look for a corresponding fixed-build result and retained evidence. The table below summarizes that reading; it is not an execution report.

| Evidence type | What it can establish |
| --- | --- |
| Request code and an HTTP status | The request shape and an observable response; not the cause of a crash or code execution. |
| PHP/Apache log plus worker identity | A failure or worker transition in the stated lab; still not necessarily control of memory. |
| Sanitizer trace linked to a request | A specific memory error on the instrumented build. |
| Negative control and patched-build comparison | Whether an observed effect depends on the suspected path and disappears with the fix. |
| Unique server-side marker, with a pre-check and no echoed input | Stronger evidence of execution in that lab; portability remains a separate question. |
| A recorded result from another author | A published claim to assess, not independent verification by this repository. |

## Repositories in scope

All seven repositories are listed here at the **exact revision reviewed**. "Published evidence" describes what is committed or author-reported; it is not an independent reproduction.

| Repository and pinned revision | Headline scope | Assessment of the published tree |
| --- | --- | --- |
| [ikow/php2shell-poc](https://github.com/ikow/php2shell-poc/tree/6d2168710509a2647545ba78d68afb1e3f745038) | Default mod_php pre-auth RCE and several related probes. | Response-based probes and an incomplete lab do not establish the headline execution claim. [Detailed findings](../README.md#main-findings). |
| [eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/tree/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11) | A payload script presented under both CVEs and supporting mitigation files. | Build-specific address input and a direct success check are missing from the default path; the published files do not establish both CVEs. [Detailed findings](../README.md#main-findings). |
| [9bie/php2shell-bypass-aslr-exp](https://github.com/9bie/php2shell-bypass-aslr-exp/tree/4586f5cc120c82fd468b17469202bac629810ad9) | Reported pure-HTTP mod_php RCE chain and repeatability logs. | A multi-stage driver, proof check, and retained successes and misses make the stated lab result inspectable; offsets, configuration, and lack of a patched control limit broader claims. |
| [jf0x3f/php2shell-pocs](https://github.com/jf0x3f/php2shell-pocs/tree/cc74c1897a274f28f0a65fee5847fdfa301dae7f) | Explicitly scoped OOM and cross-request trigger reproductions. | Code constructs the two stimuli and a contained lab; the reported outcomes are triggers and worker abort, with no RCE claim or patched-build control. |
| [0x4148/CVE-2026-109618-php2shell-lab](https://github.com/0x4148/CVE-2026-109618-php2shell-lab/tree/bc40fb2c96d5e92121e6bf8b996f7ba3ad057413) | Timed upload said to demonstrate a stale-pointer write; no RCE claim. | The recorded session check occurs before the alleged later write, and HTTP failures cannot identify a worker crash or pointer corruption. |
| [xuehuolll/php2shell-CVE-2026-109618-109619](https://github.com/xuehuolll/php2shell-CVE-2026-109618-109619/tree/63618bd7629dbfc2d5b505e9dd74e321d8345351) | Remote mod_php chain plus auxiliary paths and lab records. | A remote-chain script and ASan evidence exist, but the checked-in trace covers a different instrumented path and the default remote result lacks a matched success receipt. |
| [dinosn/php-cve-2026-10961x-lab-poc](https://github.com/dinosn/php-cve-2026-10961x-lab-poc/tree/120b7b0eafee2ac5544b3092ff587032d3547ccf) | Lab-assisted RCE, then a reported no-debugger 12/12 chain. | Committed negative and marker controls support assessment of the narrower lab-assisted report; the later report references absent records and its driver confirms sending, not the final server-side effect. |

## Repo-by-repo findings

### ikow/php2shell-poc

The [README's execution claim](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/README.md) exceeds what its probes establish. The [OOM probe](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_oom_trigger.py) interprets a 500 or missing response, and the [FPM probe](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/poc/verify_fpm_stale_state.py) interprets a 502, as vulnerability evidence without a direct memory trace. The [setup script](https://github.com/ikow/php2shell-poc/blob/6d2168710509a2647545ba78d68afb1e3f745038/lab/setup.sh) also leaves key lab prerequisites unresolved. See the [five detailed findings in the README](../README.md#main-findings).

### eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC

The [single-request script](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/poc_109618.py) does not establish the documented two-request 109619 path. It lacks an address-disclosure stage and a robust server-side success check. The [counter patch](https://github.com/eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC/blob/e8db5ee1252ea3d6d084e3e421c6eb7a380efc11/patch/session-upload-progress-counters.patch) addresses only part of the wider [upstream fix](https://github.com/php/php-src/commit/31c0ea7d7ff1d1fccb6492e52dab544011628065). See the [five detailed findings in the README](../README.md#main-findings).

### 9bie/php2shell-bypass-aslr-exp

- **The committed driver supports a more specific claim than a simple crash probe.** The [English README](https://github.com/9bie/php2shell-bypass-aslr-exp/blob/4586f5cc120c82fd468b17469202bac629810ad9/readme_en.md) reports a pure-HTTP chain on a PHP 8.4.26 mod_php lab. The [driver](https://github.com/9bie/php2shell-bypass-aslr-exp/blob/4586f5cc120c82fd468b17469202bac629810ad9/exploit.py) separates leak and final stages, removes an existing proof artifact before its final check, and returns a failure if neither the proof file nor fetched output confirms the result. This makes its lab outcome more inspectable than a response-code-only verdict.
- **The retained run log includes misses.** The [published log](https://github.com/9bie/php2shell-bypass-aslr-exp/blob/4586f5cc120c82fd468b17469202bac629810ad9/logs/pure_http_aslr_20261021.txt) reports 5/5 successes without initial rotation and 9/10 with rotation, rather than only displaying successful runs. These remain author-generated records; this review did not verify the trials.
- **The lab and offsets bound portability.** The [Compose file](https://github.com/9bie/php2shell-bypass-aslr-exp/blob/4586f5cc120c82fd468b17469202bac629810ad9/docker/docker-compose.yml) uses a mutable PHP image tag, a single worker, and a mounted proof path. The [driver](https://github.com/9bie/php2shell-bypass-aslr-exp/blob/4586f5cc120c82fd468b17469202bac629810ad9/exploit.py) derives offsets from the local container through [offsets.py](https://github.com/9bie/php2shell-bypass-aslr-exp/blob/4586f5cc120c82fd468b17469202bac629810ad9/docker/offsets.py), or requires suitable offsets for another build. “Pure HTTP” describes the request chain; the published proof still relies on lab-specific setup. No corresponding patched-build result was found in the reviewed tree.

### jf0x3f/php2shell-pocs

- **The README states a narrower goal.** Its [README](https://github.com/jf0x3f/php2shell-pocs/blob/cc74c1897a274f28f0a65fee5847fdfa301dae7f/README.md) calls these trigger reproductions and does not claim RCE. The [109618 script](https://github.com/jf0x3f/php2shell-pocs/blob/cc74c1897a274f28f0a65fee5847fdfa301dae7f/trigger_cve_2026_109618.py) constructs a bounded multipart/header stimulus and checks target health; the source shows the intended trigger, while the reported OOM outcome remains an author observation.
- **The 109619 path is structured as two requests.** The [second script](https://github.com/jf0x3f/php2shell-pocs/blob/cc74c1897a274f28f0a65fee5847fdfa301dae7f/trigger_cve_2026_109619.py) sends a producer request followed by a consumer request and prints worker information. It does not itself assert the reported `zend_mm_heap corrupted` result or prove that both requests reached the same worker from a retained trace.
- **Containment and control are present but limited.** [Compose](https://github.com/jf0x3f/php2shell-pocs/blob/cc74c1897a274f28f0a65fee5847fdfa301dae7f/compose.yaml) places targets on internal networks and includes a configuration control using `fatal_error_backtraces=0`. The [runner](https://github.com/jf0x3f/php2shell-pocs/blob/cc74c1897a274f28f0a65fee5847fdfa301dae7f/run_poc.py) scopes operation to that lab. The control is not a corresponding patched PHP build, and the reviewed tree contains no committed sanitizer trace or run transcript that independently establishes a memory primitive.

### 0x4148/CVE-2026-109618-php2shell-lab

- **The timing of the observation matters.** The [Python script](https://github.com/0x4148/CVE-2026-109618-php2shell-lab/blob/bc40fb2c96d5e92121e6bf8b996f7ba3ad057413/scripts/php2shell_poc.py) sends a split upload, reads session progress during the pause, and then sends the remaining body. It sets its “armed” state from the session read *before* the alleged later stale-pointer write. The [progress endpoint](https://github.com/0x4148/CVE-2026-109618-php2shell-lab/blob/bc40fb2c96d5e92121e6bf8b996f7ba3ad057413/webroot/check_progress.php) confirms session visibility, not a write through freed memory.
- **The failure verdict is ambiguous.** The script treats a missing response or HTTP 502/503 as a crash but retains no worker identity, direct pointer trace, or sanitizer output. The [README](https://github.com/0x4148/CVE-2026-109618-php2shell-lab/blob/bc40fb2c96d5e92121e6bf8b996f7ba3ad057413/README.md) explicitly disclaims an RCE result; even its narrower stale-write wording requires stronger evidence.
- **The comparison changes configuration rather than PHP version.** The [Dockerfile](https://github.com/0x4148/CVE-2026-109618-php2shell-lab/blob/bc40fb2c96d5e92121e6bf8b996f7ba3ad057413/Dockerfile) selects a tag rather than an immutable digest. The lab's mitigated option disables upload progress, which can test that feature's role but does not compare a vendor-fixed build. Its single-request timing also does not establish the cross-request sequence described in the [109619 advisory](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp).

### xuehuolll/php2shell-CVE-2026-109618-109619

- **Implementation and proof are different parts of the tree.** The [remote script](https://github.com/xuehuolll/php2shell-CVE-2026-109618-109619/blob/63618bd7629dbfc2d5b505e9dd74e321d8345351/exploit/cve/remote_cve.py) implements leak and final stages. Its proof-file check is optional; without that option, sending the final stage prints a prompt to inspect the proof file and exits successfully. A successful process exit therefore does not, by default, confirm the [README's](https://github.com/xuehuolll/php2shell-CVE-2026-109618-109619/blob/63618bd7629dbfc2d5b505e9dd74e321d8345351/README.md) remote execution claim.
- **The strongest retained trace covers a narrower environment.** The [ASan record](https://github.com/xuehuolll/php2shell-CVE-2026-109618-109619/blob/63618bd7629dbfc2d5b505e9dd74e321d8345351/evidence/asan-bgctx.txt) shows a use-after-free report in an instrumented PHP Development Server/CLI lab. That supports a memory finding in that environment; it does not itself demonstrate the claimed remote mod_php result. The [PoC run record](https://github.com/xuehuolll/php2shell-CVE-2026-109618-109619/blob/63618bd7629dbfc2d5b505e9dd74e321d8345351/evidence/poc-runs.txt) does not close that gap.
- **Auxiliary chains have extra dependencies.** The [shell-demo record](https://github.com/xuehuolll/php2shell-CVE-2026-109618-109619/blob/63618bd7629dbfc2d5b505e9dd74e321d8345351/evidence/shell-demo.txt) uses a MiniCMS include path, an application condition distinct from the PHP CVEs alone. [PROVENANCE.md](https://github.com/xuehuolll/php2shell-CVE-2026-109618-109619/blob/63618bd7629dbfc2d5b505e9dd74e321d8345351/PROVENANCE.md) usefully records source hashes and build setups. A matched remote-script transcript, server-side receipt, and fixed-build control would let readers assess the headline claim more directly.

### dinosn/php-cve-2026-10961x-lab-poc

- **The committed lab-assisted record is substantial but specific.** [VALIDATION.md](https://github.com/dinosn/php-cve-2026-10961x-lab-poc/blob/120b7b0eafee2ac5544b3092ff587032d3547ccf/evidence/VALIDATION.md) and a retained [VERDICT.json](https://github.com/dinosn/php-cve-2026-10961x-lab-poc/blob/120b7b0eafee2ac5544b3092ff587032d3547ccf/evidence/validation-20261020T190429Z/VERDICT.json) document negative checks and a marker result. The verdict labels the result **lab-assisted RCE**: debugger-derived addresses and tracked/system allocator conditions are part of its scope. The [source manifest](https://github.com/dinosn/php-cve-2026-10961x-lab-poc/blob/120b7b0eafee2ac5544b3092ff587032d3547ccf/SOURCE-SHA256SUMS) improves provenance. These are published records, not an independent run by this review.
- **The later no-debugger claim has a different evidence boundary.** [END-TO-END.md](https://github.com/dinosn/php-cve-2026-10961x-lab-poc/blob/120b7b0eafee2ac5544b3092ff587032d3547ccf/docs/END-TO-END.md) reports 12/12 across eight ASLR generations and references `e2e/REPORT.md` and `FINAL-VERDICT.md`; those reports were absent from the pinned tree. The [end-to-end driver](https://github.com/dinosn/php-cve-2026-10961x-lab-poc/blob/120b7b0eafee2ac5544b3092ff587032d3547ccf/e2e/nb_pure.py) prints `CHAIN_SENT` and returns success after the final request without checking a server-side marker. That confirms submission, not the claimed final effect.
- **Documents need scope reconciliation.** [PROOF-BOUNDARY.md](https://github.com/dinosn/php-cve-2026-10961x-lab-poc/blob/120b7b0eafee2ac5544b3092ff587032d3547ccf/docs/PROOF-BOUNDARY.md) still says a portable no-debugger chain is unproven and does not say it is historical. It may predate the later claim, but readers cannot reconcile the two from labels alone. The FPM stale-state result is described as crash/DoS, a separate outcome from the RCE chain. Publishing the referenced compact end-to-end receipt and dating the proof-boundary note would clarify the current claim.

**Review boundary:** These findings describe the pinned public trees. Missing public evidence does not show that an author's private or later result is false.

## Reading, using, and evaluating a public PoC

1. **Read the whole tree first.** Check the exact revision, README, runner, target configuration, dependencies, and any code that writes files, changes host settings, or accepts arbitrary targets. A repository title is not an outcome.
2. **Identify the measured effect.** A trigger, a worker abort, a sanitizer finding, information disclosure, and code execution are different results. Check whether the success branch observes the claimed effect or simply reports a sent request or HTTP status.
3. **Match the environment.** Record PHP build and SAPI, image digest or source revision, allocator, relevant INI settings, worker count, endpoint behavior, and address inputs. A lab result is scoped to those conditions.
4. **Use an owned, isolated lab if independently evaluating a claim.** Keep the target and any proof artifact inside that lab. Review what the runner will do before any future execution. This review did not perform that step.
5. **Compare controls.** Require a baseline, a negative case that should not produce the effect, and a corresponding fixed build. A feature-disabled configuration is useful but does not stand in for the upstream fix.
6. **Retain observations, including misses.** Preserve build identity, logs, worker identity, sanitizer output where relevant, and exact success evidence. Distinguish a published author's report from a result independently reproduced by another reviewer.

For remediation, follow [PHP's release announcement](https://www.php.net/archive/2026.php#2026-10-21-5) and the [109618](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w) and [109619](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp) advisories. A PoC's local workaround is not a substitute for a vendor-fixed PHP build.

## Selection and update policy

We searched public repositories using “php2shell” and the two CVE IDs, then selected projects with distinct mechanisms or evidence styles and inspectable published files. The search returned many similarly named repositories; this review is **representative, not exhaustive**. We did not review forks as independent demonstrations simply because they have a different owner or title.

All conclusions are pinned to the commits linked above. A later commit, report, or patched-build result can change an assessment; update the relevant row with a new revision and review date.