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

### Important Note:
- Houdini particles to Maya does not support Alembic animation as far as I know. The only way to do an animation is to export a `bgeo.sc` sequence out of Houdini using the `ROP Geometry Output` node and import it using Houdini Engine in Maya. For single frame particle distributions, you can use the `ROP Alembic Output` node and export an Alembic (exporting a frame range will only result in a single frame in Maya)

---
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

---
Common Houdini particle/point attributes and their typical mapped equivalents in Maya particles when transferred:

| Houdini Attribute | Maya Attribute | Data Type | Description |
| :-- | :-- | :-- | :-- |
| `P` | `position` and `worldPosition` | Vector3 (float) | Particle or point position |
| `v` | `velocity` | Vector3 (float) | Particle velocity |
| `force` | `acceleration` | Vector3 (float) | Particle acceleration/force |
| `N` (normal) | `N` | Vector3 (float) | Rotation in Euler angles (degrees) |
| `orient` | N/A | Vector4 (float) | Need to convert the quaternion rotation to Vector3 (float), name it `rotationPP` |
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

This table covers the main attributes commonly transferred from Houdini to Maya particles via export/import.[^1][^2][^3][^4]

[^1]: https://forums.odforce.net/topic/27646-solved-per-particle-attributes-maya-to-houdini-via-alembic/

[^2]: https://www.sidefx.com/docs/houdini/maya/particle.html

[^3]: https://www.sidefx.com/forum/topic/40954/

[^4]: https://forums.odforce.net/topic/37227-houdini-alembic-export-to-maya-with-instancing/

---
## VEX:
### pscale_and_instanceID_attribwrangle
~~~ C linenumbers
float u = rand(@ptnum);
@pscale = chramp("ramp_scale", u);

int instances = chi("number_of_instances");
int index  = int(chramp("ramp_ratio", u) * (instances-1));
i@instanceId = index;

// Dont create i@particleId because Arnold render gets confused
~~~
### path_attribwrangle
~~~ C linenumbers
string name = "instance_" + itoa(@class);
s@path = 'obj/' + name + '/' + name + 'Shape';
~~~
### orient_attribwrangle
~~~ C linenumbers
vector up = {0, 1, 0};
@orient = quaternion(maketransform(@N, up));
~~~
### rotationPP_attribwrangle
~~~ C linenumbers
v@rotationPP = degrees(quaterniontoeuler(@orient, 0));
~~~
### cd_attribwrangle
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
[Houdini To Maya Particle Instancers and Attributes.zip](https://github.com/GuyMicciche/Tutorial-Assets/blob/main/Houdini%20To%20Maya%20Particle%20Instancers%20and%20Attributes/Assets/Houdini%20To%20Maya%20Particle%20Instancers%20and%20Attributes.zip)

### References:
[Maya - Vray Particle Color](https://www.youtube.com/watch?v=ZbRNo6X5Grk)
[Passing rgbPP to Instanced Objects](http://www.particle-effects.com/2014/06/passing-rgbpp-to-instanced-objects.html)
