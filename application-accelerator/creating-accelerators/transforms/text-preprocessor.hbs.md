# TextPreprocessor transform

This topic tells you about the Application Accelerator `TextPreprocessor` transform in Tanzu Application Platform (commonly known as TAP).

The `TextPreprocessor` transform allows conditional inclusion/exclusion of lines of text
in documents, based on SpEL expression conditions. It is inspired by the syntax of C-style
preprocessor directives (`#IF/#ENDIF`).

## <a id="syntax-reference"></a>Syntax reference

```plaintext
TextPreprocessor()
```

The `TextPreprocessor` doesn't accept configuration parameter.
It simply acts on (text) files that are fed to it (typically via `Include()`) and uses 
the set of values visible at invocation time to evaluate SpEL expressions.

## <a id="behavior"></a>Behavior

The `TextPreprocessor` looks for lines that contain the following constructs:
 * `#IF(condition)`: evaluates the _condition_ (using SpEL syntax) and discards all lines until a matching closing directive if it evaluates to `false`
 * `#ELSE`: if the (previous) condition(s) evaluated to `false`, the contents of this branch is retained
 * `#ELIF(condition)`: if the (previous) condition(s) evaluated to `false`, considers a new condition
 * `#ENDIF`: closes an `#IF/#ELSE/#ELIF` block. Every block needs to be correctly balanced with a closing `#ENDIF` directive.
 
Those constructs don't have to be at start of the line. In particular, directives can be embedded inside comments in the target language so they don't break parsing of files in raw form). All lines that contain directives are cut out from the resulting file.

Note however that the directives are case sensitive (must appear in CAPITALS) and that at least the `#IF(` characters always need to appear as a block (no space in between).

Lastly, directives can be nested:

```plaintext
#IF(a && b)
something
#ENDIF
#IF(a && c)
something else
#ENDIF
```

is equivalent to

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

## <a id="examples"></a>Examples

Here is an hypothetical example that includes pom.xml dependencies based on the
value of some accelerator option (`databaseType` in this case):

`pom.xml`:
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

`accelerator.axl`:
```plaintext
engine {
  ...
  applyTo('pom.xml') {
    TextPreprocessor()
  }
  ...
}
```
