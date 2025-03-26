---
title: Simulation of a Rotating Pendulum 
date: 2025-01-25 10:00:00 +0100
categories: [simulation, ode]
tags: [colab, mujoco, rk4]     # TAG names should always be lowercase
description:
  A Google Colab notebook for simulating gyroscopic angular momentum is presented.
  It achieves a high numerical stability and the simulation results match theoretical estimations.
---

Developing and testing software for controlling modern robots, e.g., with reinforcement learning,
is much cheaper and safer in simulation than with a physical system. However, the simulation
must be sufficiently accurate.

Many robots, including UAVs, have to deal with gyroscopic angular momentum and the corresponding resistive forces.
I've tried several physics engines for such a case, and the results were often unstable&mdash;small changes
in the initial conditions led to large changes in the simulated trajectory.

My [Colab Notebook](https://colab.research.google.com/drive/1mZpGmR-I76UO1jId7dski0G49QXuZrB_?usp=sharing)
simulates a simple mechanical experiment inspired by the short [video](https://www.youtube.com/watch?v=8H98BgRzpOM)
from MIT about gyroscopic precession.
The simulator uses the [MuJoCo](https://mujoco.org/) framework, and the numeric integration is performed using
the RK4 [Runge–Kutta](https://en.wikipedia.org/wiki/Runge%E2%80%93Kutta_methods) method.

The simulated system demonstrates behavior visually similar to the real one, and the quantitative characteristics
match their theoretical estimates (for example, the precession frequency). The underlying theory can be found in the
[Precession of a Gyroscope](https://openstax.org/books/university-physics-volume-1/pages/11-4-precession-of-a-gyroscope)
chapter of the OpenStax physics book. The [Michel van Biezen lecture](https://www.youtube.com/watch?v=qS_dcNqs3d4)
describes a setting very similar to ours, but makes some additional assumptions.

The notebook also demonstrates how some properties of a more complex system, e.g., its moments of inertia,
can be determined experimentally.

Please see
the [notebook](https://colab.research.google.com/drive/1mZpGmR-I76UO1jId7dski0G49QXuZrB_?usp=sharing)
itself!

