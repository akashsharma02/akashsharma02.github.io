---
layout: distill
title: A gentle introduction to Graph based SLAM
description: This blog post introduces Simultaneous Localization and Mapping from first principles. It covers necessary topics such as Unimodal Gaussian distributions, Factor Graphs, Matrix Factorization techniques, and the Bayes Tree
date: 2022-03-14

authors:
  - name: Akash Sharma
    url: "/"
    affiliations:
      name: RI, Carnegie Mellon

bibliography: 2022-03-14-graph-slam.bib

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: Introduction
  - name: Preliminaries
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

## Introduction

The problem of Simultaneous Localization and Mapping (SLAM) has a rich history in robotics dating back to 1986 <d-cite key="1638022"></d-cite>. Briefly, it is the problem of localizing -- locating -- the robot in an *unknown* map of the environment, a fundamental problem for any mobile robot. Since the location of the robot is specified against an unknown map, the localization task takes the form of estimating a piece-wise linear trajectory for the robot, as it moves around in the environment from its arbitrary origin. The map may be represented as a set of distinct landmarks (called landmark-based SLAM) or as a dense representation of the environment such as a voxel map (called dense SLAM).

Most recent literature (post 2000s) partition the problem into the frontend and backend.
 - The front end of any SLAM system that tackles state estimation, deals with processing and associating the raw sensor measurements. This results in a set of measurements associated with each to-be-estimated variable -- typically the robot and landmark poses.
 - The back end is a *sensor agnostic* optimization framework that takes in the measurement(s) and produces the most likely estimate for each variable according to some notion of distance from the actual (unobserved) variable values.

When all the variables to be estimated are Special Euclidean $$ \text{SE}(n) $$ poses, and the sensors generate relative pose measurements, the optimization problem is called Pose Graph Optimization (PGO) <d-cite key="carlone2015initialization"></d-cite> or Synchronization over the Special Euclidean Group <d-cite key="rosen2019se"></d-cite>. If all the variables are Special orthogonal $$ \text{SO}(n) $$ rotations, then the problem is also called rotation averaging <d-cite key="hartley2013rotation"></d-cite>.

This article addresses the backend optimization and motivates the problem formulation from first principles.

## Preliminaries

### Filtering vs Smoothing

