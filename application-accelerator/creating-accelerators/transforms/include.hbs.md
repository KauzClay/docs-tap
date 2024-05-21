# Include transform

This topic tells you about the Application Accelerator `Include` transform in Tanzu Application Platform (commonly known as TAP).

The `Include` transform retains files based on their `path`, letting in _only_ those files
whose path matches at least one of the configured `patterns`.
The contents of files, and any of their other characteristics, are unaffected.

`Include` is a basic building block seldom used as is, which
makes sense if composed inside a [Chain](chain.md) or a [Merge](merge.md).

## <a id="syntax-ref"></a>Syntax reference

```go
Include(LIST-OF-STRINGS)
```

Where `LIST-OF-STRINGS` is a list of `patterns` matching the files that you want to include.

## <a id="examples"></a>Examples

```go
 Include({"**/*.yaml"})
```

![Diagram showing an include transform.](images/include.svg)

## <a id="see-also"></a> See also

- [Exclude](exclude.md)
