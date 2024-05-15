# Contributing

Guide for teams contributing docs to the Supply Chain component section.

## Top section

* Beneath the title, start with an `In this section` paragraph such as:

  ```console
  The section introduces...
  The topics in this section explain...
  ```

* Try to use a product name where practical

This helps with SEO.

## Naming

The base product is called `Tanzu Supply Chain`. It's especially important not to
shorten this to `Supply Chain` as it becomes too easily confused with the resource kind.

Resource Kinds should be referred to by their CamelCase name:

* SupplyChain
* Component
* WorkloadRun

## Words and replacements

| Misused | Prefer | why?                                                                                                                                                       |
|---------|--------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Steps   | Stage  | The term is `stages` in a supply chain, so use "stages" when referring to supply chains. Obviously tekton has steps, and that's a good time to use `steps` |
|         |        |                                                                                                                                                            |
|         |        |                                                                                                                                                            |

## Headings and verbs

Each H1 heading in a topic and TOC heading should, where practical start with a verb.
For example,
`Supply Chain Authoring` should be `Author a supply chain`
`How to create Workload` should be `Create a workload`

## Diagrams

The Spark boards are exported from Miro. They will need restyling since they no longer match.

* [Spark Board](https://lucid.app/lucidspark/9b937249-2925-444a-b938-cdc87ca63ebc/edit?view_items=xRgMARDgDguVA&invitationId=inv_ff75f9dc-d1e8-4ef6-93b0-65e959e8a0db)

## Tanzu Supply Chain Doc Automation Description

Why: There is a history of the old supply chain docs being wrong because sections haven't been updated, and information is spread across different places. The source of truth is in too many places. 
Solution: One source of truth. Doc automation that takes descriptions directly from components, generates md, and raises a doc PR.

What topic in the doc contains the automated doc: [Catalog of Tanzu Supply Chain Components](https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.8/tap/supply-chain-reference-catalog-about.html)
What does this page describe: Provides descriptions for components provided by the authoring profile
Other pages: [Output Types](https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.8/tap/supply-chain-reference-catalog-output-types.html), while the content of this page is manually created, the Catalog of Tanzu
Supply Chain Components page links to this page.

### Automation Description

The component's description can now be pulled verbatim from an installed component with a command like:

```shell
kubectl get component -n sonarqube-catalog sonarqube-sast-scan-1.0.0 -oyaml | yq .status.docs -r
```

## How to describe the APIs

Top level sections usually broken into
* Metadata
* Spec
* Status

Subsections, especially in Spec and Status are alphabetical to make finding them easier, even if the topics _feel_ out of sequence.
Tend to focus on each sections meaning, where children fields can be further subheadings or generalized in examples. 