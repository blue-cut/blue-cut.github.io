---
layout: default
title:  "A Simple 2D Beacons-based Positioning System"
author: Aethor
date:   2022-10-28 16:52:00:00 +0800
categories: robotics
---

Positioning systems in robotics are important for a wide range of applications. We are competing in the [France Robotics Cup](https://www.coupederobotique.fr/) as Sharp'Attack, and one of this year's project is a positioning system based on a lidar and a set of beacons.


# Problem statement

![Problem statement]({{site.baseurl}}/assets/images/2d_beacon/problem_statement.svg)

We formulate the absolute positioning problem as trying to determine our robot's position and angle in a global frame, which we can represent by the tuple $$ (x_r, y_r, \theta_r) $$.


# Lidar and beacons tracking

Using our own tracking library, we are able to track several beacons using our [RPLidar A2M8](https://www.slamtec.com/en/Lidar/A2) lidar. As beacon tracking is outside of the scope of this article, we can simplify and suppose that at each timestep $$ t $$, we measure the distance and angle to all beacons.


# Computing the robot's position using trilateration

![Position estimation]({{site.baseurl}}/assets/images/2d_beacon/pos_estimation.svg)

We can compute the position of our robot using the trilateration principle. Let's first assume the beacon distance measures is perfect and completely reflects reality. To be able to unambiguously determine the position of the robot, we need 3 beacons. Let's name them $$ B_1 $$, $$ B_2 $$ and $$ B_3 $$. We define their position as $$ (x_1, y_1) $$, $$ (x_2, y_2) $$ and $$ (x_3, y_3) $$ respectively.

As we measure distance $$ d_1 $$ to beacon $$ B_1 $$, we know that the robot can only be around a circle of center $$ (x_1, y_1) $$ and of radius $$ d_1 $$. Adding distance $$ d_2 $$ to beacon $$ B_2 $$ gives us two possible locations for our robot (the two intersections of the two circles). Finally, with distance $$ d_3 $$ to beacon $$ B_3 $$, the position of our robot can be completely determined (see schema).

Recall the the equation of a circle of center $$ (x_c, y_c) $$ and radius $$ r $$ is given by $$ r^2 = (x - x_c)^2 + (y - y_c)^2 $$. We then have: 

$$ d_1^2 = (x_r - x_1)^2 + (y_r - y_1)^2 $$

$$ d_2^2 = (x_r - x_2)^2 + (y_r - y_2)^2 $$

$$ d_3^2 = (x_r - x_3)^2 + (y_r - y_3)^2 $$


We can linearize this system by substracting line 2 and 3 to line 1:

$$ d_1^2 - d_2^2 = x (-2x_1 + 2x_2) + y (-2y_1 + 2y_2) + x_1^2 + y_1^2 -x_2^2 -y_2^2 $$

$$ d_1^2 - d_3^2 = x (-2x_1 + 2x_3) + y (-2y_1 + 2y_3) + x_1^2 + y_1^2 -x_3^2 -y_3^2 $$


Finally:

$$ 2 (-x_1 + x_2) x_r + 2 (-y_1 + y_2) y_r = d_1^2 - d_2^2 - x_1^2 - y_1^2 + x_2^2 + y_2^2 $$

$$ 2 (-x_1 + x_3) x_r + 2 (-y_1 + y_3) y_r = d_1^2 - d_3^2 - x_1^2 - y_1^2 + x_3^2 + y_3^2 $$


Which you can solve using your favorite computer tool (I just hope it's `scipy` and not `Matlab` !).


## Accounting for measurement imperfections

The previously shown system can only be solved under perfect conditions. In reality, distance measurements will be subject to noise, which will lead to a situation where the above system of equations does not have a solution.

Thankfully, we can use the concept of *pseudo-inverse* to solve this problem. First, recall the solving the following system for $$ x_1 $$ and $$ x_2 $$:


$$ a_{1,1} x_1 + a_{1,2} x_2 = b_1 $$

$$ a_{2, 1} x_1 + a_{2, 2} x_2 = b_2 $$


is equivalent to finding the inverse of matrix A in the following equation:

$$ Ax = b $$

since we can find $$ x $$ when knowing $$ A^{-1} $$:

$$ A^{-1}Ax = A^{-1}b $$

$$ x = A^{-1}b $$


But in our case, $$ A $$ might not be inversible since we might not have a solution... however, we can still find its pseudo-inverse $$ A^+ $$ ! 

The pseudo-inverse of a matrix is a generalization of the inverse having certain properties. Notably, we can use it to compute a "good enough" $$ x $$ approximation, $$ \hat{x} = A^+b $$. The approximation is "good enough" in the sense that it minimizes the square error:

$$ E_{\hat{x}} = min_{x} || Ax - b ||_2 $$

Again, $$ A^+ $$ can be computed using your favorite software.


# Computing the angle of the front of the robot

![Angle estimation]({{site.baseurl}}/assets/images/2d_beacon/rot_estimation.svg)

Once the robot's position has been computed, one can easily compute the angle $$ \theta $$ of the front of the robot. We denote the angle to beacon $$ B_1 $$ as $$ \varphi_1 $$, and the vector from the robot to the beacon $$ B_1 $$ as $$ \vec{v} $$. As $$ \theta $$ is the full angle between $$ \vec{x} $$ and $$ \vec{v} $$ minus the angle to the beacon, we simply have :

$$ \theta = atan2(v_y, v_x) - \varphi_1 $$
