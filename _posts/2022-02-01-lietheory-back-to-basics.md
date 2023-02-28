---
layout: distill
title: Lie Theory Back to Basics
description: This blog post introduces Lie Groups and defines important concepts that are needed for state estimation or SLAM. Largely adapted from "A Micro lie-theory for state estimation in Robotics" <d-cite key=""></d-cite>
date: 2022-02-01

authors:
  - name: Akash Sharma
    url: "/"
    affiliations:
      name: RI, Carnegie Mellon

bibliography: 2022-02-01-lietheory.bib

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: Preliminaries
    subsection:
        - name: Manifold
        - name: Group
        - name: Lie Group

  - name: What is a Group?
    subsections:
        - name: Gaussian Random Variables
        - name: Conditional Independence
        - name: Filtering vs Smoothing
  - name: MAP estimation over Factor Graphs
    subsections:
        - name: Why Factor Graphs?
        - name: Nonlinear least squares formulation
        - name: Methods to solve linear systems (Matrix Factorization)
        - name: Gauss Newton and Levenberg Marquardt
  - name: The Bayes Tree (iSAM2)
    # if a section has subsections, you can add them as follows:
    # subsections:
    #   - name: Example Child Subsection 1
    #   - name: Example Child Subsection 2
  - name: Footnotes
  - name: References

# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }

---

## Preliminaries

### Manifold

A smooth manifold is a curved and differentiable hyper-surface -- one with no spikes / edges -- that locally resembles a linear space $$ \mathbb{R}^n $$ embedded in a higher dimensional space.

#### Examples:

* 3D Vectors $$ \in \mathbb{R}^3 $$ with unit-norm constraint form the spherical manifold $$\mathbb{S}^3$$
* Rotations in the 2D plane $$ \in \mathbb{R}^2 $$ form a circular manifold
* 3D Rotations form a hypersphere (3-sphere) in $$ \mathbb{R}^4 $$


<div class="row">
    <div class="col-sm mt-3 mt-md-0" align=center>
        {% include figure.html path="assets/img/manifold.png" title="Illustration of a manifold and tangent space" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
Figure 1: A manifold $$\mathcal{M}$$ and its associated tangent space $$ \mathcal{T}_\times \mathcal{M} (\mathbb{R}^2) $$ tangent at the point $$ \mathcal{X} $$ and a side view. The velocity element $$ \dot{\mathcal{X}} = \frac{\partial{X}}{\partial{t}} $$ does not belong to the manifold but to the tangent space.
</div>


### Group

A Group $$ \mathcal{G} $$ is a set with a composition operator $$ \circ $$, that for elements $$X, Y, Z \in G$$ satisfy the following axioms:

1. Closure under $$\circ$$: $$X \circ Y \in \mathcal{G}$$
2. Identity $$E$$: $$ E \circ X = X \circ E = X \in \mathcal{G}$$
3. Inverse $$X^{-1}$$: $$ X^{-1} \circ X = X \circ X^{-1} = E \in \mathcal{G}$$
4. Associativity: $$(X \circ Y) \circ Z = X \circ (Y \circ Z) $$

Notice the omission of commutativity

### Lie Group

A smooth manifold that **looks identical** (example: A sphere, a circle) at every point and satisfies all group properties:
* Composition of Lie group elements remain in the lie group
* Each lie group has an identity element
* Every element has an inverse.

#### Examples

1. The unit complex number group $$ \mathbb{S}^1: z = \cos(\theta) + i \sin(\theta) = e^{i \theta} $$ under complex multiplication as the composition operator is a Lie group. A Lie group element $$ z \in \mathbb{S}^1$$ rotates a regular complex vector $$ \mathbf{x} = x + iy $$ via complex multiplication to $$ \mathbf{x}^\prime = \mathbf{z} \circ \mathbf{x} $$. The unit norm constraint of the unit complex group defines a 1-sphere in 2D space.


<div class="row">
    <div class="col-sm mt-3 mt-md-0" align=center>
        {% include figure.html path="assets/img/unit-complex-group.png" title="Illustration of a manifold and tangent space" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
Figure 1: A manifold $$\mathcal{M}$$ and its associated tangent space $$ \mathcal{T}_\times \mathcal{M} (\mathbb{R}^2) $$ tangent at the point $$ \mathcal{X} $$ and a side view. The velocity element $$ \dot{\mathcal{X}} = \frac{\partial{X}}{\partial{t}} $$ does not belong to the manifold but to the tangent space.
</div>
