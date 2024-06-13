# Loop transform

This topic tells you about the Application Accelerator `Loop` transform in Tanzu Application Platform
(commonly known as TAP).

The `Loop` transform iterates over elements in a list and applies the provided transform for every
element in that list.

Depending on the style you use, denoted by the `chain` or `merge` keyword, the transform behaves differently:

- When you use `foreach .. in .. merge {..}`, a copy of the input for the `Loop` transform is passed
  to each transform and the outputs from each transform are merged using a set union.

- When you use `foreach .. in .. chain {..}`, each transform is executed sequentially, receiving the
  output from the previous transform as its input. The first transform is to receive the input from
  the `Loop` transform as its input.

## <a id="syntax-reference"></a>Syntax reference

```plaintext
foreach VAR-IDENTIFIER [, INDEX-IDENTIFIER] in SPEL-EXPRESSION (chain|merge) {
  ... // other transforms
}
```

- `SPEL-EXPRESSION` is a SpEL expression that evaluates a list. This is the list of elements to be
  iterated over.
- `VAR-IDENTIFIER` is the name of the variable to be assigned to the current element on each iteration.
- (Optional) `INDEX-IDENTIFIER` is the name of the variable to be assigned to the index of the current
  element on each iteration.

## <a id="behavior"></a>Behavior

Consider the following when choosing the `merge` or `chain` mode:

- `foreach .. merge` executes the transform on the same input files for every iteration and merges the
  resulting outputs. It is best suited when a transform is executed multiple times on the same input
  and does not have conflicts.

- `foreach .. chain` executes the transform on the initial input files once and then passes the resulting
  output to the second iteration and so on. It is best suited when a transform must detect any changes
  that occurred in the previous iteration.

## <a id="examples"></a>Examples

See the following examples using the `Loop` transform.

### <a id="example1"></a>Example 1

Create a new directory for every module in `modules` (a list of strings) based on the contents of
the "template" directory.

```plaintext
foreach m in #modules merge {
  RewritePath(regex: "template/(.*)", rewriteTo: #m + '/' + #g1)
}
```

The following diagram shows how this example behaves:

![Diagram showing a loop transform.](images/loop1.svg)

### <a id="example2"></a>Example 2

Add every artifactId in `artifacts` (a list of strings) as a Spring dependency.

```plaintext
foreach a in #artifacts chain {
  OpenRewriteRecipe(
    recipe: 'org.openrewrite.maven.AddDependency',
    options: {
      groupId: 'org.springframework',
      artifactIf: #a,
      version: '5.7.1
    }
  )
}
```

The following diagram shows how this example behaves:

![Diagram showing a loop transform.](images/loop2.svg)

### <a id="example3"></a>Example 3

You can use `Loop` in combination with custom types, for example:

`accelerator.yaml`:
```yaml
accelerator:
  types:
    - name: MavenPlugin
      struct:
        - name: groupId
        - name: artifactId
        - name: version
  options:
    - name: pluginsToAdd
      dataType: [MavenPlugin] # End users will be able to enter a collection of GAV tuples
```

`accelerator.axl`:
```plaintext
engine {
  Include({"pom.xml"})
  foreach p in #pluginsToAdd chain {
    OpenRewriteRecipe(
      recipe: 'org.openrewrite.maven.AddPlugin',
      options: {
        groupId:    #p['groupId'],
        artifactId: #p['artifactId'],
        version:    #p['version']
      }
    )
  }
}
```

For more information, see [Using Custom Types](../custom-types.hbs.md).
