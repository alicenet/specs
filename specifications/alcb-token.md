---
title: ALCB Token Specification
author: chgorman
status: Draft
created: 2022-11-23
---

# ALCB Token Specification

The ALCB Token (BToken) is the utility token of AliceNet.
It follows a bonding curve which is parameterized by
the total ETH reserves and specifies the BTokens/ETH (inverse price);
this is different from the standard parameterization,
which tracks the total token supply and specifies
the ETH/Token (price).

## Bonding Curve Specification

The bonding curve follows the particular form

$$
\rho(x) = a\left[\frac{b-x}{\sqrt{c + (b-x)^2}} + 1\right] + d.
$$

This is an algebraic sigmoid bonding curve.
Here, we have the parameter values

```
a = 200;
b = 2500 * 10 ** 18;
c = 5611050234958650739260304 + 125 * 10 ** 39;
d = 4;
s = 2524876234590519489452;
```

Here, we have

$$
s^{2} = c + b^{2}.
$$

While $s$ is not fundamental, it is a useful value in implementations.
