---
title: ALCB Token Specification
author: chgorman
status: Draft
created: 2022-11-23
---

# ALCB Token Specification

The ALCB Token is the utility token of AliceNet.
It follows a bonding curve which is parameterized by
the total ETH reserves and specifies the ALCB/ETH (inverse price);
this is different from the standard parameterization,
which tracks the total token supply and specifies
the ETH/Token (price).

## Minting and Burning

We let $\rho(x)$ denote the ALCB/ETH (inverse price)
when there are $x$ ETH reserves.
If the total $x$ ETH reserves, then the total ALCB supply is

$$
P(x) = \int_{0}^{x} \rho(s)\,ds.
$$

We suppose that there are $r$ ETH reserves.
If $k$ ETH is used to mint additional ALCB,
then $T_{M}$ additional ALCB are minted with

$$
\begin{align*}
T_{M} &= \int_{r}^{r+k} \rho(s)\,ds \\
    &= P(r + k) - P(r).
\end{align*}
$$

Upon burning $T_{B}$ ALCB,
$k$ ETH will be received where we have

$$
\begin{align*}
T_{B} &= \int_{r-k}^{r} \rho(s)\,ds \\
    &= P(r) - P(r-k).
\end{align*}
$$

In this case, $k$ is implicitly defined;
solving for $k$ will depend on the definition of $\rho$.

## Bonding Curve Specification

The bonding curve follows the particular form

$$
\rho(x) = a\left[\frac{b-x}{\sqrt{c + (b-x)^2}} + 1\right] + d.
$$

This is an algebraic sigmoid bonding curve.
Here, we have the parameter values

```
a = 200
b = 2500 * 10 ** 18
c = 5611050234958650739260304 + 125 * 10 ** 39
d = 4
s = 2524876234590519489452
```

Here, we have

$$
s^{2} = c + b^{2}.
$$

While $s$ is not fundamental, it is a useful value in implementations.

Given this definition of $\rho$,
we have

$$
\begin{align*}
P(r) &= a\left[\sqrt{c + b^{2}} -\sqrt{c + (b-r)^{2}} + r\right] + dr
    \\
P^{-1}(m) &= \frac{-c_{1} + c_{2}m + a\sqrt{d_{0} - d_{1}m + m^{2}}}{c_{3}}.
\end{align*}
$$

Here, we have the constants

$$
\begin{align*}
c_{1} &= a\left[\left(a+d\right)\sqrt{c+b^{2}} + ab\right] \\
c_{2} &= a+d \\
c_{3} &= d\left(2a+d\right) \\
d_{0} &= \left[\left(a+d\right)\sqrt{c+b^{2}} + ab\right]^{2} \\
d_{1} &= 2\left[a\sqrt{c+b^{2}} + \left(a+d\right)b\right].
\end{align*}
$$

These constants may be computed exactly given values for
$a$, $b$, $c$, and $d$.

Together, $P$ and $P^{-1}$ may be used to perform
minting and burning operations for the bonding curve
defined by $\rho$.

This bonding curve is also discussed in a related article
[here](https://github.com/alicenet/.github/blob/main/docs/bonding_curve.pdf).

### Implementation Notes

Because this algorithm is implemented within Solidity,
all of the operations are performed on integers (`uint256`).
This requires the use of an Integer Square Root function.
A discussion on computing the Integer Square Roots may be found
[here](https://github.com/alicenet/.github/blob/main/docs/efficient_isqrt.pdf).