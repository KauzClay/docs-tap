# Exclude transform

This topic tells you about the Application Accelerator `Exclude` transform in Tanzu Application Platform (commonly known as TAP).

The `Exclude` transform retains files based on their `path`, allowing all files except ones with a path that matches at least one of the configured `patterns`. The contents of files, and any of their other characteristics are unaffected.

`Exclude` is a basic building block seldom used _as is_, which makes sense
if composed inside a [Chain](chain.md) or a [Merge](merge.md).

## <a id="syntax-reference"></a>Syntax reference

```plaintext
Exclude(LIST-OF-STRINGS)
```

Where `LIST-OF-STRINGS` is a list of `patterns` matching the files that you want to exclude.

## <a id="examples"></a>Examples

```plaintext
Exclude({"**/secret/**"})
```

![Diagram showing an exclude transform.](images/exclude.svg)

## <a id="see-also"></a> See also

- [Include](include.md)
