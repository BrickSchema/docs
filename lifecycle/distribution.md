---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---


Brick Distributions
===================

```{admonition} Note
All distributions are available on the [Brick GitHub Releases page](https://github.com/BrickSchema/Brick/releases).
```

The following distributions are available for the Brick schema on the [Releases](https://github.com/BrickSchema/Brick/releases) page:
- `Brick+imports.ttl` (recommended for end-applications): contains the Brick and RealEstateCore ontologies, as well as all imports; no other dependencies are required
- `Brick.ttl` (recommended for platforms): contains the Brick and RealEstateCore ontologies; all other dependencies must be imported or otherwise included
- `Brick-only.ttl`: contains *only* the Brick ontology; all other dependencies, including RealEstateCore, must be imported or otherwise included
- `Brick+extensions.ttl`: contains the Brick and RealEstateCore ontologies, as well as all extensions currently in the Brick repository. All other dependencies must be imported or otherwise included.

With supplementary files:
- `imports.zip`: contains all imports for the Brick and RealEstateCore ontologies as individual Turtle (`.ttl`) files
- `extensions.zip`: contains all extensions currently in the Brick repository as individual Turtle (`.ttl`) files

The `Brick+imports.ttl` file is convenient for end-applications, as it contains all necessary imports.
This means that applications can just import this file and have access to the entire Brick schema.
This simplifies validation and inference on Brick models because the end-application does not have to resolve or find dependencies.

The `Brick.ttl` file is recommended for software platforms, as it contains only the Brick and RealEstateCore ontologies, allowing for more control over dependencies. 
By keeping the imports separate, software platforms can deduplicate imports and update or manage these dependencies independently of the core Brick schema.

The `Brick-only.ttl` file is useful for applications that only need the Brick ontology and do not require RealEstateCore or other extensions. This is unlikely to be useful for most applications, as RealEstateCore is deeply integrated with Brick and is required for many common use cases.

The `Brick+extensions.ttl` file is useful for applications that require all extensions in the Brick repository. This is unlikely to be useful for most applications, as extensions are typically used on a case-by-case basis.
