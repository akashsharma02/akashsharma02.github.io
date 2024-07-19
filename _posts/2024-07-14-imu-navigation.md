---
layout: distill
title: almost everything about IMU navigation 
description: strapdown inertial navigation basics with an explanation of IMU pre-integration 
tags: distill research 
giscus_comments: false
date: 2024-07-14
featured: true
citation: true

authors:
  - name: Akash Sharma
    affiliations:
      name: The Robotics Institute, CMU
bibliography: 2024-07-14-imu-navigation.bib

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: Introduction
  - name: A (micro) micro Lie theory review for inertial navigation
    # if a section has subsections, you can add them as follows:
    # subsections:
    #   - name: Example Child Subsection 1
    #   - name: Example Child Subsection 2
  - name: Navigation with ideal IMU measurements
  - name: Navigation with real IMU measurements
  - name: Why do we care about pre-integration? 
  - name: Bias estimation & IMU initialization
  - name: References

# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
---
## Introduction 

Inertial Measurement Units (IMU) are typically used to track the position and orientation of an object body relative to a known starting position, orientation and velocity. Two configurations of inertial navigation are common <d-cite key="woodman2007introduction"></d-cite>: 
1. stable platform system where the inertial unit is placed in the global coordinate frame and does not move along with the body, and
2. strapdown navigation system where the inertial unit is rigidly attached to the moving object i.e., the IMU is in the body coordinate frame. 

These devices typically contain 

1. **Gyroscopes** which measure the angular velocity
2. **Accelerometers** which measure the total sum of linear acceleration acting on its body. $${}_b\mathbf{a}_k$$ acting on the body frame at time instant $$k$$ and the acceleration due to gravity on the body $$ -(\mathbf{R}_k^W)^{\top} {}_w \mathbf{g}$$ therefore, the measurement made by the accelerometer is $${}_b\mathbf{a}_k -(\mathbf{R}_k^W)^{\top} {}_w \mathbf{g}$$

In this blog, I cover the basics required to understand inertial navigation in robot state estimation, and also explain the motivation behind IMU pre-integration<d-cite key="forster2016manifold"></d-cite>, the defacto way to deal with inertial measurements in visual-inertial SLAM. 

## A (micro) micro lie-theory review for inertial navigation

This section is largely adapted from Sola et al. <d-cite key="sola2018micro"></d-cite>.

{% details Smooth manifold %}
A curved and differentiable (no-spikes / edges) hyper-surface embedded in a higher dimension that locally resembles a linear space $$\mathbb{R}^n$$. 
Examples: 

- 3D vectors with unit norm constraint form a spherical manifold in $$\mathbb{R}^3$$
- Rotations in a plane (2D vectors with unit norm constraint) form a circular manifold in $$\mathbb{R}^2$$
- 3D Rotations form a hyper-sphere (3-sphere) in $$\mathbb{R}^4$$
{% enddetails %} 

{% details Group %}
Group $$\mathcal{G}$$ is a set with a composition operator $$\circ$$ that for elements $$\mathcal{X}, \mathcal{Y}, \mathcal{Z} \in \mathcal{G}$$ satisfies the following axioms: 
- Closure under $$\circ: \mathcal{X} \circ \mathcal{Y} \in \mathcal{G}$$
- Identity $$\mathcal{E}: \mathcal{E} \circ \mathcal{X} = \mathcal{X} \circ \mathcal{E} = \mathcal{X}$$
- Inverse $$\mathcal{X}^{-1}: \mathcal{X}^{-1} \circ \mathcal{X} = \mathcal{X} \circ \mathcal{X}^{-1} = \mathcal{E}$$
- Associativity $$ (\mathcal{X} \circ \mathcal{Y}) \circ \mathcal{Z} = \mathcal{X} \circ (\mathcal{Y} \circ \mathcal{Z})$$
Notice the omission of commutativity
{% enddetails %}

### Lie Group

A Lie Group is a smooth manifold that appears identical i.e., has identical curvature and properties at every point on the manifold (such as the sphere and circle) and also satisfies the group properties. 
Examples: 
* The unit complex number group $$\mathbf{S}^1: \mathbf{z} = \cos \theta + i \sin \theta = e^{i\theta}$$ under complex multiplication forms a lie manifold. The unit norm of the complex numbers forms a 1-sphere or circle in $$\mathbb{R}^2$$ 

