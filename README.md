# Common attributes standardisation

Version 0.1

## Motivation

OSL has been tremendously successful at standardising shading but there are still non standardised areas, especially around each renderer’s internal nomenclature.
Although the same compiled shaders can run in any renderer, it is difficult in 2024 to write a shader that performs correctly in multiple renderers because each renderer names primitive attributes differently.

For example, if you wish to use the surface tangent:

| Renderer | OSL call |
| -: | :- |
| RenderMan | `getattribute("Tn", tangent)` |
| Arnold | `Vector tangent = normalize(dPdu)` |
| Vray | `Vector tangent = normalize(dPdu)` |
| Cycles | `getattribute("geom:tangent", tangent)` |
| RedShift | `Vector tangent = normalize(dPdu)` |
| 3Delight | *TBD* |

In this case, the shader writer hopes that u, v is not the parametric uv on meshes, otherwise dPdu may be pretty useless. But there is no guarantee.

## Standard attribute names

Here is a list of proposed names for common attributes, inspired by various sources. Of course, some of these attributes should be computed on-demand, if possible.

### Renderer Infos

Shader writers have no way to tell which renderer is executing our code. I think this would be useful as there are other types of discrepancies that need to be taken care of by shader writers, like the space in which some attributes are returned or the meaning of uv (face or mesh-wide coordinates ?).

This could be fixed by adding 2 standard attributes.

| Attribute | Type | Description | |
| - | - | - | - |
| `"osl:version"` | int | Major*10000 + Minor*100 + patch. | ![std]( https://img.shields.io/badge/std-grey) |
| `"shader:shadername"` | string | Name of the shader master. | ![std]( https://img.shields.io/badge/std-grey) |
| `"shader:layername"` | string | Name of the layer instance. | ![std]( https://img.shields.io/badge/std-grey) |
| `"shader:groupname"` | string | Name of the shader group. | ![std]( https://img.shields.io/badge/std-grey) |
| `"renderer:name"` | string | lower-case name i.e. `"renderman"` | ![new]( https://img.shields.io/badge/new-blue) |
| `"renderer:version"` | int[2] | [major, minor] i.e. `[26, 2]` | ![new]( https://img.shields.io/badge/new-blue) |

### Camera

These attributes are already defined in the OSL standard, so this is just for completeness.

| Name | Type | Description | |
| - | - | - | - |
| `"camera:resolution"` | int[2] | Image resolution. | ![std]( https://img.shields.io/badge/std-grey) |
| `"camera:pixelaspect"` | float | Pixel aspect ratio. | ![std]( https://img.shields.io/badge/std-grey) |
| `"camera:projection"` | string | Projection type (e.g., "perspective", "orthographic", etc.) | ![std]( https://img.shields.io/badge/std-grey) |
| `"camera:fov"`  | float | Field of fiew. | ![std]( https://img.shields.io/badge/std-grey) |
| `"camera:clip near"`  | float | Near clip distance. | ![std]( https://img.shields.io/badge/std-grey) |
| `"camera:clip far"`  | float | Far clip distance. | ![std]( https://img.shields.io/badge/std-grey) |
| `"camera:clip"`  | float[2] | Near and far clip distances. | ![std]( https://img.shields.io/badge/std-grey) |
| `"camera:shutter open"`  | float | Shutter open time. | ![std]( https://img.shields.io/badge/std-grey) |
| `"camera:shutter close"`  | float | Shutter close time. | ![std]( https://img.shields.io/badge/std-grey) |
| `"camera:shutter"`  | float[2] | Shutter open and close times. | ![std]( https://img.shields.io/badge/std-grey) |
| `"camera:screen window"` | float[4] | Screen window (xmin, ymin, xmax, ymax). | ![std]( https://img.shields.io/badge/std-grey) |

### Geometry

> [!NOTE]
> Would it be possible to standardize the space in which geometric quatities are returned ?

#### Geometric Quantities

| Attribute | Type | Description | |
| - | - | - | - |
| `"geom:tangent"` | vector | The normalised surface tangent [^1]. | ![new]( https://img.shields.io/badge/new-blue) |
| `"geom:undisplaced_P"` | point | The surface position before displacement. | ![new]( https://img.shields.io/badge/new-blue) |
| `"geom:undisplaced_N"` | normal | The surface normal before displacement. | ![new]( https://img.shields.io/badge/new-blue) |
| `"geom:reference_P"` | normal | The surface position in reference pose in object space. | ![new]( https://img.shields.io/badge/new-blue) |
| `"geom:reference_N"` | normal | The surface normal in reference pose in object space. | ![new]( https://img.shields.io/badge/new-blue) |
| `"geom:reference_WP"` | normal | The surface position in reference pose in world space. | ![new]( https://img.shields.io/badge/new-blue) |
| `"geom:reference_WN"` | normal | The surface normal in reference pose in world space. | ![new]( https://img.shields.io/badge/new-blue) |

[^1]: Should always return a usable vector, even if there is no explicit surface parameterization, i.e. no UVs on a mesh.

#### Identifiers

| Attribute | Type | Description | |
| - | - | - | - |
| `"geom:id"` | int | A unique object id, or first part of 64 bits id. | ![new]( https://img.shields.io/badge/new-blue) |
| `"geom:id2"` | int | The second part of a 64 bits unique object id. | ![new]( https://img.shields.io/badge/new-blue) |
| `"geom:instance_id"` | int | A unique object instance id. | ![new]( https://img.shields.io/badge/new-blue) |
| `"geom:curve_id"` | int | A unique curve id. | ![new]( https://img.shields.io/badge/new-blue) |
| `"geom:point_id"` | int | A unique point id. | ![new]( https://img.shields.io/badge/new-blue) |

#### classifications

| Attribute | Type | Description | |
| - | - | - | - |
| `"geom:is_mesh"` | int | 1 if object is a mesh, 0 otherwise | ![new]( https://img.shields.io/badge/new-blue) |
| `"geom:is_subdiv"` | int | 1 if object is a subdivision surface, 0 otherwise | ![new]( https://img.shields.io/badge/new-blue) |
| `"geom:is_curve"` | int | 1 if object is a curve, 0 otherwise | ![new]( https://img.shields.io/badge/new-blue) |
| `"geom:is_point"` | int | 1 if object is a point, 0 otherwise | ![new]( https://img.shields.io/badge/new-blue) |
| `"geom:is_volume"` | int | 1 if object is a point, 0 otherwise | ![new]( https://img.shields.io/badge/new-blue) |

### Rendering infos

| Attribute | Type | Description | |
| - | - | - | - |
| `"stage:displace"` | int | 1 if running in displacement context, 0 otherwise. | ![new]( https://img.shields.io/badge/new-blue) |
| `"stage:shade"` | int | 1 if running in shading context, 0 otherwise. | ![new]( https://img.shields.io/badge/new-blue) |
| `"hit:direct"` | int | 1 if running on a direct ray hit, 0 otherwise. | ![new]( https://img.shields.io/badge/new-blue) |
| `"hit:indirect"` | int | 1 if running on an indirect ray hit, 0 otherwise. | ![new]( https://img.shields.io/badge/new-blue) |
| `"hit:roughness"` | int | The incoming ray's spread, with [0:1] range. | ![new]( https://img.shields.io/badge/new-blue) |
