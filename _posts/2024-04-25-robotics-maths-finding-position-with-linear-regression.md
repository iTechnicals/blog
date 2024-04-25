---
layout: post
title: "Robotics maths: finding position with linear regression"
date: 2024-04-24
usemathjax: true
---

<body markdown='1' style="text-align: justify;">

Late last year I found myself involved in writing the code for a robot to take part in the [Student Robotics](https://studentrobotics.org/) competition, in particular for part of it that was taking place entirely within the [Webots](https://cyberbotics.com/) simulator. It turns out that, as well as an extended exercise in software engineering, it was also an opportunity for some extremely interesting maths, at least some of which I can't find easily anywhere on the Internet.

![Student Robotics 2024 arena schematic](/blog/assets/2024-04-25-robotics-maths-finding-position-with-linear-regression/arena.svg)

Above you can see a schematic of the arena for this year's competition, with 4 competing robots shown as green boxes, and a series of 'asteroids' in brown to be collected over the course of 150 seconds. However, these details aren't the most important thing, and in fact this article presents a solution to a far more general situation - determining the absolute position of an object when only the absolute positions of objects around it are known[^1].

![Starting view in the simulator](/blog/assets/2024-04-25-robotics-maths-finding-position-with-linear-regression/arena-view.png)

My knowledge on the topic of robot design is negligible, but I would imagine a fairly common situation to be in is to have some sort of camera, distance sensor or similar providing information about the robot's surroundings - in the case of the competition, the cubes and walls are plastered with markers, specifically AprilTags, readable by the robot, which are used for informing behaviour (the technical term for these sorts of things is a _fiducial marker_). In real life, they look like the below, but in the simulator, the robot is just fed data directly from the simulator, with a little bit of garbling applied to more accurately simulate real life. For example, the above image shows, in red boxes, all the markers recognised by the simulated robot in its starting position.

![Some examples of AprilTags from the 36h11 family](/blog/assets/2024-04-25-robotics-maths-finding-position-with-linear-regression/apriltags.png)

Given this information, one of the design choices we might want to make is between an absolute coordinate system - relative to some arbitrary stationary point - and a relative coordinate system - relative to the robot, which is of course moving all the time. All the data we get from fiducial markers or similar is of course relative, and so it's certainly much easier and faster (computationally) to implement a relative system, as no extra data processing is required - and, the more data processing, the more potential for things to go wrong. On the other hand, an absolute system is far more powerful in terms of the capabilities it offers to the robot - even a simple task such as turning 90 degrees is non-trivial in a relative system, whereas in an absolute system it's easy: just turn until the angle we're facing is 90 degrees away from where it started. Thus, wherever possible, this unqualified individual believes that an absolute approach is to be preferred - and this brings us to the core issue.

In the robotics competition, we have the relative positions of objects around us, thanks to the marker detection. We also have, in the cases of the wall markers, a good idea of their absolute position, thanks to the specifications of the arena. But how, given this information, can we find the absolute position of the robot?

![Diagram of the relative and absolute coordinate axes, with the relative and absolute axes parallel](/blog/assets/2024-04-25-robotics-maths-finding-position-with-linear-regression/coordinates.png)

Let's first consider a simplified case, in which the robot never turns, and its relative $x$ and $y$ axes are aligned with those of the absolute coordinate system of choice - this situation is shown above. Let's say that we have $n$ objects, where for the $i^\mathrm{th}$ object, we know the absolute position $(X_i, Y_i)$ and the relative position $(x_i, y_i)$ - also, we'll denote the absolute position of the robot, which we don't know, as $(X, Y)$. Now, in order to build towards the general case, we'll take an interesting approach - establish a transformation that takes absolute to relative coordinates, and use it to reverse engineer $X$ and $Y$.

In this case, it's not too bad. If we translate the absolute coordinate system to the left $X$ units and down $Y$ units, then the point $(X, Y)$ - the position of the robot - moves to $(0, 0)$. Since the relative and absolute axes are aligned and they have the same scale, steps (or more technically, the basis vectors) in the relative and absolute coordinate systems are the same. But now, our coordinates relative to the robot are relative to $(0, 0)$ - so the coordinates of the $i^\mathrm{th}$ object after the transformation are simply $(x_i, y_i)$, and we have established a transformation taking absolute coordinates to relative coordinates.

But what, mathematically, does it mean to translate left $X$ and down $Y$ units? Well, hopefully it's not too hard to see that this is the same thing as subtracting $X$ from the $x$-coordinate and $Y$ from the $y$-coordinate. Given this, we can extract a couple equations by considering the $x$ and $y$ coordinates:

$$
\begin{cases}
x_i = X_i - X \Rightarrow X = X_i - x_i \\
y_i = Y_i - Y \Rightarrow Y = Y_i - y_i
\end{cases}
$$

and so we can actually find values for $X$ and $Y$ with just one point. Now, we could do something like taking a robust mean between the values of $X$ and $Y$ obtained from each point, being careful to ignore outliers - in practice, the [median absolute deviation](https://en.wikipedia.org/wiki/Median_absolute_deviation), which is analogous to the standard deviation, can be helpful.

This is quite nice, but a robot that never turns is unorthodox and difficult to implement on the engineering side - and, if we can solve the generalised case where the robot does turn, there's no reason to attempt avoiding it. So let's try the same trick as before, finding a transformation that will take absolute to relative coordinates. First, though, we need to define a new attribute of the robot - $\theta$, the angle between the robot's $x$-axis and the absolute $x$-axis - we'll follow the standard convention of anti-clockwise = positive. Thus, our new situation is summarised by the diagram below:

![Diagram of the relative and absolute coordinate axes, with the relative axes rotated counterclockwise by an angle theta](/blog/assets/2024-04-25-robotics-maths-finding-position-with-linear-regression/coordinates-angled.png)

Now the transformation we want isn't actually that bad. As before, we translate left by $X$ and down by $Y$, but in this instance we have to follow up with a clockwise rotation by the angle $\theta$[^2]. If we like, we can express these in terms of vectors and matrices, using a vector for the translation and a matrix for the rotation:

$$
\begin{align*}
\begin{pmatrix}
x_i \\ y_i
\end{pmatrix} &= 
\begin{pmatrix}
\cos{\theta} & \sin{\theta} \\
-\sin{\theta} & \cos{\theta}
\end{pmatrix} \left[
\begin{pmatrix}
X_i \\ Y_i
\end{pmatrix} -
\begin{pmatrix}
X \\ Y
\end{pmatrix}
\right] \\ &=
\begin{pmatrix}
\cos{\theta} & \sin{\theta} \\
-\sin{\theta} & \cos{\theta}
\end{pmatrix}
\begin{pmatrix}
X_i - X \\ Y_i - Y
\end{pmatrix} \\ &=
\begin{pmatrix}
(X_i - X)\cos{\theta} + (Y_i - Y)\sin{\theta} \\
(Y_i - Y)\cos{\theta} - (X_i - X)\sin{\theta} 
\end{pmatrix}
\end{align*} \\ \Rightarrow
\begin{cases}
x_i = (X_i - X)\cos{\theta} + (Y_i - Y)\sin{\theta} \\
y_i = (Y_i - Y)\cos{\theta} - (X_i - X)\sin{\theta} 
\end{cases}
$$

Now this is good to have, but it's a lot trickier to deal with than our earlier formulae - mostly because there are now 3 variables, as we would have expected when we introduced $\theta$, and so we can't solve directly for $X$ and $Y$ (remember that $x_i$, $y_i$, $X_i$ and $Y_i$ are essentially constants here). We could _maybe_ try taking sets of 3 equations across multiple points, and then we would be able to solve for $X$, $Y$ and $\theta$, but there's actually a piece of mathematical machinery that is perfect for this situation, and is far less ugly and difficult to work with than solving directly would be - namely, the technique of linear regression, which I aimed to introduce in a beginner-friendly way in a [previous post](https://itechnicals.github.io/blog/2024/04/02/linear-regression-from-scratch.html).

The first step of linear regression is to produce a well-defined cost function, and the first step of that step is to introduce an assumed error (technically a residual), reflecting the facts that our measurements are subject to random error and so will not be perfect. So instead of $x_i$ and $y_i$ being literally equal to the formulae above, we say that they are equal to the formulae plus a residual which we'll call $\varepsilon_{x_i}$ and $\varepsilon_{y_i}$, and the first step is to rearrange for these residuals:

$$
\begin{cases}
\varepsilon_{x_i} = (X_i - X)\cos{\theta} + (Y_i - Y)\sin{\theta} - x_i \\
\varepsilon_{y_i} = (Y_i - Y)\cos{\theta} - (X_i - X)\sin{\theta} - y_i
\end{cases}
$$

and now, to get an overall residual for the data point, we square these and then sum them - the squaring being in order to make the numbers nice, ensure that positive and negative residuals do not cancel, and ostensibly to increase the effect of very large errors.

Now, as you might expect, the algebra at this point gets a bit out of hand, and there's no real value to be gained by number-crunching - sadly, in fact, I don't really have any problem-specific intuition to share for the process that follows. Nonetheless, for completeness, I will show the full calculations as a footnote to this article. Anyways, once we've found an expression for a particular point's residual, we sum these over all residuals to find a cost function - a function that, for given values of $X$, $Y$ and $\theta$, tells us exactly how badly they reflect the observations. Now we take the partial derivatives with respect to each variable, and this yields:

$$
\begin{cases}
\displaystyle{\frac{\partial}{\partial X}} &= 2nX - 2\sum X_i + 2\sum x_i\cos\theta - 2\sum y_i\sin\theta \\
\displaystyle{\frac{\partial}{\partial Y}} &= 2nY - 2\sum Y_i + 2\sum x_i\sin\theta + 2\sum y_i\cos\theta \\
\displaystyle{\frac{\partial}{\partial \theta}} &= \cos\theta (2Y\sum x_i + 2\sum y_i\sum X_i - 2\sum x_i\sum Y_i - 2X\sum y_i) + \\&\quad \sin\theta (2\sum x_i\sum X_i - 2X\sum x_i + 2\sum y_i\sum Y_i - 2Y\sum y_i) \\
\end{cases}
$$

If we were feeling pessimistic about our chances, we could implement gradient descent using these cost functions, and solve the problem by throwing a good old bit of CPU time at it. However, if we can, it'd be far, far preferable to get a closed form solution - and it shouldn't be _too_ hard, since we're dealing with trigonometric functions, which have a tendency to surprise us by doing nice things (spoiler alert). So let's give it a shot, and we'll just go for a simple substitution approach - right now, we aren't even sure how we're going to deal with $\sin\theta$ and $\cos\theta$ being present. Setting all the derivatives to 0 as per usual, we rearrange the first two resulting equations to get

$$
\begin{cases}
X = \frac{1}{n}\left(\sum X_i - \cos\theta\sum x_i + \sin\theta\sum y_i\right) \\
Y = \frac{1}{n}\left(\sum Y_i - \sin\theta\sum x_i - \cos\theta\sum y_i\right)
\end{cases}
$$

and we can sub these into the final equation to get something very long, but that has the potential to be very nice. Indeed, after extensive algebraic manipulation and many cancellations thanks to the Pythagorean identity, we arrive at:

$$
\begin{align*}
&\cos\theta\left(\sum x_i\sum Y_i + n\sum y_iX_i - n\sum x_iY_i - \sum y_i\sum X_i\right) = \\ &\sin\theta\left(\sum y_i\sum Y_i + \sum x_i\sum X_i - n\sum x_iX_i - n\sum y_iY_i\right)
\end{align*}
$$

and this is really just incredible news. Now we can rearrange our result to get an exact formula for $\tan\theta$, and thus, by taking the inverse tangent, we can get an exact formula for $\theta$:

$$
\boxed{\theta = \tan^{-1}\left(\frac{n\sum y_iX_i - n\sum x_iY_i - \sum y_i\sum X_i + \sum x_i\sum Y_i}{\sum y_i\sum Y_i + \sum x_i\sum X_i - n\sum x_iX_i - n\sum y_iY_i}\right)}
$$

and this is just pretty incredible. It's a real gift from the maths gods that this all works out so nicely, particularly that we end up with no terms at the end without a coefficient of either $\sin\theta$ or $\cos\theta$, so we can get a closed formula for tangent as opposed to something very ugly. Even better, while I don't intend to prove it mathematically, using `atan2` for this inverse tangent, which is a special version of the tangent function that can differentiate between angles in the range from $0$ to $2\pi$, despite tangent having period $\pi$, gives exactly the values we would expect. And it goes without saying that, once we have $\theta$, it's all but trivial to find $X$ and $Y$ using the very formulae we derived earlier.

What's more, we can make this all lightning fast thanks to numpy's blistering array operations, and, while outlier detection isn't quite so simple as just applying a robust mean, it's still definitely possible - I know because we've done it. I won't actually post the code here, for the general implementation or the outlier detection, because my team might not want me leaking our robot's source code on the Internet.

This is intended to be the first of two or three posts covering the maths and computer science that goes into coding a robot. One of the interesting things about robotics is that we are often faced with a huge quantity of data, which it seems wasteful not to use in its entirety, but which it can be hard to see a use for. We've already partially solved this problem here by including every stationary point we can when finding the robot's position, but there are still questions as time changes. If we simply apply this formula every timestep, then we are only using the current iteration's data, and while this seems sensible, since it is most up to date, we might nonetheless yearn for the smoothing or robustness that could be provided by some method that does make use of previous iterations' data. Maths is charitable enough to provide us with yet another tool for this very purpose - the Kalman filter. I might not write a blog post on this topic, since (a) I am really underqualified to talk about it and (b) there is already an incredible resource, the [kalmanfilter.net website](https://www.kalmanfilter.net/), which goes into great depth - so make sure to check it out if you're interested.

### Full derivation

![A full derivation of the formulae presented above](/blog/assets/2024-04-25-robotics-maths-finding-position-with-linear-regression/poscalc.png)

[^1]: Of course, in many situations, we have something like GPS that can tell us where we are directly, and so this technique is mostly moot. Nonetheless, it's an interesting exploration.

[^2]: It might be confusing that we perform a clockwise rotation here, and if you look more generally at the effect that these transformations have on the robot, you'll see that they move the robot's relative $x$ and $y$ axes to the absolute ones, instead of vice versa - yet we claim that we are finding the relative coordinates in terms of the absolute ones. But, since the coordinate system we're really working in is the absolute one, if we move the relative system to be absolute and read off the absolute coordinates of the resulting points, they will be the same as the coordinates that were originally relative. Hopefully that makes sense.

</body>