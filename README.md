# pull-request-processor-runner

<!-- 561-group-standards:badges:start -->
[![Red Cup Engineering](https://img.shields.io/badge/561%20Group-Red%20Cup%20Engineering-c0392b)](https://github.com/red-cup-engineering)
[![License: MIT](https://img.shields.io/badge/license-MIT-c0392b)](LICENSE)
[![Indigenous data: CARE](https://img.shields.io/badge/Indigenous%20data-CARE-4b6f44)](https://github.com/red-cup-engineering/.github/blob/main/GOVERNANCE.md)
[![Tips welcome](https://img.shields.io/badge/tips-welcome-d4a017)](https://github.com/561-group/.github/blob/main/TIPS.md)
<!-- 561-group-standards:badges:end -->

Runs the deterministic pull-request and issue processors of
[red-cup-engineering/github-engineering-services-section](https://github.com/red-cup-engineering/github-engineering-services-section)
on GitHub-hosted runners, over the five-organization matrix, so no Group host
does this work. The Group App `561-group-github-services` supplies a separate
short-lived installation token for each organization. Act selection, limits,
permissions and dry runs are defined by [the workflow](.github/workflows/process-pull-requests.yml).

## How it is triggered: events, not a clock

There is no schedule ([C1](specifications/causal-pr-processor-invocation.md)).
A run starts when it is dispatched, and a run whose acts changed something
dispatches the next one (the `continue` job). A merge moves a base, a rebase
starts checks, and a close changes duplicates, so that change is the event for
the next run, which reads the new state. The chain stops at a fixed point: the
first run that applies nothing. Rebased pull requests do not wait for a run:
GitHub's own auto-merge lands each one when its checks pass. The processor
turns that setting on where a repository lacks it (`allow-auto-merge`).

## Selected processor source

Both jobs fetch `scripts/` and `src/` of the GitHub section at the one commit
named by `PROCESSOR_COMMIT` (depth 1, blobless, sparse). The processor is pure
Node with nothing to install, so a merged fix reaches the runner by changing
that line, without waiting for a registry release. Receipts name the source
repository and commit that ran.

The `process` job applies the act flags of input `acts`, else repository
variable `PR_PROCESSOR_ACTS`, else `--label`. The `issues` job applies input
`issue-acts`, else `ISSUE_PROCESSOR_ACTS`, else `--label`, and runs while
`ISSUE_PROCESSOR_ENABLED` is `true`. Its public evidence is counts only.

## Receiving each run

Every job attempts to retain one `github.pull-request-run-receipt.v1` JSON
artifact, including when the processor fails or cannot finish observing its
selected Work. Its artifact name is
`pr-processor-ORGANIZATION-RUN_ID-RUN_ATTEMPT`, retained for 14 days. The same
sanitized receipt appears in the job summary.

The receipt binds the public runner commit, run and attempt, organization,
actual released processor version and downloaded tarball digest to the observed
step outcome, exit code, source-receipt kind, coverage status, reason-code counts
and numeric processing summaries. These are separate claims: successful process
exit does not establish complete source coverage, and a missing source receipt
is reported as missing. Upload failure remains a workflow failure. A runner
terminated before cleanup can execute cannot supply an artifact.

Consumers can list or download the artifact through the existing GitHub Actions
artifact API, correlating its run and attempt with the original duty. An
incomplete observation is useful recovery evidence, not code-development or
merge completion. The released processor supplies detailed coverage reasons
when supported; older releases still provide their available counts and status.

## Public evidence boundary

This runner is public, while many repositories it observes are private.
[Repository readers can download workflow artifacts](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/download-workflow-artifacts).
The raw `plan.json` and raw processor output remain in temporary runner storage.
They are never uploaded or printed. Only a fixed field allowlist is published:
no private repository names, PR numbers/titles/bodies, source heads, identities,
cursor values, raw errors or tokens. Reason details are reduced to known reason
codes and counts, and unknown keys or provider text are excluded.

The public aggregate supports recurring-operation monitoring. It does not carry
the private source witnesses needed to receive a particular WorkCell. That
receiving edge belongs to the existing private Work owner and its authorized
transport; this workflow does not create a new publication route for it.

