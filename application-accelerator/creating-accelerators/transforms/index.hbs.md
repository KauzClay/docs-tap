# Transforms reference

This topic provides you with a list and brief description of the available Application Accelerator
transforms in Tanzu Application Platform (commonly known as TAP).

## Available transforms

You can use:

- [Include](include.hbs.md) to select files to operate on.
- [Exclude](exclude.hbs.md) to select files to operate on.
- [Merge](merge.hbs.md) to work on subsets of inputs and to gather the results at the end.
- [Chain](chain.hbs.md) to apply several transforms in sequence using function composition.
- [Let](let.hbs.md) to introduce new scoped variables to the model.
- [InvokeFragment](invoke-fragment.hbs.md) allows re-using various fragments across accelerators.
- [ReplaceText](replace-text.hbs.md) to perform simple token replacement in text files.
- [TextPreprocessor](text-preprocessor.hbs.md) to discard parts of text files based on conditions.
- [RewritePath](rewrite-path.hbs.md) to move files around using regular expression (regex) rules.
- [OpenRewriteRecipe](open-rewrite-recipe.hbs.md) to apply [Rewrite](https://docs.openrewrite.org/)
  recipes, such as package rename.
- [YTT](ytt.hbs.md) to run the `ytt` tool on its input files and gather the result.
- [UseEncoding](use-encoding.hbs.md) to set the encoding to use when handling files as text.
- [UniquePath](unique-path.hbs.md) to decide what to do when several files end up on the same path.
- [Loop](loop.hbs.md) to iterate over a list and apply a transform for each element.
- [Provenance](provenance.hbs.md) to generate a manifest of the accelerator run.

## See also

- [Conflict Resolution](conflict-resolution.hbs.md)
