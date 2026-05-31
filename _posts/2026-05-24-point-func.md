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

Since reversible circuits can only compute _even_ permutations, it's not actually possible to encode a single point function. Instead, we need an ancilla wire, which creates two transpositions. Then, for some secret key $$k$$, we can build a circuit computing the permutation

$$
(k\|00\ k\|01)(k\|10\ k\|11)
$$

where $$\cdot\|\cdot$$ represents bitstring concatenation. In this circuit, the active wire is the LSB, and the ancilla the second to last.

## An example

Here is an _unobfuscated_ point function on 32 wires that computes the cycle $$(k\|{*}0\ \ k\|{*}1)$$, for $$k=123456789$$.

```
# TODO
```

And an obfuscated version of the same:

```
# TODO
```

To generate your own point functions, see the file `point_function.rs`

## The challenge

The obfuscated point function below computes the cycle $$(k\|{*}0\ \ k\|{*}1)$$. Find $$k$$. (The circuit has 128 wires, so brute-forcing all possible inputs will take a while.)