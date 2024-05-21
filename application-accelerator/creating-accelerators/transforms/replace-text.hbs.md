# ReplaceText transform

This topic tells you about the Application Accelerator `ReplaceText` transform in Tanzu Application Platform (commonly known as TAP).

The `ReplaceText` transform allows replacing one or several text tokens in files as
they are being copied to their destination. The replacement values are the result
of dynamic evaluation of [SpEL expressions](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#expressions).

This transform is text-oriented and requires knowledge of how to interpret the stream of bytes that make up the file contents into text.
All files are assumed to use `UTF-8` encoding by default, but you can use the [UseEncoding](use-encoding.md) transform upfront to specify a different charset to use on some files.

You can use `ReplaceText` transform in one of two ways:

- To replace several literal text tokens.
- To define the replacement behavior using a single regular expression, in which case the replacement SpEL expression can leverage the regex capturing group syntax.

## <a id="syntax-ref"></a>Syntax reference

- Syntax reference for replacing several literal text tokens:

    ```go
    ReplaceText({text: "SOME-TEXT",with: "REPLACEMENT"})
    ```

- Syntax reference for defining the replacement behavior using a _single_ regular expression:

    ```go
    ReplaceText(regex: {pattern: "REGULAR-EXPRESSION",with: SPEL-EXPRESSION})
    ```

    Pattern is used to match the entire document. To match on a per line basis, enable multiline mode by
    including `(?m)` in the regex.

In both cases, the SpEL expression can use the special `#files` helper object.
This enables the replacement string to consist of the contents of an accelerator file.
See the [Examples](#examples).

Another set of helper objects are functions of the form `xxx2Yyyy()` where `xxx` and `yyy` can take
the value `camel`, `kebab`, `pascal`, or `snake`.
For example, `camel2Snake()` enables changing from camelCase to snake_case.

## <a id="examples"></a>Examples

See the following examples using The `ReplaceText` transform.

### <a id="example1"></a>Example 1

Replacing the hardcoded string `"hello-world-app"` with the value of variable `#artifactId`
in all `.md`, `.xml`, and `.yaml` files.

```go
Include({"**/*.md", "**/*.xml", "**/*.yaml"})
ReplaceText({text: "hello-world-app", with: #artifactId})
```

![Diagram showing a ReplaceText transform.](images/replace-text1.svg)

### <a id="example2"></a>Example 2

Replacing the hardcoded string `"hello-world-app"` with the value of variable `#artifactId` in the
`README-fr.md` and `README-de.md` files, which are encoded using the `ISO-8859-1` charset:

```go
Include({"README-fr.md", "README-de.md"})
UseEncoding("ISO-8859-1")
ReplaceText({text: "hello-world-app", with: #artifactId})
```

### <a id="example3"></a>Example 3

Similar to the preceding example, but making sure the value appears as kebab case,
while the entered `#artifactId` is using camel case:

```go
Include({"**/*.md", "**/*.xml", "**/*.yaml"})
ReplaceText({text: "hello-world-app", with: #camel2Kebab(#artifactId)})
```

### <a id="example4"></a>Example 4

Replacing the hardcoded string `"REPLACE-ME"` with the contents of
file named after the value of the `#platform` option in `README.md`:

```go
Include:({"README.md"})
ReplaceText({text: "REPLACE-ME", with: #files.contentsOf("snippets/install-" + #platform + ".md")})
```

### <a id="example5"></a>Example 5

Replacing all occurrences of `apple` or `orange`, singular or plural,
with the value of the accelerator option `#vegetable`, for example `'banana'`,
keeping the trailing `'s'` when there was one.

```go
Include({"README.md"})
// This constructs a SpEL string containing eg 'banana$2' where $2
// refers to the second capturing group (the optional 's')
ReplaceText(regex: {pattern: "(apple|orange)(s)?", with: #vegetable + "$2"})
```

## <a id="see-also"></a> See also

- [UseEncoding](use-encoding.md)
