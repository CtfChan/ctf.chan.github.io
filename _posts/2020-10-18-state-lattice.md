---
title: "Robotics Blog: State Lattice"
tags: [robotics]
style: fill
color: dark
description: Making lots of pretty curves
---
<script type="text/javascript"
        src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.0/MathJax.js?config=TeX-AMS_CHTML"></script>


Work in Progress


The methods outlined in this post are based on the following paper:
- State Space Sampling of Feasible Motions for High-Performance Mobile Robot Navigation in Complex Environments  


## State Lattice Methods
State lattice methods involve precalculating a set of feasible trajectories for your robot using a given shape parametrization. At runtime, the best trajectory can be calculated quickly based on your cost function (e.g. closeness to obstacles). The three shape parametrizations $$p_{ss}$$ identified in the paper are uniform, biased and lane state sampling. In this post we will be consideration polar uniform and biased sampling. 

In all of the sampling methods below each trajectory is represented with a start pose and a goal pose. Using the lookup table for goal poses made using the trajectory generator in our previous post, we can use the $$\{s, k_0, k_f\}$$ as an initial guess in our optimization for a new trajectory.  


## Uniform State Sampling
Uniform state sampling involves uniformly sampling goal poses in a cone in front of the robot. 

$$
p_{ss} =
\begin{bmatrix}
n_p \\
n_h \\
d \\
\alpha_{min} \\
\alpha_{max} \\
\psi_{min} \\
\psi_{max} \\
\end{bmatrix}
$$

$$n_p$$ is the number samples in the terminal state position and represents the number of points along the arc to sample from.

$$n_h$$ is the  terminal heading and represents the number of final orientations to sample from at each sample point.

$$d$$ is the terminal position horizon and represents the radius of the cone.

$$\alpha_{min}$$ and $$\alpha_{max}$$ represent the angular range to sample for terminal positions.

$$\psi_{min}$$ and $$\psi_{max}$$ represent the angular range to sample for terminal headings.

The green triangle represents the robot in the diagram below. With $$n_h = 3$$ you can see that we generate 3 red trajectories to each blue dot.
![Uniform State Sampling Diagram](/assets/uniform_state_lattice.png)

In `calculateUniformPolarStates`, we first generate a series of angles uniformly and then generate our target trajectory using some simple geometry in `sampleStates`.
```cpp
// NOTE nxy = np in this code
std::vector<Pose2D> calculateUniformPolarStates(size_t nxy, size_t nh, float d,
                        float a_min, float a_max, float p_min, float p_max) {

    std::vector<float> angle_samples(nxy);
    for (int i = 0; i < nxy; ++i)
        angle_samples[i] = i / (nxy - 1.f);

    std::vector<Pose2D> states = sampleStates(angle_samples, a_min, a_max, d, p_max, p_min, nh);
    
    return states;
}

std::vector<Pose2D> sampleStates(std::vector<float>& angle_samples, float a_min,
                 float a_max, float d, float p_max, float p_min, size_t nh) {
    std::vector<Pose2D> states; 
    
    float xf, yf, yawf;
    for (float i : angle_samples) {
        float a = a_min + (a_max - a_min) * i;
        for (int j = 0; j < nh; ++j) {
            xf = d * std::cos(a);
            yf = d * std::sin(a);
            if (nh == 1)
                yawf = (p_max - p_min) / 2.f + a;
            else 
                yawf = p_min + (p_max - p_min) * j / (nh - 1.f) + a;
            Pose2D final_state {xf, yf, yawf};
            states.push_back(final_state);
        }
    }

    return states;
}
```


## Biased State Sampling
The parametrization for biased state sampling is similar to uniform state sample except we add an extra term $$n_s$$ which is the number of navigation function samples. This new parameter biases our sampling at points where the global cost function is lower. So more blue dots appear near our goal angle. 

$$
p_{ss} =
\begin{bmatrix}
n_s \\
n_p \\
n_h \\
d \\
\alpha_{min} \\
\alpha_{max} \\
\psi_{min} \\
\psi_{max} \\
\end{bmatrix}
$$

