# Use transforms in Application Accelerator

This topic tells you about using transforms with Application Accelerator.

When the accelerator engine executes the accelerator, it produces
a set of files. The purpose of the `accelerator.axl` file is to describe
precisely how that set of files is created.


`accelerator.yaml`:
  ```yaml
  accelerator:
    # Describes options, custom types and imports
    ...
  ```

`accelerator.axl`:
  ```
  engine {
    // Describes what happens to the files of the accelerator
  }
  ```

## <a id="why-transforms"></a>Why transforms?

When you run an accelerator, the contents of the accelerator produce the result.
It is made up of subsets of the files taken from the accelerator `<root>` directory and
its subdirectories. You can copy the files as is, or transform them in a number of ways before
adding them to the result.

The DSL notation in the `engine` section defines a transformation that takes as
input a set of files (in the `<root>` directory of the accelerator) and produces
as output another set of files, which ultimately make up the result.

Every transform has a `type`. Different types of transform have different behaviors and
different properties that control precisely what they do.

In addition to this, the accelerator DSL uses some "keyword" style syntax to make
some particular transforms first class citizens (such as `if()` or merge with `T1 + T2`)
but at its core, the behavior of the engine is _functional_: something comes in, a transformation
is applied, something comes out.

In the following example, a transform of type `Include` is a filter.
It takes as input a set of files and produces as output a subset of those files, retaining only
those files whose path matches any one of a list of `patterns`.

If the accelerator has something like this:

  ```
  engine {
    Include(patterns: {'**/*.java'})
  }
  ```

This accelerator produces a result containing all the `.java` files from the accelerator
`<root>` or its subdirectories but nothing else.

Transforms can also operate on the contents of a file, instead of merely selecting it for inclusion.

For example:
  ```
  engine {
    ReplaceText(substitutions: \{{text: "hello-fun", with: #artifactId }})
  }
  ```

This transform looks for all instances of a string `hello-fun` in all its
input files and replaces them with an `artifactId`, which is the result of
evaluating a SpEL expression.

The general syntax for using a transform of type `MyTransform` is two
write `MyTransform()`. This is referred as using the transform _constructor_.

If the transform can be parameterized with configuration properties, those (typed)
properties can be passed in the constructor call, for example:

```
  Include(patterns: {'**/*.java'})
```

Properties can be passed by name like above, or in order like in the example below.
To learn more about the existing transforms and their configuration properties,
refer to the [Transforms reference](transforms/index.hbs.md).

```
  Include( {'**/*.java'} )
```


## <a id="combine-transforms"></a>Combining transforms

From the preceding examples, you can see that transforms such as `ReplaceText`
and `Include` are too primitive to be useful by themselves. They are meant to be the building blocks of more complex accelerators.

To combine transforms, Application Accelerators rely on two operators called `Chain` and `Merge`.
These operators are recursive in the sense that they compose a number of child transforms to create
a more complex transform. This allows building arbitrarily deep and complex trees of
nested transform definitions.

The following example shows what each of these two operators does and how they are used together.

### <a id="chain"></a>Chain

Because transforms are functions whose input and output are of the same type (a set of files),
you can take the output of one function and feed it as input to another. This is what `Chain` does.
In mathematical terms, `Chain` is function composition.

![Diagram of a chain transform.](transforms/images/chain.svg)

You might, for example, want to do this with the `ReplaceText` transform.
Used by itself, it replaces text strings in all the accelerator input files. What if
you wanted to apply this replacement to only a subset of the files? You can use an `Include`
filter to select only a subset of files of interest and chain that subset into `ReplaceText`.

The way to use chains in the DSL syntax is straightforward. Just write transform `A` followed
by transform `B`:

```
engine {
  Include(patterns: {'**/*.java'})
  ReplaceText(substitutions: \{{text: "hello-fun", with: #artifactId }})
}
```

### <a id="merge"></a>Merge

Chaining `Include` into `ReplaceText` limits the scope of `ReplaceText` to a subset of the input files.
It also eliminates all other files from the result.

