# pull-request-processor-runner

<!-- 561-group-standards:badges:start -->
[![Red Cup Engineering](https://img.shields.io/badge/561%20Group-Red%20Cup%20Engineering-c0392b)](https://github.com/red-cup-engineering)
[![License: MIT](https://img.shields.io/badge/license-MIT-c0392b)](LICENSE)
[![Indigenous data: CARE](https://img.shields.io/badge/Indigenous%20data-CARE-4b6f44)](https://github.com/red-cup-engineering/.github/blob/main/GOVERNANCE.md)
[![Tips welcome](https://img.shields.io/badge/tips-welcome-d4a017)](https://github.com/561-group/.github/blob/main/TIPS.md)
<!-- 561-group-standards:badges:end -->

Runs the released deterministic pull request processor twice an hour on the
existing five-organization matrix. The existing GitHub App supplies a separate
short-lived installation token for each organization. Act selection, limits,
permissions and manual dry-run behavior are defined by
[the workflow](.github/workflows/process-pull-requests.yml).

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

