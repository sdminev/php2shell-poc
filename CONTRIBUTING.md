# Contributing research and corrections

This project tracks public evidence about PHP2Shell (CVE-2026-109618 and CVE-2026-109619). Corrections, new primary sources, and reproducibility notes are welcome through [GitHub Issues](https://github.com/sdminev/php2shell-poc/issues).

## For a news or status update

Include:

- A direct link to the original advisory, release, research report, telemetry publication, or incident report.
- The **publication date** and, if different, the **date of the event** being reported.
- The exact claim the source makes and the evidence it publishes.
- Whether the observation is a controlled demonstration, attack attempt, confirmed compromise, patch release, or a source-analysis conclusion.
- Any scope that changes the meaning: CVE, PHP version, SAPI, configuration, and affected path.

A later post repeating an earlier claim is useful context, but should not be counted as independent confirmation without its own evidence.

## For a PoC-repository review

Include the repository URL and a **pinned commit**, the specific files supporting the claim, the stated test environment, the success condition implemented in code, and any retained results. Distinguish a request being sent from an observed server-side effect. If you independently tested it, state the exact build and controls; otherwise label the finding as a static review.

## Corrections to our own findings

Point to the sentence or table row, the source that changes it, and the narrower wording you recommend. We will date substantive changes and keep earlier reviews tied to their pinned revisions. A missing public receipt is a limit of what a reviewer can verify from that tree, not a claim that an author's underlying result is false.

Please keep issue bodies focused on evidence and avoid posting secrets or data from systems you do not control.

## Licensing contributions

By submitting original research text or diagrams for inclusion, you agree to license that contribution under [CC BY 4.0](LICENSE). Original source code and code examples submitted for inclusion use the [MIT License](LICENSE-CODE), unless an accepted file clearly states a different license. Submit only material you have the right to license; link and attribute third-party sources rather than copying their work into a contribution.
