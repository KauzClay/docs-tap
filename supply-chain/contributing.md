# Contributing

This is a guide for teams contributing docs to the Supply Chain component section.

## Top section

To boost search engine optimization:

- Beneath the title start with an `In this section...` sentence that, where practical, includes the
  product name, such as:

   ```text
   The section tells you about...
   ```

   or

   ```text
   The topics in this section tell you how to...
   ```

## Naming

The base product is called `Tanzu Supply Chain`. It is especially important not to
shorten this to `Supply Chain` because it is otherwise too easily confused with the `kind` resource
that has a nearly identical name.

Resource kinds should be referred to by their CamelCase name:

- `SupplyChain`
- `Component`
- `WorkloadRun`

## Words and replacements

| Misused | Prefer | Why?                                                                                                                                                          |
|---------|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Steps   | Stage  | The term is `stages` in a supply chain, so use "stages" when referring to supply chains. Obviously, Tekton has steps, and that is a good time to use `steps`. |
|         |        |                                                                                                                                                               |
|         |        |                                                                                                                                                               |

## Headings and verbs

Make each topic title and H1 heading start with a verb, where practical.

For example:

| Original Header        | Improved Header       |
|------------------------|-----------------------|
| Supply Chain Authoring | Author a supply chain |
| How to create Workload | Create a workload     |

## Diagrams

See the [Miro Board](https://miro.com/app/board/uXjVNvc1o0E=/).

## Tanzu Supply Chain Doc Automation Description

**Problem:** There is a history of the old supply chain docs being wrong because sections haven't
been updated, and information is spread across different places. The source of truth is in too many
places.

**Solution:** One source of truth. Doc automation that takes descriptions directly from components,
generates md, and raises a doc PR.

What topic in the doc contains the automated doc:
[Catalog of Tanzu Supply Chain Components](https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.8/tap/supply-chain-reference-catalog-about.html)

What does this page describe: Provides descriptions for components provided by the authoring profile

Other pages: [Output Types](https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.8/tap/supply-chain-reference-catalog-output-types.html),
while the content of this page is manually created, the Catalog of Tanzu Supply Chain Components
page links to this page.

### Platform Engineer User Experience

- The PE installs the authoring profile when they want to author supply chains. This provides components
  that the PE can use when authoring supply chains.
- The PE uses the CLI to see a catalog of installed components. `tanzu supply-chain component list`
- The PE uses the CLI to see detail of the component, including documentation:
  `tanzu supply-chain <component>  get or describe`
- While the PE is authoring supply chains, they can see details about each component. In the CLI
  there are links to the vmware.doc docs page that has the component descriptions.

### Component Description

Each component has the following fields that provide descriptions of the component:

- **Name**

- **Version**

- **Description** could be a multiline string that has Markdown. Some of the descriptions can be
  long and might require some templated headings underneath to accommodate the full scope.

- **Inputs** is a list of objects, usually just 1.

- **Outputs** is a list of objects, usually just 1.

- **Config** is a YAML sample that shows what inputs this component accepts in terms of
  configuration from the workload. The YAML has in-line documentation and this is based on OpenAPI
  Spec V3, which is a Swagger doc format.

### Automation Description

This is custom binary tool, created by Rasheed, reads the component field descriptions and generates
Markdown. The Markdown is then manually added to a Markdown file and a doc merge request is raised.

The aim is to have as little cleanup as possible required for the automatically generated Markdown.
The future plan is for this Markdown to appear at the bottom of the component.

Future plan: When a component is updated, the doc automation tool reads the component, generates the
Markdown, and then automatically generates a doc merge request. But at the moment the steps to copy
the Markdown and raise the merge request are manual.