For example:

  ```
  engine {
    Include(patterns: {'**/pom.xml'})
    ReplaceText(substitutions: \{{text: "hello-fun", with: #artifactId }})
  }
  ```

The preceding accelerator produces a project that only contains `pom.xml` files and nothing else.

What if you also wanted other files in that result? Perhaps you want to include some Java files as well,
but don't want to apply the same text replacement to them.

You might be tempted to write something such as:

  ```
  engine {
    Include(patterns: {'**/pom.xml'})
    ReplaceText(substitutions: \{{text: "hello-fun", with: #artifactId }})
    Include(patterns: {'**/*.java'})
  }
  ```


However, that doesn't work. If you chain non-overlapping includes together like this,
the result is an empty result set. The reason is that the first include retains only `pom.xml` files. These files
are fed to the next transform in the chain. The second include only retains `.java` files, but because there are only `pom.xml` files left in the input, the result is an empty set.

This is where `Merge` comes in. A `Merge` takes the outputs of several transforms executed independently
on the same input sourceset and combines or merges them together into a single sourceset.
The way to write merges in the DSL syntax is to use the `+` operator, symbolizing the union
of the result sets:



For example:

  ```
  engine {
    {
      Include(patterns: {'**/pom.xml'})
      ReplaceText(substitutions: \{{text: "hello-fun", with: #artifactId }})
    }
    + Include(patterns: {'**/*.java'})
  }
  ```

The preceding accelerator produces a result that includes both:

- The `pom.xml` files with some text replacements applied to them.
- Verbatim copies of all the `.java` files.


## <a id="apply-to"></a>Thinking "Chain First" with the applyTo operator

Combining effects on some files (say, all `pom.xml` files) and then doing something
else with other files (say, all `*.java` files) can be cumbersome if the only tools
at hand are chains and merges. You'd need to create an N+1 merge construct, where
each of the N transformations apply specifically to the files of interest, and the
last one brings _the rest of files_ (that were of no interest to any of the N first inclusions).

To avoid having to worry about this, the `applyTo()` operator can be used
sequentially:

```
engine {
  applyTo('**/pom.xml') {
    ReplaceText(substitutions: \{{text: "hello-fun", with: #artifactId }})
  }
  applyTo('**/*.java') {
    OpenRewriteRecipe('org.openrewrite.java.ChangePackage',
      { oldPackageName: 'com.acme',
      newPackageName: #companyPkg }
    )
  }
}
```

What `applyTo` does behind the scenes is that it constructs the complex `Merge, Include, Exclude` equivalent setup for you: the inner transform (`ReplaceText` in the first example) will only be applied
to the files selected (`**/pom.xml` in the example), _while the other files (everything that is not a `pom.xml` in the example) will still carry through, untouched_ to the next transform down the line.


## <a id="conditional-transforms"></a>Conditional transforms

Transforms (or sequences of transforms) can be wrapped inside an `if()` construct, like so:


  ```
  engine {
    if (#k8sConfig == 'k8s-resource-simple') {
      Include({"kubernetes/app/*.yaml"})
      ReplaceText(\{{text: 'hello-fun', with: #artifactId}})
    }
  }
  ```

When an `if` condition is `false`, that transform is deactivated.
This means it is replaced by a transform that does nothing.
However, doing nothing can have different meanings depending on the context:

* When in the context of a `Merge`, a deactivated transform behaves like something that
returns an empty set. A `Merge` adds things together using a kind
of union; adding an empty set to union essentially does nothing.

* When in the context of a `'Chain` however, a deactivated transform behaves like
the `identity` function instead (that is, `lambda (x) => x`). When you chain
functions together, a value is passed through all functions in succession. So each
function in the chain has the chance to do something by returning a different modified value.
If you are a function in a chain, to do nothing means to return
the input you received unchanged as your output.


## <a id="merge-conflict"></a>Merge conflict

The representation of the set of files upon which transforms operate is richer than what you can
physically store on a file system. A key difference is that in this case, the set of files allows for multiple files
with the same path to exist at the same time. When files are initially read from the accelerator
repository this situation does not arise. However, as transforms are
applied to this input, it can produce results that have more than one file with
the same path and yet different contents.


