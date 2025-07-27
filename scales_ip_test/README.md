# Scales IP Test
This is a solution for post-fixes on scales and feather IPs based on a simple mathematical method relayed to me by a rigging supervisor at the same company I was working at. He had a successful implementation working in Maya and I had translated it into Houdini using VEX. So far I've recreated the solution only with a single row of scales, and it's a very robust solution for that particular problem. 

The next step for this tool is to find a method for resolving intersections between scales when there are a bunch of them on the surface of a snake body. However, on a larger set of scales it requires more consideration for more axes of rotation due to all the different angles of intersections that the scales can have. I had previously implemented a solution that worked relatively well, but not perfect as I couldn't get the loops working in Houdini the way I wanted them to at the time. Tackling that would be my first issue, then I will need to address the problem of continuity with the fixes (as fixing the scales per frame does not necessarily translate smoothly to consecutive frames). One solution in mind is to use PCA filtering. 

### Scales intersecting each other (before) and scales with IP fixes (after):

<img src="https://github.com/Jechli/Houdini/blob/main/scales_ip_test/scales_ip_before_after.gif" alt="Gif of scales IPs before and after fix solution." width="500">

### Main portion of SOP network for running the calculations: 

<img src="https://github.com/Jechli/Houdini/blob/main/scales_ip_test/main_network.png" alt="Main portion of the node network for fixing intersections." width="500">

### Fetching the next scale (primitive wrangle node inside foreach loop):

<img src="https://github.com/Jechli/Houdini/blob/main/scales_ip_test/primwrangle_next_scale.png" alt="Fetching the next scale with a primitive wrangle node." width="500">

### Calculations for the angle of rotation of each scale (attribute wrangle node inside foreach loop):

<img src="https://github.com/Jechli/Houdini/blob/main/scales_ip_test/attribwrangle_intersect_hit.png" alt="Angle of rotation calculations within attribute wrangle node." width="500">

**Full VEX script:**

```
// Get root and tip of colliding scale
vector root_b, tip_b;
for (int i=0; i<npoints(1); i++) {
    if (point(1, "is_root", i)) {
        root_b = point(1, "root", i);
    }
    if (point(1, "is_tip", i)) {
        tip_b = point(1, "tip", i);
    }
}
vector ray_b = tip_b - root_b;

// Get root and tip of current scale
vector root_a, tip_a, N_a;
for (int i=0; i<npoints(0); i++) {
    if (point(0, "is_root", i)) {
        root_a = point(0, "root", i);
        N_a = point(0, "N", i);
    }
    if (point(0, "is_tip", i)) {
        tip_a = point(0, "tip", i);
    }
}
vector ray_a = tip_a - root_a;
vector hit_pt;
float hit_u, hit_v;
int hit_prim = intersect(1, root_a, ray_a, hit_pt, hit_u, hit_v);

// Rotate scale if there is a hit
if (hit_prim >= 0) {
    float theta = acos(dot(ray_a, ray_b) / (length(ray_a)*length(ray_b)));
    float beta = PI - theta;
    float length_A = length(hit_pt - root_a);
    float length_B = length(ray_a);
    float alpha = asin((length_A / length_B) * sin(beta));
    float gamma = PI - alpha - beta;
    vector rot_axis = normalize(cross(ray_a, N_a));
    
    matrix3 m = ident(); 
    rotate(m, gamma, rot_axis);
    for (int i=0; i<npoints(0); i++) {
        vector new_pt = ((point(0, "P", i)-root_a)*m)+root_a;
        setpointattrib(0, "P", i, new_pt);
    }
}

```
