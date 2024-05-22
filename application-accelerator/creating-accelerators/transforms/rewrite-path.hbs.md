# RewritePath transform

This topic tells you about the Application Accelerator `RewritePath` transform in Tanzu Application Platform (commonly known as TAP).

The `RewritePath` transform allows you to change the name and path of files without affecting their content.

## <a id="syntax-ref"></a>Syntax reference

```plaintext
RewritePath(regex: REGEX, rewriteTo: SPEL-EXPRESSION, matchOrFail:BOOLEAN)
```

Where:

- `REGEX` is a regular expression (regex). For each input file, `RewritePath` attempts to match its
  `path` by using this regex.
  If the regex matches, `RewritePath` changes the `path` of the file to the evaluation result of `rewriteTo`.

- `SPEL-EXPRESSION` is a SpEL expression that has access to the overall engine model and to variables
  defined by capturing groups of the regular expression. Both named capturing groups `(?<example>[a-z]*)`
  and regular index-based capturing groups are supported.
  `g0` contains the whole match, `g1` contains the first capturing group, and so on.

- `BOOLEAN` is the behavior you want if the regex doesn't match:
  - If set to `false`, which is the default, the file is left untouched.
  - If set to `true`, an error occurs. This prevents misconfiguration if you
    expect all files coming in to match the regex. For more information about
    typical interactions between `RewritePath` and `Chain + Include`,
    see the following section, [Interaction with Chain and Include](#interaction-chain-include).

### <a id="defaults"></a> Default values

The default values for `RewritePath` are as follows:

- **`regex`:** The default is the following regular expression, which provides convenient access
  to some named capturing groups:

    ```regex
    ^(?<folder>.*/)?(?<filename>([^/]+?|)(?=(?<ext>\.[^/.]*)?)$)
    ```

    Using `some/deep/nested/file.xml` as an example, the default regular expression captures:

    - **folder:** The full folder path the file is in. In this example, `some/deep/nested/`.
    - **filename:** The full name of the file, including extension _if present_. In this example, `file.xml`.
    - **ext:** The last dot and extension in the filename, _if present_. In this example, `.xml`.

- **`rewriteTo`:** The default is the expression `#folder + #filename`, which doesn't rewrite paths.

- **`matchOrFail`:** The default is `false`, which leaves the file untouched if the regex doesn't match.

## <a id="examples"></a>Examples

See the following examples using the `RewritePath` transform.

### <a id="example1"></a>Example 1

The following moves all files from `src/main/java` to `sub-module/src/main/java`:

```plaintext
RewritePath(regex: "src/main/java/(.*)", rewriteTo: "sub-module/src/main/java" + #g1)
```

![Diagram showing a RewritePath transform.](images/rewrite-path.svg)

### <a id="example2"></a>Example 2

The following flattens all files found inside the `sub-path` directory and its subdirectories,
and puts them into the `flattened` folder:

```plaintext
RewritePath(regex: "sub-path/(.*/)*(?<filename>[^/]+)", rewriteTo: "flattened" + #filename)
```

### <a id="example3"></a>Example 3

The following turns all paths into lowercase:

```plaintext
RewritePath(rewriteTo: #g0.toLowerCase())
```

## <a id='interaction-chain-include'></a>Interaction with Chain and Include

It's common to define pipelines that perform a `Chain` of transformations
on a subset of files, typically selected by `Include/Exclude`:

```plaintext
Include({"**/*.java"})
T1()
T2()
T3()
```

If one of the transformations in the chain is a `RewritePath` operation,
chances are you want the rewrite to apply to _all_ files matched by the `Include`.
For those typical configurations, you can set the `matchOrFail` guard to `true` to
ensure the `regex` you provide indeed matches all files coming in.

## <a id="see-also"></a> See also

- Use [UniquePath](unique-path.md) to ensure rewritten paths don't clash with
  other files, or to decide which path to select if they do clash.
