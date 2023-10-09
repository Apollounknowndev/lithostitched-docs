---
layout: page
title: Dimension Data
permalink: /dimension-data/
nav_order: 4
---

# Dimension Data

## Introduction

Dimension data controls certain information about a dimension. Currently, this is only used for slice data.

## Slice Data

Slice data controls configuration sizes of slices in the modded biome slice system on a per-dimension basis. If a size is not explicitly defined, the size from the slice data of the `lithostitched:default` dimension is used. *(Note that the `lithostitched:default` dimension doesn't actually exist, as slice data can reference nonexistant dimensions.)*

To define the slice size for a dimension, a file named `slice_data.json` should be put in `data/(dimension_id)/lithostitched/dimension_data/(dimension_path)/slice_data.json` For example, overriding the slice data for the Overworld would require the `slice_data.json` file to be in `data/minecraft/lithostitched/dimension_data/overworld/slice_data.json`

The `slice_data` format looks like this:

```json
{
  "size": 9
}
```

Currently, it has one field: a `size` integer, determining the size of slices.