### Lie Group Action

Elements of the Lie Group can act on elements from other sets. For example, a unit quaternion $$ \mathbf{q} \in \mathbb{H}$$ (a lie group element) acts on an element $$\mathbf{x} \in \mathbb{R}^3$$ through quaternion multiplication to cause its rotation $$ \mathbb{H} \times \mathbb{R}^3 \rightarrow \mathbb{R}^3 : \mathbf{q} \cdot \mathbf{x} \cdot \mathbf{q}^*$$. 

### Tangent space and the Lie Algebra

Let $$\mathcal{X}(t)$$ be a point on the Lie manifold $$\mathcal{M}$$ then $$\dot{\mathcal{X}} = \frac{d\mathcal{X}}{dt}$$ belongs to its tangent space at $$\mathcal{X}$$ (or linearized at $$\mathcal{X}$$) denoted as $$T_\mathcal{X}\mathcal{M}$$. Since we note that the lie group has the same curvature throughout the manifold, the tangent space $$T_{\mathcal{X}}\mathcal{M}$$ also has the same structure everywhere. 

The *Lie Algebra* then is simply the tangent space of a Lie group -- linearized -- at the identity element $$\mathcal{E}$$ of the group. Every Lie group $$\mathcal{M}$$ has an associated lie algebra $$\mathfrak{m}$$. The Lie algebra $$\mathfrak{m}$$ is a vector space.

### $$\mathbf{Exp}$$ and $$\mathbf{Log}$$ map

$$\mathbf{exp}: \mathfrak{m} \rightarrow \mathcal{M}$$ map takes elements on the tangent vector space to the Lie Group space *exactly*. The $$\mathbf{log}: \mathcal{M} \rightarrow \mathfrak{m}$$ does the inverse. We can  

Because the *tangent space* is a vector space, it can also be expressed as a linear combination of basis elements -- called *generators* -- that constrain $$\mathbb{R}^m$$ to the tangent space $$\mathfrak{m}$$. Then we define two operators *hat* and *vee* that transfer elements between their Lie algebra and their vector space representation and vice-versa. 



$$\mathbf{SO(3)}$$ example


### $$\oplus$$, $$\ominus$$ and the Adjoint operators

## Navigation with ideal IMU measurements

As eluded to in the <a href="#{{Introduction}}">Introduction</a>, an ideal Inertial Measurement Unit (IMU) mainly contains two sensors *Gyroscope* and *Accelerometer* and sometimes optionally a *magnetometer*.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ideal_imu_navigation.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
  Figure 1: Figure shows a body with an attached IMU moving along a trajectory operating within two instants $i$ and $j$. $\mathbf{W}$ denotes the world frame. At time $i$ the IMU is rotating about the illustrated axis, and eventually reaches a different position and axis of rotation at an intermediate timestep $k$ and reaches the final configuration at time $j$.  
</div>

### Gyroscope and inertial orientation
 An ideal Gyroscope measures the angular velocity or the angular rate of motion $${}_b\omega_k$$ in the body frame $$b$$ at time instant $$k$$. 
 
 Let us consider the IMU in Figure 1. As it travels along the trajectory from $$i$$ to $$j$$, the axis of rotation changes continuously. If we make the piece-wise linear approximation and assume that the axis of rotation remains fixed between two timesteps, then for an angular velocity measurement at $$k$$ as $$\omega_k$$, angular change in rotation is $$\omega_k \Delta t_k^{k+1} \in \mathfrak{so}(3)$$. Subsequently $$\Delta\mathbf{R}_k^{k+1} = \text{Exp}(\omega_k \Delta t_k^{k+1})$$.

Now assuming that all $$\Delta t$$ are equal we have for the complete trajectory:

$$
\begin{align}
{}_W \mathbf{R}_j &= {}_W \mathbf{R}_i \text{Exp}(\omega_i \Delta t_i) \dots \text{Exp}(\omega_k \Delta t_k)\dots \text{Exp}(\omega_j \Delta t_j) \\ 
\implies {}_W \mathbf{R}_j &= {}_W \mathbf{R}_i \prod_{k=i}^{j}\text{Exp}(\omega_k \Delta t_k)
\end{align}
$$

### Accelerometer and inertial velocity and position

## Navigation with real IMU measurements

## Why do we care about pre-integration? 

## Bias estimation and IMU initialization
