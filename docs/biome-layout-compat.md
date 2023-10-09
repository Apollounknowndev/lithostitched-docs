---
layout: page
title: Biome Layout Compat
permalink: /biome-layout-compat/
nav_order: 3
---

# Biome Layout Compatibility

a. [Introduction](#introduction)

b. [Built-in Layout Types](#built-in-layout-types)

c. [Adding New Layout Types](#adding-new-layout-types)

d. [The Original Source Marker](#original-source-marker)

## Introduction

The biome layout system in vanilla Minecraft is not well built for compatibility between itself and biome datapacks and mods. To address this, Lithostitched introduces the *Modded Biome Slice* system, a system where datapacks and mods can add to the biome layout while remaining compatible with one another.

Lithostitched splits the world into slices/areas, and each worldgen project can have full control over the biome layout in these if assigned to it, Worldgen projects can define their own modded biome slice through a datapack file. A slice controls the weight of itself in generation and the biome layout to use if it is chosen. A modded biome slice file looks like this:

```json
{
  "levels": [
    "minecraft:overworld"
  ],
  "layout": {
    "type": "lithostitched:original"
  },
  "weight": 100
}
```

A modded biome slice has three fields:

- `levels`: A list of dimensions that the slice should apply to
- `layout`: A modded biome layout which controls the biome layout of the slice. It will always have a `type` field determining the modded biome layout type to use, with each type adding more fields when applicable.
- `weight`: The weight of the biome slice. The biome weight for the default biome layout is 100.

## Built-in Layout Types

Lithostitched has several built-in modded biome layout types that cover most use cases.

### `original`

The `original` modded biome layout type returns the biome that would've been placed without the modded biome slice system. This is used in Lithostitched's built-in "vanilla" slice.

```json
{
  "type": "lithostitched:original"
}
```

This modded biome layout type has no further fields.

### `multi_noise`

The `multi_noise` modded biome layout type functions like the vanilla multi noise biome source, with some extra fields.

```json
{
  "type": "lithostitched:multi_noise",
  "biomes": [
    {
      "biome": "mod_id:map_from",
      "parameters": {
        "continentalness": 0.0,
        "depth": 0.0,
        "erosion": 0.0,
        "humidity": 0.0,
        "offset": 0.0,
        "temperature": 0.0,
        "weirdness": 0.0
      }
    }
  ],
  "areas": {
    "mod_id:map_from": "mod_id:map_to"
  },
  "only_map_from_areas": true
}
```

- `biomes`: A list of biome entries and their parameters, identical in format to the vanilla multi noise biome source configuration.
- `areas`: A optional map of biome keys to other biome keys, meant to make it easier for users to change the biomes placed in the slice. In the above example, all instances of `mod_id:map_from` are changed to `mod_id:map_to` when actually placed.
- `only_map_from_areas`: If enabled, any biome in the `biomes` list that does not have a key in the `areas` field will be replaced with an [original source marker](#original-source-marker).

### `overlay`

The `overlay` modded biome layout type overlays biomes onto the dimension's original biome layout.

```json
{
  "type": "lithostitched:overlay",
  "overlays": [
    {
      "matches_biomes": "minecraft:end_midlands",
      "biome_source": {
        "type": "minecraft:fixed",
        "biome": "mod_id:custom_end_midlands"
      }
    },
    {
      "matches_biomes": "minecraft:end_highlands",
      "biome_source": {
        "type": "minecraft:fixed",
        "biome": "mod_id:custom_end_highlands"
      }
    }
  ]
}
```

- `overlays`: A list of biome overlays. Biome overlays are an object with two fields:
  - `matches_biomes`: Either a biome, a list of biomes, or a biome tag for the biomes in the dimension's original layout to find.
  - `biome_source`: The biome source to use if the biome in `matches_biomes` is found.

### `biome_source`

The `biome_source` modded biome layout type uses a regular biome source for determining the biomes to place.

```json
{
  "type": "lithostitched:biome_source",
  "biome_source": {
    "type": "minecraft:fixed",
    "biome": "mod_id:my_cool_biome"
  }
}
```

- `biome_source`: The biome source to use.

## Adding New Layout Types

Mods can introduce layout types for their own purposes. To do this, a new class should be added that implements `ModdedBiomeLayout`.

In the most basic form, it looks like this:

```java
public class OriginalModdedBiomeLayout implements ModdedBiomeLayout {

    @Override
    public Holder<Biome> getNoiseBiome(int x, int y, int z, Climate.Sampler sampler, BiomeSource original, Registry<Biome> registry) {
      return original.getNoiseBiome(x, y, z, sampler);
    }

    @Override
    public Set<Holder<Biome>> getAdditionalPossibleBiomes(Registry<Biome> registry) {
      return ImmutableSet.of();
    }

    @Override
    public Codec<? extends ModdedBiomeLayout> codec() {
        return Codec.unit(new OriginalModdedBiomeLayout());
    }
}
```

The `getNoiseBiome` method is used for determining the biome layout when a slice with the layout type is selected. In the example above, it simply returns the biome in the original biome source.

The `getAdditionalPossibleBiomes` method returns all of the extra biomes that can generate in this biome source on top of the ones in the vanilla source. For example, if your biome source could generate either a biome from the original source or a `cold_beach` biome, The `getAdditionalPossibleBiomes` method should return the `cold_beach` biome.

The `codec` method returns the codec that the layout uses to deserealize JSON files that use the modded biome layout type.

## Original Source Marker

Let's say you have a worldgen mod that adds new Desert biome variants through a modded biome slice. What happens if your modded biome slice is selected somewhere that isn't desert? The original source marker allows you to tell the game that you don't want control of the biome layout in certain areas and to instead reroll which slice gets control.

To tell the game to reroll a biome, the biome selected in the modded biome slice needs to be `lithostitched:original_source_marker`. For example, the below modded biome layout configuration will place the Jungle biome on land and relinquish the biome slice control to another slice in the sea:

```json
{
  "type": "lithostitched:multi_noise",
  "biomes": [
    {
      "biome": "lithostitched:original_source_marker",
      "parameters": {
        "continentalness": [
          -1.2,
          -0.19
        ],
        "depth": 0,
        "erosion": 0,
        "humidity": 0,
        "offset": 0,
        "temperature": 0,
        "weirdness": 0
      }
    },
    {
      "biome": "minecraft:jungle",
      "parameters": {
        "continentalness": [
          -0.19,
          1.0
        ],
        "depth": 0,
        "erosion": 0,
        "humidity": 0,
        "offset": 0,
        "temperature": 0,
        "weirdness": 0
      }
    }
  ]
}
```

In code, you can the biome's RegistryKey is `LithostitchedBiomes.ORIGINAL_SOURCE_MARKER`.