---
usemathjax: true
layout: post
title: "The matrix logarithm of an orthogonal matrix"
subtitle: "A love letter"
date: 2021-04-27
background: '/img/bg-index.png'
---

This post is a tribute to the fact that the matrix logarithm of a $3\times 3$ rotational matrix is

$$
\log{R} = \frac{\theta}{2\sin{\theta}}(R^T - R) \label{eq:logm} \tag{1}
$$

where $\theta$ satisfies $1 + 2\cos(\theta) = \text{trace}(R)$.

I think this is beautiful and surprising: The matrix logarithm is proportional to the simplest skew-symmetric matrix one can create from $R$: $R^T - R$.

We will argue that a matrix logarithm of a rotational matrix will output a skew-symmetric matrix, but that a priori does not have to mean the relationship is as simple as it is!

A more thorough treatment of this topic can be found in this [paper](https://www.sciencedirect.com/science/article/pii/S0377042709007791).

# Matrix exponentials

Matrix exponentials are neat. They appear in the (analytical) solutions to linear systems of ordinary differential equations (ODEs):

$$
\mathbf{\dot{x}} = A\mathbf{x} \quad \Rightarrow \quad \mathbf{x}(t) = e^{At}\mathbf{x}_0 \label{eq:ode} \tag{2}
$$

in which

$$
e^A = 1 + A + \frac{1}{2} A^2 + \frac{1}{3!}A^3 + \ldots + \frac{1}{n!}A^n + \ldots
$$

There is a famous paper by MATLAB founder Cleve Moler on how there are many ways to compute it, but all of them are [dubious](https://www.cs.cornell.edu/cv/ResearchPDF/19ways+.pdf). Luckily, we will only be dealing with a specific type of small systems in this post: $3 \times 3$ skew-symmetric systems. A skew-symmetric system is one that satisfies

$$ 
S^T = - S
$$

It follows rather directly that 

$$(e^S)^Te^S = e^{S^T}e^S = e^{-S}e^S = e^{-S + S} = e^0 = I$$


The matrix exponential of a skew-symmetric matrix is orthogonal. And conversely the matrix logarithm of an orthogonal matrix will be skew-symmetric.

We have not actually used the $3\times 3$ size of the matrices yet, but we'll need that to also understand the $\theta$ which appears in equation \eqref{eq:logm}

We will take a detour through some application: 3D curves

# 3D curves
For sufficiently smooth curves in 3D, one can define an orthogonal coordinate frame at every point along the curve consisting of the 3 vectors:
- $T$: the tangent
- $N$: the normal
- $B$: the binormal

A picture says a thousand words:

<img src="https://upload.wikimedia.org/wikipedia/commons/1/11/Frenet.svg" alt="Frenet-Serret frame" width="300"/>

This frame is known as the [Frenet-Serret frame](https://en.wikipedia.org/wiki/Frenet%E2%80%93Serret_formulas). Associated are the Frenet-Serret equations which establish the relationship between the change of these frames when one moves along the curve, and the curvature $\kappa$ and torsion $\tau$ of the 3D curve:

$$
\begin{bmatrix} \mathbf{T'} \\ \mathbf{N'} \\ \mathbf{B'} \end{bmatrix} = \begin{bmatrix}
 0 & \kappa & 0 \\
 -\kappa & 0 & \tau \\
 0 & -\tau & 0
\end{bmatrix}
\begin{bmatrix} \mathbf{T} \\ \mathbf{N} \\ \mathbf{B} \end{bmatrix}
$$

Et voila, we have our skew-symmetric matrix! Solving these differential equations is generally not as simple as using equation \eqref{eq:ode}, as $\kappa$ and $\tau$ are usually functions of time/arc length. But matrix exponentials can be useful when designing dedicated integrators for these types of systems, which will be a topic for a future blog post.

Note that these are not general skew-symmetric matrices, as the $(1, 3)$ and $(3, 1)$ entries are zero. Parallel transport frames and [Darboux frames](https://en.wikipedia.org/wiki/Darboux_frame) are alternative frames for which these entries might not be zero.

Why would one be interested in using all of these equations? Well, the applications are plenty: rollercoasters are real life examples of freeform curves, flight paths of planes and drones as well, [freeform metal tubes](/2023/03/25/dynobend_tube_optimizer.html) are used in many design applications, etc... 

# 3D rotation formalisms

Wikipedia has a very extensive page on [rotation formalisms in 3D](https://en.wikipedia.org/wiki/Rotation_formalisms_in_three_dimensions), which lists are relates formalisms like
- Euler angles
- Rotation matrices
- Quaternions
- Axis-angle representations

Matrix exponentials are abundant. Even in 2D this is actually the case when we consider the relationship between angles, complex numbers and exponentials through:

$$
e^{i\theta} = \cos(\theta) + i \sin(\theta)
$$

In 3D, the key connection between the matrix exponentials and logarithms and the rotation formalisms is that the skew-symmetric matrix is very closely related to the axis-angle representation. A $3\times 3$ skew-symmetric matrix only has 3 'degrees-of-freedom' $\mathbf{u} = (u_1, u_2, u_3)$:

$$
S_u = \begin{bmatrix}
 0 & -u_3 & u_2 \\
 u_3 & 0 & -u_1 \\
 -u_2 & u_1 & 0
\end{bmatrix}
$$

The 'axis' in axis-angle is only a direction, so a normalized $\hat{\mathbf{u}} = \frac{\mathbf{u}}{\|\mathbf{u}\|}$. The angle $\theta$ from equation \eqref{eq:logm} is precisely the length of $\mathbf{u}$:

$$
\theta = \|\mathbf{u}\|
$$

Finally, we can remark the relationship between the cross product and matrix vector multiplication with $S$

# Scipy experiments


```python
import numpy as np
import scipy.linalg
```

## Matrix exponential


```python
S = np.random.randn(3, 3)
S = S.T - S # Random skew-symmetric matrix
print(S)
```

    [[ 0.         -0.06587121  0.39412242]
     [ 0.06587121  0.          3.5626287 ]
     [-0.39412242 -3.5626287   0.        ]]
    


```python
R = scipy.linalg.expm(S)
print(R)
print(R.T @ R)
```

    [[ 0.97635354 -0.20005875 -0.08191619]
     [-0.21582354 -0.88030499 -0.42247286]
     [ 0.01240816  0.43016232 -0.90266628]]
    [[ 1.00000000e+00 -1.29061949e-17 -8.79726730e-17]
     [-1.29061949e-17  1.00000000e+00 -4.21722237e-16]
     [-8.79726730e-17 -4.21722237e-16  1.00000000e+00]]
    


```python
theta = np.sqrt(S[0, 1]**2 + S[0, 2]**2 + S[1, 2]**2)
C = S/theta
C2 = C @ C
R2 = np.eye(3) + np.sin(theta)*C +  (1 - np.cos(theta))*C2
print(R2)
```

    [[ 0.97635354 -0.20005875 -0.08191619]
     [-0.21582354 -0.88030499 -0.42247286]
     [ 0.01240816  0.43016232 -0.90266628]]
    

## Matrix logarithm


```python
R = np.random.randn(3, 3)
R, _ = np.linalg.qr(R)
```


```python
print(R)
print(R.T @ R)
```

    [[-0.81111708  0.55235245 -0.19234307]
     [-0.25976631 -0.63485003 -0.72765851]
     [-0.52403296 -0.54025199  0.65842026]]
    [[ 1.00000000e+00  9.10029322e-18 -6.74003476e-18]
     [ 9.10029322e-18  1.00000000e+00  1.55457201e-16]
     [-6.74003476e-18  1.55457201e-16  1.00000000e+00]]
    


```python
scipy.linalg.logm(R) / (R.T - R)
```


    array([[        inf, -2.98369391, -2.98369391],
           [-2.98369391,        -inf, -2.98369391],
           [-2.98369391, -2.98369391,         inf]])

-----

## Related posts
- [Dynobend Tube Optimizer](/2023/03/25/dynobend_tube_optimizer.html)

## Want to leave a comment?
Very interested in your comments but still figuring out the most suited approach to this. For now, feel free to send me an [email](mailto:lenseswaenen@gmail.com)
