# Common attributes standardisation

Version 0.1

## Motivation

OSL has been tremendously successful at standardising shading but there are still non standardised areas, especially around each renderer’s internal nomenclature.
Although the same compiled shaders can run in any renderer, it is difficult in 2024 to write a shader that performs correctly in multiple renderers because each renderer names primitive attributes differently.

For example, if you wish to use the surface tangent:

| Renderer | OSL call |
| -: | :- |
| ![RenderMan]( https://img.shields.io/badge/RenderMan-blue) | `getattribute("Tn", tangent)` |
| ![Arnold]( https://img.shields.io/badge/Arnold-teal) | `Vector tangent = normalize(dPdu)` |
| ![!Vray](https://img.shields.io/badge/VRay-yellow) | `Vector tangent = normalize(dPdu)` |
| ![Cycles]( https://img.shields.io/badge/Cycles-orange) | `getattribute("geom:tangent", tangent)` |
| ![RedShift]( https://img.shields.io/badge/Redshift-red) | `Vector tangent = normalize(dPdu)` |

In this case, the shader writer hopes that u, v is not the parametric uv on meshes, otherwise dPdu may be pretty useless. But there is no guarantee.

## Renderer identification

Note that we have no way to tell which renderer is executing our code. I think this would be useful as there are other types of discrepancies that need to be taken care of by shader writers.

This could be fixed by adding 2 standard attributes:

| Attribute | Type | Description |
|- | - | - |
| `"renderer:name"` | string | lower-case name i.e. `"renderman"` |
| `"renderer:version"` | int[2] | [major, minor] i.e. `[26, 2]` |

## Standard attribute names

Here is a list of proposed names for common attributes, inspired by various sources.

### Geometry

> [!NOTE]
> some of these positions could be globals (Po, Non, Tn, etc).

| Attribute | Type | Description | Could be a global ? |
| - | - | - | - |
| `"geom:uv"` | float[2] | The default UV set | |
| `"geom:tangent"` | vector | The normalised surface tangent [^1] | &#10004; |
| `"geom:bitangent"` | vector | The normalised surface bitangent [^1] | &#10004; |
| `"geom:undisplaced_P"` | point | The surface position before displacement | &#10004; |
| `"geom:undisplaced_N"` | normal | The surface normal before displacement | &#10004; |

[^1]: Should always return a usable vector, even if there is no explicit surface parameterization, i.e. no UVs.