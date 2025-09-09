# Houdini To Maya Cd Color Data
This tutorial explains how to transfer color data (Cd) from Houdini to Maya.

We will export geometry out of Houdini into Alembic format and import the geometry into Maya.

We will show methods to render shaders using the (Cd) values in these render engines:
* Arnold
* V-Ray
* Renderman
* Redshift

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

### References:
[Houdini to Maya part1: Multiple Vertex Color Data](https://ddankhazi.com/2020/12/16/houdini-to-maya-part1-multiple-vertex-color-data/)
