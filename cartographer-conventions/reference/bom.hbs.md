# BOM for Cartographer Conventions

This reference topic describes the `BOM` structure you can use with Cartographer Conventions.

## Overview

The `BOM` is a structure wrapping a Software Bill of Materials (SBOM) that describes the software
components and their dependencies.

## Structure

The structure of the `BOM` is defined as follows:

```toml
{
  "name": "BOM-NAME",
  "raw": "BYTE-ARRAY"
}
```

Where:

- `BOM-NAME` is the prefix `cnb-sbom:` followed by the location of the `BOM` definition in the layer
  for a Cloud Native Buildpack (CNB) SBOM. For example:
  `cnb-sbom:/layers/sbom/launch/paketo-buildpacks_executable-jar/sbom.cdx.json`. For a non-CNB SBOM,
  the value of `name` might be different.

- `BYTE-ARRAY` is the content of the BOM. The content can be in any format or encoding. Read the
  name to learn how the content is structured.

The convention controller forwards BOMs to the convention servers that it can detect from known
sources, including [CNB-SBOM](https://github.com/buildpacks/rfcs/blob/main/text/0095-sbom.md).