Below is heavily commented version of the `calculateBiasedPolarStates` code.
```cpp
std::vector<Pose2D> calculateBiasedPolarStates(float goal_angle, int ns, int nxy,
                                   int nh, int d,
                                   float a_min, float a_max,
                                   float p_min, float p_max){
    std::vector<float> as(ns-1);
    std::vector<float> cnav(ns-1);
    for (size_t i = 0; i < ns - 1; ++i) {
        as[i] = a_min + (a_max - a_min) * i / (ns - 1.f); // this is the angle used in uniform sampling
        cnav[i] = M_PI - std::abs(as[i] - goal_angle); // angle difference between uniformly sampled node and goal angle
    }

    // sum up all the angle diffs and find max element
    float cnav_sum = std::accumulate(cnav.begin(), cnav.end(), 0.f);
    float cnav_max = *std::max_element(cnav.begin(), cnav.end());

    // normalise the distribution of cnav
    for (float& cnavi : cnav) {
        cnavi = (cnav_max - cnavi) / (cnav_max * ns - cnav_sum);
    }

    // cumalative sum
    std::vector<float> cumsum_cnav(cnav.size());
    std::partial_sum(cnav.begin(), cnav.end(), cumsum_cnav.begin());

    // output angles based on cumulative sume
    std::vector<float> di;
    int li = 0;
    for (int i = 0; i < nxy; ++i ) {
        for (int ii = li; ii < ns-1; ++ii) {
            if ( ii*(1.f/ ns) >= i / (nxy - 1.f) ) {
                di.push_back(cumsum_cnav[ii]);
                li = ii - 1;
                break;
            }
        }
    }
    
    // use sample states as before 
    auto states = sampleStates(di, a_min, a_max, d, p_max, p_min, nh);
    return states;  
}
```



## Lane Sampling
Lane sampling samples based on the geometry of a road lane. 

$$
p_{ss} =
\begin{bmatrix}
l_{center} \\
l_{heading} \\
l_{width} \\
v_{width} \\
d \\
n_{p} \\
n_{l}
\end{bmatrix}
$$

$$ l_{center} $$ is the lane centerline.

$$l_{heading} $$ is the lane heading.

$$l_{width} $$ is the lane width.

$$ v_{width} $$ is the vehicle width.

$$ d $$ is the lane horizon.

$$ n_{p} $$ is the number of lateral offsets to sample from along the center position.

$$ n_{l} $$ is the number of lanes. In this example we kept this to one.


The figure below represents how each variable is used. Our robot (green triangle) seeks to reach the red goal pose (red triangle). The robot is currently in frame a and the target pose is in frame b. $$l_{heading} $$ parametrized the 2D rotation matrix to transform a point from frame b to frame a.

![](/assets/lane_state_lattice.png)


Recall a 2D rotation matrix is parametrized as follows.

$$ 
\begin{bmatrix}
cos(\theta)  & -sin(\theta)\\
sin(\theta) & cos(\theta)\
\end{bmatrix}
$$

The function below illustrates how each of these variables are used:
```cpp
std::vector<Pose2D> calculateLaneStates(
    float l_center, float l_heading, float l_width, 
    float v_width, float d, size_t nxy) { 
    // apply rotation matrix to the points
    float xc = d * std::cos(l_heading) - l_center * std::sin(l_heading);
    float yc = d * std::sin(l_heading) + l_center * std::cos(l_heading);

    // move the center point by a small delta to the left or right 
    // depending on how much "wiggle room" is left in the lane
    std::vector<Pose2D> states; states.reserve(nxy);
    for (size_t i = 0; i < nxy; ++i) {
        float delta = -0.5*(l_width-v_width) + (l_width-v_width) * (i / (nxy - 1.f));
        float xf = xc - delta * std::sin(l_heading);
        float yf = yc + delta * std::cos(l_heading);
        float yawf = l_heading;
        Pose2D target_state{xf, yf, yawf};
        states.push_back(target_state);
    }

    return states;
}

```




