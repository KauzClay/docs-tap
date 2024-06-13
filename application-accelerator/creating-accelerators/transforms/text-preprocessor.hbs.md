# TextPreprocessor transform

This topic tells you about the Application Accelerator `TextPreprocessor` transform in
Tanzu Application Platform (commonly known as TAP).

The `TextPreprocessor` transform allows conditional inclusion or exclusion of lines of text
in documents, based on SpEL expression conditions. It is inspired by the syntax of C-style
preprocessor directives (`#IF/#ENDIF`).

## <a id="syntax-reference"></a>Syntax reference

```plaintext
TextPreprocessor()
```

The `TextPreprocessor` doesn't accept configuration parameter.
It acts on (text) files that are input to it, typically through `Include()`, and uses
the set of values visible at invocation time to evaluate SpEL expressions.

## <a id="behavior"></a>Behavior

The `TextPreprocessor` looks for lines that contain the following constructs:

- `#IF(condition)`: evaluates the condition, using SpEL syntax, and discards all lines until a
  matching closing directive if it evaluates to `false`.

- `#ELSE`: if the previous conditions evaluated to `false`, the contents of this branch is retained.

- `#ELIF(condition)`: if the previous conditions evaluated to `false`, it considers a new condition.

- `#ENDIF`: closes an `#IF/#ELSE/#ELIF` block. Every block requires a closing `#ENDIF` directive.

These constructs don't have to be at start of the line. In particular, you can embed directives
inside comments in the target language so that they don't break parsing of files in raw form.
All lines that contain directives are cut out from the resulting file.

The directives are case-sensitive, they must be in capitals. The `#IF(` characters must always
appear as a block and have no space in between.

You can also nest directives. For example, the following two snippets are equivalent:

```plaintext
#IF(a && b)
something
#ENDIF
#IF(a && c)
something else
#ENDIF
```

```plaintext
#IF(a)
#IF(b)
something
#ENDIF
#IF(c)
something else
#ENDIF
#ENDIF
```

## <a id="examples"></a>Example

This example includes `pom.xml` dependencies based on the value of some accelerator option,
in this case, `databaseType`.

Example `pom.xml`:

```xml
  <!-- #IF(#databaseType == 'postgres') -->
  <dependency>
      <groupId>org.postgresql</groupId>
      <artifactId>postgresql</artifactId>
      <scope>runtime</scope>
  </dependency>
  <!-- #ENDIF -->
  <!-- #IF(#databaseType == 'mysql') -->
  <dependency>
      <groupId>com.mysql</groupId>
      <artifactId>mysql-connector-j</artifactId>
      <scope>runtime</scope>
  </dependency>
  <!-- #ENDIF -->
```

Example `accelerator.axl`:

```plaintext
engine {
  ...
  applyTo('pom.xml') {
    TextPreprocessor()
  }
  ...
}
```
