# Houdini to Maya Particle Instancers and Attributes
This tutorial explains one way to export particles out of Houdini and import them into Maya. 

## Key Points:

1. We will build a particle node tree with position (P), color (Cd), normal (N), scale/radius (pscale), and id (id) attributes. After which we will export the particles to Alembic format. 
2. We will build a simple file importer subnet (HDA) in Houdini, and use Houdini Engine in Maya to import our particles as an nParticle object.
3. We will use an Instancer in Maya to instance geometry onto our particles.
4. We will learn about the nuances, setups, render engine quirks to shade and texture our particle system.

We will show methods to render shaders using the (Cd) values in these render engines:
* Arnold
* V-Ray
* Renderman
* Redshift

## All Render Engines 
- In Maya, in the Attribute Editor for the nParticle shape, go to `Add Dynamic Attributes`, click on `General`, then `Particle` tab, and manually add the `rotationPP` attribute.
- Still in the nparticle shape, go to `Instancer (Geometry Replacement)` and check `Allow All Data Types`.
- Still in the nparticle shape, go to `General Options` and set the following options:
  - Set `Position` to `World Position`
  - Set `Scale` to `Radius PP`
  - Set `Object Index` to `Particle ID` (Except Arnold, set to another attribute you created in Houdini, for example `instanceId`)
- Still in the nparticle shape, go to `General Options` and set the following options:
  - Set `Rotation` to `rotationPP`

## V-Ray Quirks:
- In Maya, create `VRay Mtl` material and a `particleSamplerInfo` node. Connect the `Rgb PP` from the sampler node to the `Diffuse Color` of the shader.
- In Maya, in the Attribute Editor for the nParticle shape, click on Attributes > VRay > and check `per-particle attribute export`, then scroll down to `Extra V-Ray Attribute` and check `RGB`.

## Renderman Quirks:
- In Maya, create `PxrSurface` material and a `PxrAttribute` or `PxrPrimvar` node. In either node, set the `Variable Name` to `rgbPP` and the `Variable Type` to `color`. Connect the `Result RGB` from the node to the `Diffuse Color` of the shader.

## Arnold Quirks:
- In Maya, create `aiStandardSurface` material and a `particleSamplerInfo` node. Connect the `Rgb PP` from the sampler node to the `Base Color` of the shader.
- In Maya, in the Attribute Editor for the nParticle shape, click on `Arnold` rollout:
  -  Uncheck `Primary Visibility`
  -  Export Attributes, type in `rgbPP`
- In Houdini, do not export a `id` or `particleId` attribute, you must rename.

Here is a table summarizing common Houdini particle/point attributes and their typical mapped equivalents in Maya particles when transferred via Alembic:

| Houdini Attribute | Maya Attribute | Data Type | Description |
| :-- | :-- | :-- | :-- |
| `P` | `position` and `worldPosition` | Vector3 (float) | Particle or point position |
| `v` | `velocity` | Vector3 (float) | Particle velocity |
| `force` | `acceleration` | Vector3 (float) | Particle acceleration/force |
| `N` (normal) | `N` | Vector3 (float) | Rotation in Euler angles (degrees) |
| `orient` | N/A | Vector4 (float) | Need to convert the quaternion rotation to Vector3 (float) |
| `Cd` (color) | `rgbPP` | Vector3 (float) | Per-particle color |
| `pscale` | `radiusPP` | Float | Particle scale or radius |
| `life` | `finalLifespanPP` | Float | Particle lifespan |
| `id` | `particleId` | Float | Particle unique ID (If using Arnold, rename to something different, like `instanceId` |
| `age` | `age` | Float | Particle age |

### Notes:

- Maya expects rotation as Euler angles in `rotationPP`, so if `orient` (quaternion) is used in Houdini, you will need to convert it to Euler before or after export.
- Colors and scale transfer smoothly if attributes are named correctly and exported.
- Some attributes may need manual promotion or conversion depending on export/import pipeline specifics.
- Arnold render will glitch and not render the Maya Instancer if you export `id` or `particleId` out of Houdini, you will need a custom name (Arnold only).

This table covers the main attributes commonly transferred from Houdini to Maya particles through Alembic export/import.[^1][^2][^3][^4]

<div style="text-align: center">⁂</div>

[^1]: https://forums.odforce.net/topic/27646-solved-per-particle-attributes-maya-to-houdini-via-alembic/

[^2]: https://www.sidefx.com/docs/houdini/maya/particle.html

[^3]: https://www.sidefx.com/forum/topic/40954/

[^4]: https://forums.odforce.net/topic/37227-houdini-alembic-export-to-maya-with-instancing/

---
## VEX:
### attributewrangle1
~~~ C linenumbers
// Maya import separate geometry
string name = "instance_" + itoa(@class);
s@path = 'obj/' + name + '/' + name + 'Shape';
~~~
### attributewrangle2
~~~ C linenumbers
// Use point number as seed for randomness
float seed = @ptnum;

// Get random float from seed
float rand_val = rand(seed);

// Normalize rand_val to 0-100 scale
float val100 = rand_val * 100.0;

// Create masks for each range
float mask1 = (val100 >= 0 && val100 < 25) ? 1.0 : 0.0;
float mask2 = (val100 >= 25 && val100 < 50) ? 1.0 : 0.0;
float mask3 = (val100 >= 50 && val100 < 75) ? 1.0 : 0.0;
float mask4 = (val100 >= 75 && val100 <= 100) ? 1.0 : 0.0;

// Remap rand_val from each range back to 0-1 for the ramp lookup
float norm1 = fit(val100, 0, 25, 0, 1);
float norm2 = fit(val100, 25, 50, 0, 1);
float norm3 = fit(val100, 50, 75, 0, 1);
float norm4 = fit(val100, 75, 100, 0, 1);

// Create a color ramp parameter on the wrangle node (name: "color_ramp")
// Use the ramp to map random float to color
v@Cd   = mask1 * floor(chramp("color_ramp", norm1) * 100)/100.0;
v@Cd2  = mask2 * floor(chramp("color_ramp", norm2) * 100)/100.0;
v@Cd3  = mask3 * floor(chramp("color_ramp", norm3) * 100)/100.0;
v@Cd4  = mask4 * floor(chramp("color_ramp", norm4) * 100)/100.0;

// Mark Cd attribute as color for export
setattribtypeinfo(0, "point", "Cd", "color");
setattribtypeinfo(0, "point", "Cd2", "color");
setattribtypeinfo(0, "point", "Cd3", "color");
setattribtypeinfo(0, "point", "Cd4", "color");
~~~

## All Assets:
[Houdini To Maya Cd Color Data.zip](https://github.com/GuyMicciche/Tutorial-Assets/blob/main/Houdini%20To%20Maya%20Cd%20Color%20Data/Assets/Houdini%20To%20Maya%20Cd%20Color%20Data.zip)

### References:
[Houdini to Maya part1: Multiple Vertex Color Data](https://ddankhazi.com/2020/12/16/houdini-to-maya-part1-multiple-vertex-color-data/)
