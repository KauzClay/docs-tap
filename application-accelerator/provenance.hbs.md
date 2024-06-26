# Track life cycle using Provenance transform

This topic tells you about the Application Accelerator `Provenance` transform in Tanzu Application
Platform (commonly known as TAP).

## <a id="overview"></a> Overview

Use the `Provenance` transform to track the life cycle of generated projects.

The `Provenance` transform generates a file that
provides details of the accelerator engine transform.

The `Provenance` transform provides traceability and visibility for the generation of an application
from an accelerator. The following information is embedded into a file that is part of the generated
project:

- Which accelerator was used to bootstrap the project
- Which version of the accelerator was used
- When the application was bootstrapped
- Who bootstrapped the application

For more information about the structure of the file and how to enable application bootstrapping
provenance, see [Provenance transform](creating-accelerators/transforms/provenance.hbs.md).

## <a id="amr"></a> Integration with AMR

Application Accelerator integrates with [Artifact Metadata Repository](../scst-store/overview.hbs.md) (AMR).

When you generate an application with an accelerator, an event that contains the same information
captured by the `Provenance` transform is sent to the AMR store. The relevant data saved to AMR is
`AppAcceleratorRuns` (alpha) and `AppAcceleratorFragments` (alpha).

### <a id='appacceleratorruns'></a> AppAcceleratorRuns (alpha)

The `AppAcceleratorRuns` data model represents a generation of a project from an accelerator.
An `accelerator.yaml` file in the repository declares input
options for the accelerator. This file contains instructions for processing the
files when you generate a new project. This information is sent to the
CloudEvent Handler to store `AppAcceleratorRuns`.

There is one `AppAcceleratorRuns` for each invocation of an accelerator, including version
information about which accelerator was used.
Each `AppAcceleratorRuns` data entry has a unique `guid`.
The `AppAcceleratorRuns` data entry contains information about the Git repository including, `AppAcceleratorRepoURL`, `AppAcceleratorRevision`, and `AppAcceleratorSubpath`.
Multiple `AppAcceleratorFragments` entries can point to the same `AppAcceleratorRuns` entry. `AppAcceleratorRuns` are associated with one `AppAcceleratorSource`, also known as `Commit`.

### <a id='appacceleratorfragments'></a> AppAcceleratorFragments (alpha)

The `AppAcceleratorFragments` Accelerator fragments are reusable accelerator
components that can provide options, files, or transforms. You can import
accelerators using an `import` entry. An `InvokeFragment` transform references
the transforms from the fragment in the accelerator that declares the import.
The `AppAcceleratorFragments` data model represents the information of a fragment
in the accelerator app. The Observer sends this information to the Cloud Event
Handler to store `AppAcceleratorFragments`.

Each `AppAcceleratorFragment` data entry stores information about source Git repository:
`AppAcceleratorFragmentSourceRepoURL`, `AppAcceleratorFragmentSourceRevision`, and
`AppAcceleratorFragmentSourceSubpath`. You can associate an `AppAcceleratorFragment` to
an `AppAcceleratorRuns`. You can point a `AppAcceleratorFragmentSource`
(also known as `Commit`) to one `AppAcceleratorFragment`.

There is one instance of AppAcceleratorFragment for each named fragment used by the running
accelerator.

## <a id='querying_data'></a> Querying the data

When invocations are recorded, use the
GraphQL capability to query the system about
accelerator use and gain insights about generated applications. For more information,
see [Run query with GraphQL](../scst-store/amr/graphql-query.hbs.md)

For more information about how to use the Artifact Metadata Repository,
and some sample queries relevant to `AppAcceleratorRuns`, see
the [Artifact Metadata Repository](../scst-store/overview.hbs.md#amr).