Most methods prior to 2000s were predominantly filtering methods. In fact, most courses [[1](/teaching/)] [[2](http://www.ipb.uni-bonn.de/robot-mapping/)] that deal with state estimation also start the course with filtering techniques.

When a robot moves around in an environment, it's state space (the variables it estimates) grows, i.e., for every time-step, a new pose variable needs to be estimated. Loosely speaking, filtering methods only update the variables at the current time step, while smoothing methods *may* update all the variables, past and current. Chapter 3 and 4 of Probabilistic Robotics <d-cite key="thrun2002probabilistic"></d-cite> is a good treatment of Filtering methods relevant to the problem of SLAM.

Here, I address Smoothing techniques.

### Gaussian Random Variables

A random variable is an outcome of a random event, and has an associated probability distribution. In our context, the act of measurement is the random event, which produces an output that is a random variable. A classic instance that is cited in many books <d-cite key="thrun2002probabilistic"></d-cite> is the example of an odometry measurement. Perhaps due to wheel slip or inconsistent hall-effect, or some other physical property, the sensor measurement may be noisy (random). In most cases, we expect the sensor to reproduce measurements faithful to its underlying state, but many a time, it may not. This property may be mathematically modeled by a Gaussian random variable.

$$
\begin{align}
\mathbf{z} &= h(\mathbf{x}) + \nu \\
\nu &\sim \mathcal{N}(0, \Sigma)
\end{align}
$$

Here, equation (1) means that the state $$ \mathbf{x} $$ is transformed by a measurement function $$ h $$ which is corrupted by inherent additive noise $$ \nu $$ in the sensor to produce a measurement $$\mathbf{z}$$. We model this noise $$\nu$$ as a random variable sampled from a Gaussian probability distribution. If $$ h(\mathbf{x}) $$ were the *true* underlying measurement, then if we sampled from the sensor, we get noise added to the measurement and probability distribution of the noise as follows

$$
\begin{align}
\nu &\sim \mathbf{z} - h(\mathbf{x}) \\
p(\nu) &= \frac{1}{\sqrt{2 \pi \Sigma}} \exp( \| \nu \|^2_{\Sigma} )
\end{align}
$$


<div class="row">
    <div class="col-sm mt-3 mt-md-0" align=center>
        {% include figure.html path="assets/img/normal-distribution-pdf.svg" title="Gaussian probability distribution" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
Figure 1: The Normal distribution or the Gaussian probability distribution illustrated with different means ($ \mu $) and variances ($ \sigma^2 $). Image taken ingloriously from Wikipedia.
</div>

The choice of the probability distribution is rather curious. The Gaussian distribution is part of the exponential family of distributions, and has some convenient *algebraic* properties:
1. It is fully described by its *sufficient* statistic. For instance, even though the Gaussian distribution has probability mass almost everywhere in its domain (infinite support), the distribution is fully described by its mean and covariance.
2. The product of Gaussian distributions results in another Gaussian distribution i.e., the conjugate prior of a Gaussian distribution is another Gaussian.

### Conditional Independence of Random variables

In Probability theory, two random variables $$\mathbf{x}$$ and $$\mathbf{y}$$ are said to be independent, if their joint distribution equals the product of their probabilities. Intuitively, it means that if the value of the random variable $$\mathbf{x}$$ is observed, then the probability of $$\mathbf{y}$$ is unaffected:

$$
\begin{align}
p(\mathbf{x}, \mathbf{y}) = p(\mathbf{x}) p(\mathbf{y})
\end{align}
$$

Similarly, if we consider three random variables $$\mathbf{x}$$, $$\mathbf{y}$$ and $$\mathbf{z}$$, then they are said to be conditionally independent, if
$$
\begin{align}
p(\mathbf{x}, \mathbf{y} | \mathbf{z}) = p(\mathbf{x} | \mathbf{z}) p(\mathbf{y} | \mathbf{z})
\end{align}
$$

Conditional independence in the context of SLAM is *hopefully* illustrated with the following example.

#### Example 1:

Consider a robot in a one-dimensional world as in Figure 2 (a).
At time $$ t_0 $$, if we know the location $$ \mathbf{x}_0 $$ and $$\mathbf{x}_1$$, then the robot samples from the odometry sensor a measurement $$\mathbf{z}_0$$ of (say 50.3 m) as follows

$$
\begin{align}
\mathbf{z}_0 \sim h_{\text{odometry}}(\mathbf{x}_0, \mathbf{x}_1) + \nu_{\text{odometry}}
\end{align}
$$

Similarly, at time $$ t=1 $$, if we know that the robot is at 50 meters from origin, and then it samples from the range sensor two measurements $$ \mathbf{z}_1 $$ and $$ \mathbf{z}_2 $$ as follows:

$$
\begin{align}
\mathbf{z}_1 \sim h_{\text{range}}(\mathbf{x}_1, \mathbf{l}_0) + \nu_{\text{range}}\\
\mathbf{z}_2 \sim h_{\text{range}}(\mathbf{x}_1, \mathbf{l}_1) + \nu_{\text{range}}
\end{align}
$$

<div class="row">
    <div class="col-sm mt-3 mt-md-0" align=center>
        {% include figure.html path="assets/img/robot-1d-illustration.png" title="Robot in a 1D world" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
Figure 2: Illustration of a 1-D robot that observes two landmarks after moving forward by 50 meters at timestep 1.
</div>
Figure 3(a) is a graphical representation of the above robot, where the state variables are denoted with large circles annotated with variable names, and measurements are denoted with edges with a filled dot.

Next, consider the selected portion -- in dotted lines -- of Figure 3(a) for simplicity. Here we see that, to sample measurements $\mathbf{z}_0$ and $\mathbf{z}_1$, we need to sample from the joint probability distribution i.e., $$ p(\mathbf{z}_0, \mathbf{z}_1, \mathbf{x}_0, \mathbf{x}_1, \mathbf{l}_0) $$.

$$
p(\mathbf{z}_0, \mathbf{z}_1, \mathbf{x}_0, \mathbf{x}_1, \mathbf{l}_0) = p(\mathbf{z}_0, \mathbf{z}_1 | \mathbf{x}_0, \mathbf{x}_1, \mathbf{l}_0) p(\mathbf{x}_0, \mathbf{x}_1, \mathbf{l}_0) \\
$$

Now consider the conditional distribution over the two measurements $$ \mathbf{z}_0, \mathbf{z}_1 $$. From Equation (6) and (7) we know $$ \mathbf{z}_0 $$ only depends on $$ \mathbf{x}_0 $$ and $$ \mathbf{x_1} $$, and from Equation (8) a similar argument can be made for $$ \mathbf{z_1} $$. Therefore, we can safely *assume* the following:

$$
\begin{align*}
p(\mathbf{z}_0, \mathbf{z}_1 | \mathbf{x}_0, \mathbf{x}_1, \mathbf{l}_0) &= p(\mathbf{z}_0 |  \mathbf{x}_0, \mathbf{x}_1, \mathbf{l}_0) p(\mathbf{z}_1 |  \mathbf{x}_0, \mathbf{x}_1, \mathbf{l}_0)~~~~\text{conditionally independent}\\
p(\mathbf{z}_0, \mathbf{z}_1 | \mathbf{x}_0, \mathbf{x}_1, \mathbf{l}_0) &= p(\mathbf{z}_0 |  \mathbf{x}_0, \mathbf{x}_1) p(\mathbf{z}_1 | \mathbf{x}_1, \mathbf{l}_0)~~~~~~~~~~~~~~~\text{conditioned on redundant variables}
\end{align*}
$$

The above assumption of conditional independence between measurements given state variables is key in reducing the complexity of estimation.

### Why Factor Graphs?

Factor graphs are bipartite graphs -- graphs containing two types of random variables -- that describe the factorization of a probability distribution. Factorizing a joint probability distribution amounts to finding the subsets of random variables that are conditionally independent of each other and therefore be written as a product.

For example 1, Figure 3(a) concisely represented the factorization of the joint probability distribution between all the random variables.

<div class="row">
    <div class="col-sm mt-3 mt-md-0" align=center>
        {% include figure.html path="assets/img/bipartite-graph.png" title="Bipartite graph" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
Figure 3: (a) Factor Graph representation for Example 1.  (b) Equivalent factor graph illustrating the bipartite nature of the graph
</div>

