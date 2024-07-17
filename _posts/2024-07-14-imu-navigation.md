---
layout: distill
title: almost everything about IMU navigation 
description: strapdown inertial navigation basics with an explanation of IMU pre-integration 
tags: distill research 
giscus_comments: true
date: 2024-07-14
featured: true

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
  - name: Lightning Review of Lie Algebra
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

1. **Gyroscopes** which measure the angular velocity $${}_b\omega_k$$ in the body frame $$b$$ at time instant $$k$$ 
2. **Accelerometers** which measure the sum of linear acceleration $${}_b\mathbf{a}_k$$ acting on the body frame at time instant $$k$$ and the acceleration due to gravity on the body $$ -(\mathbf{R}_k^W)^{\top} {}_w \mathbf{g}$$ therefore, the measurement made by the accelerometer is $${}_b\mathbf{a}_k -(\mathbf{R}_k^W)^{\top} {}_w \mathbf{g}$$

In this blog, I cover the basics required to understand inertial navigation in robot state estimation, and also explain the motivation behind IMU pre-integration<d-cite key="forster2016manifold"></d-cite>, the defacto way to deal with inertial measurements in visual-inertial SLAM. 

## ~~Lightning~~ Review of Lie Algebra

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

Let $$\mathcal{X}(t)$$ be a point on the Lie manifold $$\mathcal{M}$$ then $$\dot{\mathcal{X}} = \frac{d\mathcal{X}}{dt}$$ belongs to its tangent space at $$\mathcal{X}$$

$$\mathbf{SO(2)}$$ algebra example

### $$\mathbf{Exp}$$ and $$\mathbf{Log}$$ map

### $$\oplus$$, $$\ominus$$ and the Adjoint operators

## Navigation with ideal IMU measurements

## Navigation with real IMU measurements

## Why do we care about pre-integration? 

## Bias estimation and IMU initialization

---