For example when using a merge:
  ```
  engine {
    Include({"**/*})          // Transform A
    + {                       // Transform B
      Include({**/pom.xml})
      ReplaceText(...)
    }
  }
  ```

The result of the preceding `merge` is two files with path `pom.xml`, assuming there was
a `pom.xml` file in the input. **Transform A** produces a `pom.xml` that is a verbatim copy of the
input file. **Transform B** produces a modified copy with some text replaced in it.

It is impossible to have two files on a disk with the same path. Therefore, this conflict
must be resolved before you can write the result to disk or pack it into a ZIP file.

As the example shows, merges are likely to give rise to these conflicts, so you
might call this a "merge conflict." However,
such conflicts can also arise from other operations. For example, `RewritePath`:

  ```
  RewritePath(regex: '.*\.md', rewriteTo: 'docs/README.md')
  ```

This example renames any `.md` file to `docs/README.md`. Assuming the input contains
more than one `.md` file, the output now contains multiple files with path `docs/README.md`.
Again, this is a conflict, because there can only be one such file in a physical file system or
ZIP file.

### <a id="resolve-merge-conflicts"></a>Resolving merge conflicts

By default, when a conflict arises, the engine doesn't do anything with it. Our internal representation
for a set of files allows for multiple files with the same path. The engine carries on
manipulating the files as is. This isn't a problem until the files must be written
to disk or a ZIP file. If a conflict is still present at that time, an error is raised.

If your accelerator produces such conflicts, they must be resolved
before writing files to disk. VMware provides the [UniquePath](transforms/unique-path.md)
transform. This transform allows you to specify what to do when more than one file has the same
path. For example:

  ```
  engine {
    RewritePath(regex: '.*\.md', rewriteTo: 'docs/README.md')
    UniquePath(strategy: Append)
  }
  ```


The result of the above transform is that all `.md` files are gathered up and concatenated into a
single file at path `docs/README.md`. Another possible resolution strategy is to keep
only the contents of one of the files. See [Conflict Resolution](transforms/conflict-resolution.md).

### <a id="file-ordering"></a>File ordering

As mentioned earlier, our set of files representation is richer than the files on a typical file
system in that it allows for multiple files with the same path. Another way in which it is richer is that
the files in the set are ordered. That is, a `FileSet` is more like an ordered list than an unordered set.

In most situations, the order of files in a `FileSet` doesn't matter. However, in conflict resolution
it is significant. If you look at the preceding `RewritePath` example again,
you might wonder about the order in which the various
`.md` files are appended to each other. This ordering is determined by the order of
the files in the input set.

So what is that order? In general, when files are read from disk to create a
`FileSet`, you cannot assume a specific order. Yes, the files are read and processed in a sequential order,
but the actual order is not well defined. It depends on implementation details of the underlying
file system. The accelerator engine therefore does not ensure a specific order in this case.
It only ensures that it preserves whatever ordering it receives from the file system, and processes files in
accord with that order.

If you do not want the file order produced from reading directly from a file system and want to
control the order of the sections in the `README.md` file, change the order of the merge children.
`Merge` processes its children in order and reflects this order in the resulting output

For example:

  ```
  engine {
    Include({'README.md'})
    + {
      Include({'DEPLOYMENT.md'})
      RewritePath(rewriteTo: 'README.md')
    }

    UniquePath(Append)
  }
  ```

In this example, `README.md` from the first child of `merge` comes
before `DEPLOYMENT.md` from the second child of `merge`.

## <a id="conclusion"></a>Next steps

This introduction focused on an intuitive
understanding of the `<transform-definition>` notation. This notation defines
precisely how the accelerator engine generates new project content from the files
in the accelerator root.

For more information, see:

- An exhaustive [Reference](transforms/index.hbs.md) of all built-in transform types
- A sample, commented [accelerator](accelerator-yaml-sample.hbs.md) to learn from a concrete example
