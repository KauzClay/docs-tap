# View scan status conditions for Supply Chain Security Tools - Scan

This topic explains how you can view scan status conditions for Supply Chain Security Tools (SCST) -
Scan.

{{> 'partials/scst-scan/scan-1-0-deprecation' }}

## <a id='view-scan-stat'></a> Viewing scan status

You can view the scan status by running `kubectl describe` on a `SourceScan` or `ImageScan`. You can
see information about the scan status under the `Status` field for each scan CR.

## <a id='understand-con'></a> Overview of conditions

The `Status.Conditions` array is populated with the scan status information during and after
scanning execution, and the policy validation (if defined for the scan) after the results are
available.

### <a id='con-type-scan'></a> Condition types for the scans

Condition types for the scans include `Scanning`, `Succeeded`, `SendingResults`, and
`PolicySucceeded`.

#### <a id='scanning'></a> Overview of `Scanning`

The condition type `Scanning` describes the running of the scanning `TaskRun`. The `Status` field
indicates whether the scan is running or has already finished. For example, if you see
`Status: True`, the scan `TaskRun` is still running. If you see `Status: False`, the scan is done.

The `Reason` field is `JobStarted` while the scan is running and `JobFinished` when it is done.

The `Message` field can either be `The scan job is running` or `The scan job terminated` depending
on the current `Status` and `Reason`.

#### <a id='succeeded'></a> Overview of `Succeeded`

The condition type `Succeeded` describes the scanning `TaskRun` result. The `Status` field
indicates whether the scan finished successfully or if it encountered an error. For example, the
status is `Status: True` if it completed successfully and `Status: False` otherwise.

The `Reason` field is `JobFinished` if the scanning was successful and `Error` if otherwise.

The `Message` and `Error` fields have more information about the last seen status of the scan
`TaskRun`.

#### <a id='send-results'></a> Overview of `SendingResults`

The condition type `SendingResults` describes the sending of the scan results to the Metadata Store.

The condition can indicate that results were sent successfully, that the Metadata Store integration
has not been configured, or that there was an error sending the results.

An error is usually because of a misconfigured Metadata Store URL or the Metadata Store being
otherwise inaccessible. Verify the installation steps to ensure that the configuration is correct
regarding secrets being set within the `scan-link-system` namespace.

#### <a id='policy-succeed'></a> Overview of `PolicySucceeded`

The condition type `PolicySucceeded` describes the compliance of the scanning results against the
defined policies. For more information, see
[Code Compliance Policy Enforcement using Open Policy Agent (OPA)](policies.hbs.md). The `Status`
field indicates whether the results are compliant or not (`Status: True` or `Status: False`
respectively) or `Status: Unknown` if an error occurred during policy verification.

The `Reason` field is `EvaluationPassed` if the scan complies with the defined policies. The `Reason`
field is `EvaluationFailed` if the scan is not compliant and `Error` if something went wrong.

The `Message` and `Error` fields are populated with `An error has occurred` and an error message if
something went wrong during policy verification. Otherwise, the `Message` field displays
`No CVEs were found that violated the policy` if there no non-compliant vulnerabilities were found
or `Policy violated because of X CVEs` if unique vulnerabilities were found.

## <a id='understand-cvecount'></a> Overview of `CVECount`

The `status.CVECount` is populated with the number of CVEs in each category (`CRITICAL`, `HIGH`, `MEDIUM`,
`LOW`, `UNKNOWN`) and the total (`CVETOTAL`).

> **Note** You can also view scan the CVE summary in print columns by running `kubectl get` on a
> `SourceScan` or `ImageScan`.

## <a id='understand-metaurl'></a> Overview of `MetadataURL`

The `status.metadataURL` is populated with the URL of the vulnerability scan results in the
Metadata Store integration. This is only available when the integration is configured.

## <a id='understand-phase'></a> Overview of `Phase`

The `status.phase` field is populated with the current phase of the scan. The phases are `Pending`,
`Scanning`, `Completed`, `Failed`, and `Error`:

- `Pending`: Initial phase of the scan.
- `Scanning`: Execution of the scan `TaskRun` is running.
- `Completed`: Scan completed and no CVEs were found that violated the scan policy.
- `Failed`: Scan completed but CVEs were found that violated the scan policy.
- `Error`: Indication of an error (for example, an invalid `scantemplate` or scan policy).

> **Note** The PHASE print column also shows this by running `kubectl get` on a `SourceScan` or
> `ImageScan`.

## <a id='understand-scannedby'></a> Overview of `ScannedBy`

The `status.scannedBy` field is populated with the name, vendor, and scanner version that generates
the security assessment report.

## <a id='understand-scannedat'></a> Overview of `ScannedAt`

The `status.scannedAt` field is populated with the latest date when the scanning finishes.