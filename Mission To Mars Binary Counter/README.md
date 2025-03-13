
# Mission To Mars Binary Counter

A presentation showing how to make a binary counter without keyframes using Python in Cinema 4D and Maya, VEX in Houdini, and Expressions in After Effects. 

The inspiration for this comes from the movie Mission To Mars and my love for cinematography and math! Please enjoy!


## Cinema 4D Python Effector Code:

```python
import c4d, math

# Parameter Controls (Attach these as User Data)
animation_length = op[c4d.ID_USERDATA, 1]  # Add a float User Data for this
rotation_interval = op[c4d.ID_USERDATA, 2]  # Add a float (0 to 1) User Data for this
easing_enabled = op[c4d.ID_USERDATA, 3]  # Add a checkbox User Data for this

def easeInOutSine(t: float) -> float:
    """Applies a sine-based easing function for smooth interpolation from 0 to 1."""
    return (0.5 * (1 - math.cos(math.pi * t))) if easing_enabled else t

def main():
    """Binary counter rotation effect for MoGraph Cloner in Cinema 4D."""
    md = c4d.modules.mograph.GeGetMoData(op)
    if md is None:
        return False

    # Get clone count and transformation matrices
    count = md.GetCount()
    matrices = md.GetArray(c4d.MODATA_MATRIX)

    # Get current frame
    frame = doc.GetTime().GetFrame(doc.GetFps())
    time = frame / animation_length

    for i in range(count):
        interval = 2 ** i
        steps = math.floor(time / interval)

        matrices[i] *= c4d.utils.MatrixRotX(math.radians(90 * steps))

        phase = time % interval
        if phase < rotation_interval:  # First half of interval = rotating
            matrices[i] *= c4d.utils.MatrixRotX(math.radians(90 * easeInOutSine(phase / rotation_interval) + 90))

    # Apply updated transformations
    md.SetArray(c4d.MODATA_MATRIX, matrices, True)

    return True
```

## After Effects Expression:

```javascript
// === Binary Counter Rotation with Easing ===
// User-defined settings
var animationLength = 12;  // How long the rotation animation will be, including rest period
var rotationInterval = 0.5; // 0 = instant, 0.5 = half animation/half rest, 1.0 = always rotating
var easingEnabled = true; // Toggle easing (true = smooth, false = linear)

// Helper easing function (Sine ease-in-out)
function easeInOutSine(t) {
    return easingEnabled ? (0.5 * (1 - Math.cos(Math.PI * t))) : t;
}

// Get the current frame number (rounded to integer)
var frame = timeToFrames();

// Ensure animationLength is never 0 to avoid division errors
animationLength = Math.max(animationLength, 1);

// Get normalized time
var timeNorm = frame / animationLength;

// Get layer index (used for binary timing)
var layerIndex = Math.max(index - 1, 0);  // Ensure index starts at 0

// Compute binary timing
var interval = Math.pow(2, layerIndex);  // Binary progression (1x, 2x, 4x, etc.)
var steps = Math.floor(timeNorm / interval); // Count full rotations

// Compute base rotation (90-degree steps)
var rotationAngle = steps * 90;

// Compute smooth rotation over rotation interval
var phase = timeNorm % interval;
if (phase < rotationInterval && rotationInterval > 0) {
    rotationAngle += 90 * easeInOutSine(phase / rotationInterval) + 90;
}

// Return the computed rotation
rotationAngle;
```

## Maya Python Mash Node Code:

```python
import openMASH
from maya.api.OpenMaya import MVector as vec
import math

animation_length = cmds.getAttr(thisNode + ".animationLength")  # How long the rotation animation will be, including rest period
rotation_interval = cmds.getAttr(thisNode + ".rotationInterval") # 0 is instant, 0.5 is half animation half rest, and 1.0 is always rotating
easing = cmds.getAttr(thisNode + ".easing")

def easeInOutSine(t):
    """Easing function: Sine-based smooth transition from 0 to 1"""
    return (0.5 * (1 - math.cos(math.pi * t))) if easing else t

def updateRotations(data):
    frame = data.getFrame()
    time = frame / animation_length  # Current time in seconds

    # Initialize rotation array
    rotations = [0] * data.count()

    for i in range(data.count()):
        # Calculate rotation steps for binary progression
        interval = 2 ** i  # Each torus waits twice as long as previous
        steps = math.floor(time / interval)
        
        # Calculate rotation angle (90° per step)
        rotations[i] = 90 * steps
        
        # Optional: Add smooth rotation over 0.5 seconds
        phase = time % interval
        if phase < rotation_interval:  # First half of interval = rotating
            rotations[i] += 90 * easeInOutSine(phase / rotation_interval) + 90

    # Apply rotation output for MASH
    data.outRotation = [vec(rot, 0, 0) for rot in rotations]

# Initialize MASH data
md = openMASH.MASHData(thisNode)
updateRotations(md)
md.setData()
```

## Houdini Attribute Wrangle VEXpression:

```javascript
float frame = @Frame-1;  // Get current frame
int i = @ptnum;  // Get current point index

// Parameters (UI controls in Houdini)
int animation_length = chi("animation_length");  // Total cycle time (frames)
float rotation_interval = chf("rotation_interval");  // Ratio of rotation vs rest period (0 to 1)

// Rotation easing function
float easeInOutSine(float t) {
    int easing = chi("easing");  // Enable/Disable easing
    return easing ? (0.5 * (1 - cos(M_PI * t))) : t;
}

// Binary progression timing
float step_time = animation_length * pow(2, i);
float time = frame / animation_length;  // Normalized time

// Compute rotation interval
int interval = pow(2, i);
// Compute rotation steps
int steps = int(floor(time /interval));
// Calculate rotation angle (90° per step)
float rotation = 90 * steps;

// Calculate phase position in cycle
float phase = time % interval;
if (phase < rotation_interval) {  // Rotation interval (e.g., first 50% of step time)
    rotation += 90 * easeInOutSine(phase / rotation_interval) + 90;
}

// Apply rotation using quaternion
p@orient = quaternion(radians(rotation), {1,0,0});

```
