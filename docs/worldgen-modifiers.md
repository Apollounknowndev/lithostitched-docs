---
layout: page
title: Worldgen Modifiers
permalink: /worldgen-modifiers/
nav_order: 2
---

# Worldgen Modifiers

a. [Introduction](#introduction)

b. [Built-in Modifier Types](#built-in-modifier-types)

c. [Adding New Modifier Types](#adding-new-modifier-types)

d. [Modifier Predicates](#modifier-predicates)

e. [Modifier Phases](#modifier-phases)

## Introduction

Lithostitched introduces worldgen modifiers, a system where datapacks and mods can add, remove and replace values in worldgen files in a compat-friendly way.

Worldgen modifiers are stored in the `lithostitched/worldgen_modifier` folder of the `data/(namespace)` folder. This means that a worldgen modifier with the identifier `foo:bar` is stored in `data/foo/lithostitched/worldgen_modifier/bar.json`.

Here's an example worldgen modifier:

```json
{
  "type": "lithostitched:remove_features",
  "predicate": {
    "type": "lithostitched:true"
  },
  "biomes": "#minecraft:is_overworld",
  "features": [
    "minecraft:disk_sand",
    "minecraft:disk_gravel",
    "minecraft:disk_clay"
  ],
  "step": "underground_ores"
}
```

All worldgen modifiers have two fields:
- `type`, which is the identifier of the modifier type.
- `predicate`, which is a [modifier predicate](#modifier-predicates). The predicate must pass for the modifier to be applied. Optional field, if left empty the modifier will always apply.

Further fields are defined based on the `type` field. For the `lithostitched:remove_features` type, there's three additional fields: `biomes`, `features`, and `step`.

## Built-In Modifier Types

### `add_features`

The `add_features` modifier adds placed features entries to biome entries in a given generation step.

```json
{
  "type": "lithostitched:add_features",
  "biomes": "#minecraft:is_overworld",
  "features": "example:ore_ancient_debris",
  "step": "underground_ores"
}
```

- `biomes`: Either a biome, a list of biomes, or a biome tag that defines what biomes the features should be added to.
- `features`: Either a placed feature, a list of placed features, or a placed feature tag that defines what features should be added to the specified biomes.
- `step`: The generation step that the placed features should be added to. The steps are as follows:
    - `raw_generation`
    - `lakes`
    - `local_modifications`
    - `underground_structures`
    - `surface_structures`
    - `strongholds`
    - `underground_ores`
    - `underground_decoration`
    - `fluid_springs`
    - `vegetal_decoration`
    - `top_layer_modification`

### `remove_features`

The `remove_features` modifier removes placed features entries from biome entries in a given generation step.

```json
{
  "type": "lithostitched:remove_features",
  "biomes": "#minecraft:is_overworld",
  "features": [
    "minecraft:disk_sand",
    "minecraft:disk_gravel",
    "minecraft:disk_clay"
  ],
  "step": "underground_ores"
}
```

- `biomes`: Either a biome, a list of biomes, or a biome tag that defines what biomes the features should be removed from.
- `features`: Either a placed feature, a list of placed features, or a placed feature tag that defines what features should be removed from the specified biomes.
- `step`: The generation step that the placed features should be removed from.

### `add_biome_spawns`

The `add_biome_spawns` modifier adds mob spawns to biome entries.

```json
{
  "type": "lithostitched:add_biome_spawns",
  "biomes": "#c:swamp",
  "spawners": {
    "type": "minecraft:witch",
    "weight": 450,
    "minCount": 3,
    "maxCount": 3
  }
}
```

- `biomes`: Either a biome, a list of biomes, or a biome tag that defines what biomes the mobs should be added to.
- `spawns`: Either a single entry or a list of entries of spawner data. Identical format to spawner data entries in biome files, but mob categories are assigned automatically.

### `remove_biome_spawns`

The `remove_biome_spawns` modifier removes mob spawns from biome entries.

```json
{
  "type": "lithostitched:remove_biome_spawns",
  "biomes": "#minecraft:is_overworld",
  "mobs": "minecraft:zombie"
}
```

- `biomes`: Either a biome, a list of biomes, or a biome tag that defines what biomes the mobs should be removed from.
- `mobs`: Either an entity type, a list of entity types, or an entity type tag that defines what mobs should be removed.

### `replace_climate`

The `replace_climate` modifier replaces the climate settings of biome entries.

```json
{
  "type": "lithostitched:replace_climate",
  "biomes": [
    "minecraft:savanna",
    "minecraft:savanna_plateau"
  ],
  "climate": {
    "temperature": 2.0,
    "downfall": 0.25,
    "has_precipitation": true
  }
}
```

- `biomes`: Either a biome, a list of biomes, or a biome tag that defines what biomes the modifier should replace the climate of.
- `climate`: The climate settings to use for the listed biomes. Contains four fields:
  - `temperature`
  - `downfall`
  - `has_precipitation`
  - `temperature_modifier` (optional)
  
### `replace_effects`

The `replace_effects` modifier replaces the effects setting of biome entries.

```json
{
  "type": "lithostitched:replace_effects",
  "biomes": [
    "minecraft:lush_caves",
    "minecraft:dripstone_caves",
    "minecraft:deep_dark"
  ],
  "effects": {
    "sky_color": 0,
    "fog_color": 0,
    "water_color": 0,
    "water_fog_color": 0
  }
}
```

- `biomes`: Either a biome, a list of biomes, or a biome tag that defines what biomes the modifier should replace the special effects of.
- `effects`: The special effects settings to use for the listed biomes. Contains many fields, see the worldgen docs page on [biomes](https://www.worldgen.dev/docs/biomes/introduction/) for a description of each field.

### `add_structure_set_entries`

The `add_structure_set_entries` modifier adds structure entries to a structure set.

```json
{
  "type": "lithostitched:add_structure_set_entries",
  "structure_set": "minecraft:nether_complexes",
  "entries": [
    {
      "structure": "example:village_nether",
      "weight": 3
    }
  ]
}
```

- `structure_set`: The structure set the structure entries should be added to.
- `emtries`: A list of structure entries, with the same format as the `structures` field in structure set files.

### `remove_structures_from_structure_set`

The `remove_structures_from_structure_set` modifier removes structure from a structure set.

```json
{
  "type": "lithostitched:remove_structures_from_structure_set",
  "structure_set": "minecraft:shipwrecks",
  "structures": [
    "minecraft:shipwreck_beached"
  ]
}
```

- `structure_set`: The structure set the structures should be removed from.
- `structures`: A list of structures to remove from the set.

### `add_template_pool_elements`

The `add_template_pool_elements` modifier adds template pool elements to a template pools.

```json
{
  "type": "lithostitched:add_template_pool_elements",
  "template_pool": "minecraft:village/plains/houses",
  "elements": [
    {
      "weight": 2,
      "element": {
        "element_type": "minecraft:legacy_single_pool_element",
        "projection": "rigid",
        "location": "minecraft:village/desert/houses/desert_small_house_1",
        "processors": "minecraft:empty"
      }
    }
  ]
}
```

- `template_pool`: The template pool the elements should be added to.
- `elements`: A list of template pool elements, with the same format as the `elements` field in template pool files.

### `add_surface_rule`

The `add_surface_rule` modifier adds surface rule data to dimension(s).

```json
{
  "type": "lithostitched:add_surface_rule",
  "levels": [
    "minecraft:overworld"
  ],
  "surface_rule": {
    "type": "minecraft:condition",
    "if_true": {
      "type": "minecraft:noise_threshold",
      "noise": "minecraft:surface",
      "min_threshold": -64,
      "max_threshold": -0.55
    },
    "then_run": {
      "type": "minecraft:block",
      "result_state": {
        "Name": "minecraft:deepslate",
        "Properties": {
          "axis": "y"
        }
      }
    }
  }
}
```

- `levels`: A list of dimensions to apply the surface rule to.
- `surface_rule`: The surface rule to add to the dimension's surface rules. Surface rules added via worldgen modifiers run before all defined in the noise settings file for the dimension.

### `redirect_feature`

The `redirect_feature` modifier redirects the configured feature a placed feature references to another configured feature.

```json
{
  "type": "lithostitched:redirect_feature",
  "placed_feature": "minecraft:ore_tuff",
  "redirect_to": "example:ore_obsidian"
}
```

- `placed_feature`: The placed feature which should have its feature redirected.
- `redirect_to`: The configured feature that the placed feature should direct to.

### `no_op`

The `no_op` modifier does nothing. It's primary purpose is to allow worldgen modifiers to be disabled by datapacks.

```json
{
  "type": "lithostitched:no_op"
}
```

This modifier type has no further fields.

## Adding New Modifier Types

Mods can introduce modifier types for their own purposes. To do this, a new class should be added that extends `Modifier`.

In the most basic form, it looks like this:

```java
public class TestModifier extends Modifier {
    protected TestModifier() {
        super(TrueModifierPredicate.INSTANCE, ModifierPhase.NONE);
    }

    @Override
    public void applyModifier() {

    }

    @Override
    public Codec<? extends Modifier> codec() {
        return null;
    }
}
```

The constructor for the modifier returns a [modifier predicate](#modifier-predicates) and a [modifier phase](#modifier-phases).

The `applyModifier` method is ran when the game is applying all worldgen modifiers in a registry. Here is where you actually run the worldgen modifier.

The `codec` method returns the codec that the modifier uses to deserealize JSON files that use the modifier type. The codec for the modifier should use `Modifier.addModifierFields` in order to implement configurable modifier predicates.

If the worldgen modifier modifies biomes on Forge, it must implement `AbstractBiomeModifier` in order to correctly interact with Forge's biome modifier system. In the most basic form, it looks like this:

```java
public class TestForgeBiomeModifier extends AbstractBiomeModifier {
    protected TestForgeBiomeModifier() {
        super(TrueModifierPredicate.INSTANCE, new NoneBiomeModifier());
    }

    @Override
    public Codec<? extends Modifier> codec() {
        return null;
    }
}
```

The constructor for the modifier returns a modifier predicate and an equivalent Forge biome modifier. The equivalent Forge biome modifier is running the biome modifications, not Lithostitched.

## Modifier Predicates

Modifier predicates are a subsystem of the modifier system. They are used to determine when modifiers should be applied. Lithostitched comes with generic operations and a mod loaded predicate.

Here's an example modifier predicate:

```json
{
  "type": "lithostitched:any_of",
  "predicates": [
    {
      "type": "lithostitched:mod_loaded",
      "mod_id": "terralith"
    },
    {
      "type": "lithostitched:mod_loaded",
      "mod_id": "byg"
    }
  ]
}
```

All modifier predicates have a `type` field, indicating the modifier type to use. Each modifier can add more fields if it needs to.

The built-in modifier types are as follows:

- `all_of`: All modifier predicates in the `predicates` list of this modifier predicate must pass for this modifier to pass.
- `any_of`: At least one of the modifier predicates in the `predicates` list of this modifier predicate must pass for this modifier to pass.
- `not`: The modifier predicate in the `predicate` field of this modifier predicate must fail for this modifier to pass.
- `mod_loaded`: A mod with the id in the `mod_id` field must be installed for this modifier to pass.
- `true`: This modifier predicate always passes.

## Modifier Phases

Modifier phases exist so worldgen modifiers that complete different actions run in a consistent order. Modifiers that remove features from biomes will always run after all modifier that add features to biomes, for example. Modifiers that share a phase are not guaranteed to run in a consistent order. 

The modifier phases are as follows:

- `none`: Phase for modifiers to never apply. Useful for modifiers that don't use the regular modifier system for applying modifications, like Forge biome modifiers and the AddSurfaceRule modifier.
- `before_all`: Phase for modifiers that need to run before any other steps.
- `add`: Phase for modifiers that add to worldgen, such as template pool and structure set additions.
- `remove`: Phase for modifiers that remove from worldgen, such as feature and mob spawn removals.
- `replace`: Phase for modifiers that replace/modify parts of worldgen, like climate replacements and placed feature redirections.
- `after_all`: Phase for modifiers that need to run after all other steps.