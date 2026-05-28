---
layout: post
title:  "2: Obfuscated Point Function"
date:   2026-05-24
---
{% include math.html %}


In this challenge, we have obfuscated a _point function_ -- that is, a function $$f$$ such that:

$$
f_{a,b}(x) = \begin{cases}
a &\text{if}\ x = b\\
b &\text{if}\ x = a\\
x & \text{otherwise}
\end{cases}
$$

When viewed as a permutation, $$f(\cdot)$$ comprises the single transposition $$(a\ b)$$.

## The challenge

Given the obfuscated point function below, find $$(a, b)$$. The circuit has 128 wires, so brute-forcing all possible inputs will take a while.