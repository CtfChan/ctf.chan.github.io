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

- State Space Sampling of Feasible Motions for High-Performance Mobile Robot Navigation in Complex Environments  (lattice generated)


## Uniform State Sampling

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


![](/assets/uniform_state_lattice.png)


## Biased State Sampling

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

$$n_s$$ biases our the sampling of paths paths where global cost is lowest.

## Lane Sampling

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

![](/assets/lane_state_lattice.png)



In this example, we set $$ n_{l} = 1 $$.