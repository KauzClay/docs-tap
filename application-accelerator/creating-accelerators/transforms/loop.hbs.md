# Loop transform

This topic tells you about the Application Accelerator `Loop` transform in Tanzu Application Platform (commonly known as TAP).

The `Loop` transform iterates over elements in a list and applies the provided transform for every
element in that list.

Depending on the "style" used (denoted by the optional `+` marker), the transform either behaves in "doArMerge" (`+`) or "doAsChain" (no `+`):

- When `doAsMerge` is used, a copy of the `Loop` transform's input is passed to each transform and the
outputs from each transform are merged using a set union.

- When `doAsChain` is used, each transform is executed sequentially, receiving the previous
transform's output as its input. The first transform is to receive the `Loop` transform's input as
its input.

## <a id="syntax-reference"></a>Syntax reference

```plaintext
for VAR-IDENTIFIER [, INDEX-IDENTIFIER] in SPEL-EXPRESSION [+] {
  ... // other transforms
}
```

- `SPEL-EXPRESSION` must be a SpEL expression that evaluates a list. This is the list of elements to be
  iterated over.
- `VAR-IDENTIFIER` is the name of the variable to be assigned to the current element on each iteration.
  (required)
- `INDEX-IDENTIFIER` is the variable's name to be assigned to the index of the current element on
  each iteration. (optional)


## <a id="behavior"></a>Behavior

Consider the following when choosing `doAsMerge` or `doAsChain`:

`doAsMerge` executes the transform on the same input files for every iteration and merges the
resulting outputs. It is best suited when a transform is executed multiple times on the
same input and does not have conflicts.

`doAsChain` executes the transform on the initial input files once and then passes the resulting
output to the second iteration and so on. It is best suited when a transform must detect any changes
that occurred in the previous iteration.

## <a id="examples"></a>Examples

See the following examples using the `Loop` transform.

### <a id="example1"></a>Example 1

Create a new directory for every module in `modules` (a list of strings) based on the contents of
the "template" directory.

```plaintext
for m in #modules +{
  RewritePath(regex: "template/(.*)", rewriteTo: #m + '/' + #g1)
}
```

The following diagram shows how this example behaves:

![Diagram showing a loop transform.](images/loop1.svg)

### <a id="example2"></a>Example 2

Add every artifactId in `artifacts` (a list of strings) as a Spring dependency.

```plaintext
for a in #artifacts {
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
  for p in #pluginsToAdd {
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
