// Parameters
float initial_distance = 0.01;
float increment = 0.05;
int max_iterations = 15;  // Set a loop limit
int iteration = 0;

float prim;
float uv;
vector hit_pos1, hit_pos2;
int forward, backward;
vector original_P = @P;  // Store the original position
vector N;

// Start with the initial distance
float ray_distance = initial_distance;

// Loop until all points are settled or max iterations reached
while (iteration < max_iterations)
{
    // Check if this point is in the MissedPoints group
    if (!inpointgroup(0, "MissedPoints", @ptnum))
        break;  // Skip this point if it’s not in the group

    // Reset MissedPoints group for this point
    setpointgroup(0, "MissedPoints", @ptnum, 0);
    @Cd = {1, 1, 1};

    // Calculate the ray direction based on the current distance
    N = normalize(v@N) * ray_distance;

    // Cast a ray in the direction of the normal
    forward = intersect(1, @P, N, hit_pos1, prim, uv);
    int forward_hit = (forward != -1);

    // Cast a ray in the direction of the inverted normal
    backward = intersect(1, @P, -N, hit_pos2, prim, uv);
    int backward_hit = (backward != -1);

    int bothHit = (forward_hit > 0 && backward_hit > 0);

    if (bothHit)
    {
        vector hit_N1 = normalize(prim_normal(1, int(prim), hit_pos1));
        vector hit_N2 = normalize(prim_normal(1, int(prim), hit_pos2));
        float dot1 = dot(normalize(v@N), hit_N1);
        float dot2 = dot(normalize(v@N), hit_N2);
        if (dot1 > dot2)
            @P = hit_pos1;
        else
            @P = hit_pos2;
    }
    else if (forward_hit)
    {
        @P = hit_pos1;
    }
    else if (backward_hit)
    {
        @P = hit_pos2;
    }
    else
    {
        // If no hit, add point to MissedPoints group and color it red
        setpointgroup(0, "MissedPoints", @ptnum, 1);
        @Cd = {1, 0, 0};
    }

    // Increase the ray distance for the next iteration
    ray_distance += increment;

    // Increment the iteration counter
    iteration++;
}
