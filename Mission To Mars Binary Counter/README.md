'''
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
'''
