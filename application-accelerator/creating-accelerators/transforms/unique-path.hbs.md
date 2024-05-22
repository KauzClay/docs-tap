# UniquePath transform

This topic tells you about the Application Accelerator `UniquePath` transform in Tanzu Application Platform (commonly known as TAP).

You can use the `UniquePath` transform to ensure there are no `path` conflicts between files transformed.
You can often use this at the tail of a [Chain](chain.md).

## <a id="syntax-ref"></a>Syntax reference

```plaintext
UniquePath(CONFLICT-RESOLUTION)
```

Where `CONFLICT-RESOLUTION` is the resolution strategy you want to use from the list in
[Available strategies](conflict-resolution.hbs.md#available-strategies).

## <a id="examples"></a>Examples

The following example concatenates the file that was originally named `DEPLOYMENT.md`
to the file `README.md`:

```plaintext
Include({"README.md"})
+ Include({"DEPLOYMENT.md"})
  .RewritePath({rewriteTo: "README.md"})

UniquePath(Append)
```

## <a id="see-also"></a> See also

- `UniquePath` uses a [Conflict Resolution](conflict-resolution.md) strategy to decide
  what to do when several input files use the same `path`.
