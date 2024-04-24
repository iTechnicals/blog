---
layout: post
title: Linear regression from scratch
date: 2024-04-02
usemathjax: true
---

<body markdown='1' style="text-align: justify;">

In my corner of the world, it seems that statistics has a bad reputation. When even my statistics teacher says quote 'no-one wants to each statistics' and even just 'I hate statistics', something has clearly gone wrong. But in a way, this isn't too surprising. So much of statistics requires such difficult maths to actually derive that for the majority of people it's just an exercise in mindless number manipulation until the supposedly right answer emerges. The curious student who, for example, heads to the derivation section on the Wikipedia page for Pearson's chi-squared test, is rapidly annihilated by equations so complicated that they can't even fit on the page; if you want to understand what $\sqrt{\pi}$ is doing in the normal distribution, you either need [a really good 3blue1brown video](https://www.youtube.com/watch?v=cy8r7WSuT1I) or a foundation in multivariable calculus.

![Steps in the derivation of Pearson's chi-squared test](/blog/assets/2024-04-02-linear-regression-from-scratch/chi-squared-derivation.png)

All this is why it's nice when we find a part of statistics that doesn't require such prohibitively complicated tools to derive, and so can be appreciated anyone who would otherwise suffer in computational tedium. I'm talking about linear regression.

The general ordinary least squares equation is

$$\hat{\beta} = (\mathbf{X^TX})^{-1}\mathbf{X^Ty}$$

or a special case that more people might have seen is 

$$b = \frac{\mathrm{cov}(x, y)}{\mathrm{cov}(x, x)},\; a = \bar{y} - b\bar{x}$$

Of course, if you've never seen either of these equations, it doesn't really matter. Even if you've seen the first equation, it's so mathematically dense that you might not know what it means or understand why it shows up in this situation. The aim of this article is to build ordinary least squares regression, the most common and simple type of linear regression, from the ground up, introducing concepts from calculus and linear algebra as they become helpful.

Linear regression, put simply, is the process of finding a line of best fit, which, in the simplest case, is something that looks like

$$y = \beta_0 + \beta_1x$$

or in other words just $y=mx+c$ with some fancier symbols to help us out later on. You might know that if we have two points $(x_1, y_1)$ and $(x_2, y_2)$, we can always find a line passing through them, but if we have more than two - which we'll call $(x_i, y_i)$ where $i$ ranges from 1 to some number $n$ - we can't normally find such a line. Instead, the aim is to find a line which goes as close to as many of them as possible. In order to get some maths and later electrons to do this for us, we'll first have to formalise this statement by introducing the concept of the *residual*, to turn this question into a well-defined optimisation problem. The residual for the $i^\text{th}$ point is $\varepsilon_i = y_i - (\beta_0 + \beta_1x_i)$, the difference between the value of $y$ our model predicts for a given $x_i$, and the true value of $y$, $y_i$. Hopefully you'll agree this makes sense to want to minimize.

Now we don't just want to minimize the residual for any individual point (we could do this just by making the line go through that point) - we want to minimize it for all the points, at the same time. We might decide, then, to add them all together - but, if anyone's familiar with standard deviation and variance, you might know the problem with just adding all the residuals together - that negative and positive residuals will cancel one another out. To avoid this, we can square all the residuals before adding them together, and this will also give us some much nicer results later on than, say, taking the absolute value. So we now have a very clear mathematical question: what values for $\beta_0$ and $\beta_1$ minimize

$$\begin{align*}
\Sigma\varepsilon^2&=\sum_{i=1}^n \left(y_i - \beta_0 - \beta_1^2x_i\right)^2 \\
&= \sum_{i=1}^n y_i^2 + \beta_0^2 + \beta_1x_1^2 - 2\beta_0y_i - 2\beta_1x_iy_i + 2\beta_0\beta_1x_i \\
&= \sum_{i=1}^n y_i^2 + \beta_0^2\sum_{i=1}^n 1 + \beta_1^2\sum_{i=1}^n x_1^2 - 2\beta_0\sum_{i=1}^n y_i - 2\beta_1\sum_{i=1}^n x_iy_i + 2\beta_0\beta_1\sum_{i=1}^n x_i
\end{align*}$$

If you don't know what the $\Sigma$ means, check out [this Wikipedia page](https://en.wikipedia.org/wiki/Summation#Capital-sigma_notation). You might think it's a bit absurd to write the coefficient of $\beta_0^2$ as $\sum_{i=1}^n 1$, but this is just for consistency with all the other sums. To be completely clear, here $x_i$ and $y_i$ are not variables - rather, $\beta_0$ and $\beta_1$ are. All the sums are functions of our data, which become constant coefficients for the function in the $\beta$ values that we now wish to minimize. The audacious among you might suggest completing the square, but this is regrettably a situation where we do have to employ some more heavy-duty tools (the $\beta_0\beta_1$ terms preclude completing the square) - namely, partial derivatives.

A derivative is a tool from calculus that simply tells us the gradient of a function at any given point. For example, the derivative of $x^2$ is $2x$, so the gradient of the graph at some point $(x, x^2)$ is $2x$. This is useful because, for some nice, well-behaved graphs, the minimum or maximum point is simply located wherever the gradient is 0. Hopefully it's not too hard to see that this works for $y=x^2$, but might not work for, say $y=x^3$, where the gradient is $0$ at $(0, 0)$, but this is neither the minimum or the maximum - in fact, there is no minimum or maximum. Fortunately, because we've chosen to square our residuals, all our graphs are guaranteed to be nice in this way.

![Plot of y=x^2 with a tangent line and its gradient indicated.](/blog/assets/2024-04-02-linear-regression-from-scratch/example-plot-2d.png)

Of course, our 'graph' in this instance is one of $\Sigma\varepsilon^2$ against $\beta_0$ and $\beta_1$, so we need to go a bit further than just the normal derivative. Fortunately, it's still not too bad. If we find all the points for which the gradient in the $x$-direction is 0, that gives us a parabola within a paraboloid (3d parabola) - see the green curve below. If we then also find all the points for which the gradient in the $y$-direction is 0, we have another parabola, shown in blue below - and, thanks to our graph being a paraboloid, we are guaranteed that these intersect at precisely one point, which is the minimum of the entire surface. The gradient in the $x$-direction is referred to as the *partial derivative with respect to x* or $\frac{\partial}{\partial x}$, and similarly for the $y$-direction.

![Plot of ](/blog/assets/2024-04-02-linear-regression-from-scratch/example-plot-3d.png)

Thus, we have a set of two simultaneous equations, one specifying that $\frac{\partial}{\partial x} = 0$, and the other specifying that $\frac{\partial}{\partial y} = 0$, with one solution in $\beta_0$ and $\beta_1$. All that remains is to actually find $\frac{\partial}{\partial x}$ and $\frac{\partial}{\partial y}$, and we only need a few simple differentiation rules to do it:

$$
\begin{cases}
\frac{\partial}{\partial x} x^2 = 2x \\
\frac{\partial}{\partial x} x = 1 \\
\frac{\partial}{\partial x} a = 0,\text{where $a$ does not depend on $x$} \\
\frac{\partial}{\partial x} af(x) = af'(x),\text{where $a$ does not depend on $x$ and } f'(x) = \frac{\partial}{\partial x}f(x)  \\
\end{cases}
$$

I'm sorry to just state these facts without proof, but perhaps it can be an exercise to the reader, if you're familiar with the definition of the derivative or better yet the partial derivative, to prove these from first principles. Anyway, recalling that $\Sigma\varepsilon^2 = \sum_{i=1}^n y_i^2 + \beta_0^2\sum_{i=1}^n 1 + \beta_1\sum_{i=1}^n x_1^2 - 2\beta_0\sum_{i=1}^n y_i - 2\beta_1\sum_{i=1}^n x_iy_i + 2\beta_0\beta_1\sum_{i=1}^n x_i$ and applying our rules,

$$
\begin{align*}
\frac{\partial}{\partial \beta_0} &= 2\beta_0\sum_{i=1}^n 1 - 2\sum_{i=1}^n y_i + 2\beta_1\sum_{i=1}^n x_i &= 0 \\
\frac{\partial}{\partial \beta_1} &= 2\beta_1\sum_{i=1}^n x_1^2 - 2\sum_{i=1}^n x_iy_i + 2\beta_0\sum_{i=1}^n x_i &= 0
\end{align*}
$$

and if we rearrange and cut out the factor of 2[^1],

$$
\begin{align}
\beta_0\sum_{i=1}^n 1 + \beta_1\sum_{i=1}^n x_i &= \sum_{i=1}^n y_i \\
\beta_0\sum_{i=1}^n x_i + \beta_1\sum_{i=1}^n x_1^2 &= \sum_{i=1}^n x_iy_i
\end{align}
$$

Now it's time for a fun interlude on matrices. In our beloved equation $\hat{\beta} = (\mathbf{X^TX})^{-1}\mathbf{X^Ty}$, literally everything is a matrix (or a vector, which is just a special case of a matrix), and there's a very good reason for this: paper is expensive, and writing stuff with matrices saves a *lot* on paper[^2].

A matrix is a grid of numbers with a few cool features. In particular, they have an interesting definition of multiplication, which turns out to be useful in many contexts. This is a great maths moment, because it means that if we get a computer to do matrix operations, all of a sudden it can do loads of cool stuff (computer graphics, AI, and of course linear regression, to name just a few). I really hate to not do this topic justice, and I implore you in the strongest terms to check out [3b1b's video series on linear algebra](https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab) (link every 3blue1brown video in one blog post challenge), but for now all we really need to understand is that we can use matrices and specifically matrix inversion to solve simultaneous equations. Let's consider the matrix-vector multiplication

$$
\begin{pmatrix}
\displaystyle{\sum_{i=1}^n 1} & \displaystyle{\sum_{i=1}^n x_i} \\
\displaystyle{\sum_{i=1}^n x_i} & \displaystyle{\sum_{i=1}^n x_1^2} \\
\end{pmatrix}
\begin{pmatrix}
\beta_0 \\
\beta_1
\end{pmatrix}
$$

Now we will have to refer to the definition of matrix multiplication to evaluate this, and it can be summarised thus: to find the entry in row $i$ and column $j$ of the result matrix, multiply the $k^\text{th}$ element in row $i$ of matrix 1 by the $k^\text{th}$ element in column $j$ of matrix 2, and sum this over every $k$ within the bounds of the matrices (it's easier once you see a few examples). For example, we find that the product above is equal to 

$$
\begin{pmatrix}
\displaystyle{\beta_0\sum_{i=1}^n 1} + \displaystyle{\beta_1\sum_{i=1}^n x_i} \\
\displaystyle{\beta_0\sum_{i=1}^n x_i} + \displaystyle{\beta_1\sum_{i=1}^n x_1^2}
\end{pmatrix}
$$

But, from equations 1 and 2 above, the first element is equal to $\sum_{i=1}^n y_i$ (1) and the second $\sum_{i=1}^n x_iy_i$ (2), which means that this vector is the same as the RHSes of the two equations from earlier. Thus, we have our simultaneous equations in the form of a matrix equation:

$$
\begin{pmatrix}
\displaystyle{\sum_{i=1}^n 1} & \displaystyle{\sum_{i=1}^n x_i} \\
\displaystyle{\sum_{i=1}^n x_i} & \displaystyle{\sum_{i=1}^n x_1^2} \\
\end{pmatrix}
\begin{pmatrix}
\beta_0 \\
\beta_1
\end{pmatrix} = 
\begin{pmatrix}
\displaystyle{\sum_{i=1}^n y_i} \\
\displaystyle{\sum_{i=1}^n x_iy_i}
\end{pmatrix}
$$

Now in the interest of metaphorical paper-saving, let's write $\mathbf{M_X}$ for the first matrix and $\mathbf{M_{XY}}$ for the vector on the RHS (keeping in mind that a vector is just an $n$ by 1 matrix), and, excitingly, let's write $\hat{\beta}$ for the vector of $\beta_0$ and $\beta_1$, so we have $\mathbf{M_X}\hat{\beta} = \mathbf{M_{XY}}$. Now is the right time to introduce the concept of matrix inversion or, as we are strictly forbidden from calling it, division. Given a matrix $\mathbf{M}$, $\mathbf{M}^{-1}$ is the matrix such that $\mathbf{M}^{-1}\mathbf{M} = \mathbf{M}\mathbf{M}^{-1} = \mathbf{I}$, where $\mathbf{I}$, the matrix equivalent of 1, is the matrix such that $\mathbf{IM} = \mathbf{MI} = \mathbf{M}$ (for any matrix $\mathbf{M}$). Now if we *left multiply* both sides of our matrix equation (matrix multiplication is not always commutative, which means that left multiplying and right multiplying are not always the same thing), we have

$$
\begin{align*}
\mathbf{M_X}^{-1}\mathbf{M_X}\hat{\beta} &= \mathbf{M_X}^{-1}\mathbf{M_{XY}} \\
\Rightarrow \hat{\beta} &= \mathbf{M_X}^{-1}\mathbf{M_{XY}}
\end{align*}
$$

Let's just take a moment to appreciate where we've managed to get to. In order to find the minimum of a paraboloid, which represented how 'wrong' our values of $\beta_0$ and $\beta_1$ are, we took the partial derivatives, and set them to 0, which gave us some simultaneous equations - then, it became convenient to express these as a matrix equation to get a somewhat explicit solution. The awesome thing about this is that all the minimization and calculus side of things essentially only arose as an intermediate step - having done it once in the derivation, we now don't need to worry about it in the actual implementation.

Of course, we can go a bit further, and it turns out that $\mathbf{M_X}$ and $\mathbf{M_{XY}}$ can also be expressed in terms of a matrix multiplication between simpler matrices (I told you that it was versatile!). We'll introduce the matrix $\mathbf{X}$ and the vector $\mathbf{y}$, where

$$
\mathbf{X} =
\begin{pmatrix}
1 & x_1 \\
1 & x_2 \\
\vdots & \vdots \\
1 & x_n
\end{pmatrix},\;
\mathbf{y} =
\begin{pmatrix}
y_1 \\
y_2 \\
\vdots \\
y_n
\end{pmatrix}
$$

It might seem a bit strange to have a column in $\mathbf{X}$ which is just ones, but think about the first column as being what we multiply by $\beta_0$ and the second what we multiply by $\beta_1$. In fact, we could add more columns if we wanted for other independent variables, or something like $x_i^2$ if we expect a quadratic relationship, and the work we've already done would scale, but more on that in a bit - essentially, the column of 1s is just to fit with the pattern that the rest of $\mathbf{X}$ follows.

There's just one last tool we need to introduce, which is the matrix transpose, denoted $\mathbf{M^T}$ for a matrix $\mathbf{M}$, which involves, quite simply, swapping the columns and rows of the matrix, such that the element in row $i$, column $j$ in $\mathbf{M^T}$ is the element in row $j$, column $i$ in $\mathbf{M}$. So,

$$
\mathbf{X^T} =
\begin{pmatrix}
1 & 1 & \cdots & 1 \\
x_1 & x_2 & \cdots & x_n \\
\end{pmatrix}
$$

Now we've finally got to the really exciting part. Let's evaluate $\mathbf{X^TX}$, recalling our definition of matrix multiplication from earlier:

$$
\mathbf{X^TX} =
\begin{pmatrix}
1 & \cdots & 1 \\
x_1 & \cdots & x_n \\
\end{pmatrix}
\begin{pmatrix}
1 & x_1 \\
\vdots & \vdots \\
1 & x_n
\end{pmatrix} =
\begin{pmatrix}
1 + \cdots + 1 & x_1 + \cdots + x_n \\
x_1 + \cdots + x_n & x_1^2 + \cdots + x_n^2
\end{pmatrix} =
\mathbf{M_X}
$$

and it turns out this is exactly what we had called $\mathbf{M_X}$ before!! And if we find $\mathbf{X^Ty}$,

$$
\mathbf{X^Ty} =
\begin{pmatrix}
1 & \cdots & 1 \\
x_1 & \cdots & x_n \\
\end{pmatrix}
\begin{pmatrix}
y_1 \\
\vdots \\
y_n
\end{pmatrix} =
\begin{pmatrix}
y_1 + \cdots + y_n \\
x_1y_1 + \cdots + x_ny_n
\end{pmatrix} =
\mathbf{M_{XY}}
$$

and this is $\mathbf{M_{XY}}$! So not only do matrices and their associated operation of multiplication provide the perfect way to convert our simultaneous equations into a nicer form, but they even let us express the matrices involved in those equations in an extremely compact way and in terms of matrices that are essentially just our data. Finally, then, we're able to conclude that 

$$\hat{\beta} = (\mathbf{X^TX})^{-1}\mathbf{X^Ty}$$

and this is exactly what we wanted. Now, literally all we have to do is put our data into two matrices, program our computer with a couple of the most basic matrix operations, and we instantly get linear regression for free[^3]! Admittedly, we've only bothered deriving this equation when there are only two $\beta$ values, $\beta_0$ and $\beta_1$, but hopefully it isn't too hard to see how the derivation could scale to allow for arbitrarily many independent variables (it's really just an exercise in writing $\cdots$, $\vdots$ and even a $\ddots$ from time to time). Alternatively, as alluded to earlier, we could instead (or also) populate our $\mathbf{X}$ matrix with various non-linear functions of the $x_i$ values, such $x_i^r$, $\sin{x_i}$, $\log{x_i}$ and indeed literally anything else we could want.

Now, as an offering to the A-level maths gods, why don't we do a victory lap and find ourselves an explicit expression for $\beta_0$ and $\beta_1$ in our original situation? The only extra thing we need to know is how to find the inverse of a 2x2 matrix, which isn't too bad:

$$
\mathbf{M} =
\begin{pmatrix}
a & b \\
c & d
\end{pmatrix}
\Rightarrow \mathbf{M}^{-1} =
\frac{1}{ad-bc}
\begin{pmatrix}
d & -b \\
-c & a
\end{pmatrix}
$$

The quantity $ad - bc$, by the way, is called the determinant of the matrix, and has some other very interesting properties. Anyway, now we have (introducing $\bar{x}$ and $\bar{y}$ for the means of the $x_i$ and $y_i$ values respectively):

$$
\begin{align*}
\begin{pmatrix}
\beta_0 \\
\beta_1
\end{pmatrix} &= 
\begin{pmatrix}
n & n\bar{x} \\
n\bar{x} & \displaystyle{\sum_{i=1}^n x_1^2} \\
\end{pmatrix}^{-1}
\begin{pmatrix}
n\bar{y} \\
\displaystyle{\sum_{i=1}^n x_iy_i}
\end{pmatrix} \\
&= \frac{1}{n\displaystyle{\sum_{i=1}^n x_1^2} - n^2\bar{x}^2}
\begin{pmatrix}
\displaystyle{\sum_{i=1}^n x_1^2} & -n\bar{x} \\
-n\bar{x} & n \\
\end{pmatrix}^{-1}
\begin{pmatrix}
n\bar{y} \\
\displaystyle{\sum_{i=1}^n x_iy_i}
\end{pmatrix} \\
&= \frac{1}{n\displaystyle{\sum_{i=1}^n x_1^2} - n^2\bar{x}^2}
\begin{pmatrix}
n\bar{y}\displaystyle{\sum_{i=1}^n x_1^2} - n\bar{x}\displaystyle{\sum_{i=1}^n x_iy_i} \\
n\displaystyle{\sum_{i=1}^n x_iy_i} - n^2\bar{x}\bar{y}
\end{pmatrix}
\end{align*}
$$

I fear that we're reaching a number-crunching zone, and it might be time for an exercise for the reader! Let's just consider $\beta_1$ to finish:

$$
\begin{align*}
\beta_1 &= \frac{n\displaystyle{\sum_{i=1}^n x_iy_i} - n^2\bar{x}\bar{y}}{n\displaystyle{\sum_{i=1}^n x_1^2} - n^2\bar{x}^2} \\
&= \frac{\frac{1}{n-1}\left(\displaystyle{\sum_{i=1}^n x_iy_i} - n\bar{x}\bar{y}\right)}{\frac{1}{n-1}\left(\displaystyle{\sum_{i=1}^n x_1^2} - n\bar{x}^2\right)} \\
&= \frac{\mathrm{cov}(x, y)}{\mathrm{cov}(x, x)} = \frac{\mathrm{cov}(x, y)}{\mathrm{var}(x)}
\end{align*}
$$

If you don't know what $\mathrm{cov}$ (covariance) and $\mathrm{var}$ (variance) mean, they're a bit like matrix multiplication - quite general functions that happen to be useful in loads of contexts, and therefore which it's convenient to express stuff in terms of.

Anyway, as promised, a final exercise for the curious reader (although not an exceptionally nice one): verify the second part of the formula we had at the start, that $\beta_0 = \bar{y} - \beta_1\bar{x}$.

[^1]: One of the less frequently appreciated things about squaring is that we get a factor of 2 both from differentiating $x^2$ and from repeated terms in the expansion of the residuals, allowing us to just cancel it out. To be fair, it wouldn't really be a big issue if we did just have some constant factor to deal with, but it's a nice touch.

[^2]: I'm being a bit sarcastic. In seriousness, though, the biggest advantage is simply practicality: if we wrote the full matrices out every time, we would be using a lot of space very quickly, as you can perhaps see in the final few lines deriving $\beta_1$.

[^3]: It turns out, for some very interesting reasons, even though matrix inversion is a convenient way to express the process of solving a set of simultaneous equations, it's never the best way to actually do it. See [this famous blog post](https://www.johndcook.com/blog/2010/01/19/dont-invert-that-matrix/) or alternatively [this one](https://gregorygundersen.com/blog/2020/12/09/matrix-inversion/) which goes into a bit more detail.

</body>