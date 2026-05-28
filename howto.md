---
layout: page
title: How To Obfuscate
permalink: /howto
---

{% include math.html %}

This page will describe how obfuscation works.

## Crash Course on Reversible Circuits

A _reversible circuit_ $$C$$ computes a bijective function over a fixed number of wires. $$C$$ consists of a sequence of $$m$$ gates $$g_1g_2\cdots g_m$$ where each $$g_i$$ applies a permutation to a subset of the wires. We call such circuits _reversible_ because they can be run forwards or backwards; the inverse is always computable. Since each $$g_i$$ computes a permutation, so does the entire circuit: $$C^{-1}=g_mg_{m-1}\cdots g_2g_1$$. We represent the number of _wires_ in the circuit as $$n$$.

> Insert diagram of a circuit labeled. Show example evaluation in both directions

The reversible gates that we consider are [their own inverse](https://en.wikipedia.org/wiki/Involution_(mathematics)), so for any gate $$g$$, we have that $$gg=1$$, the identity circuit. (This shows that $$CC^{-1}=1$$, as well.) We will sometimes write $$1_n$$ to explicitly represent the identity circuit over $$n$$ wires.

Since the permutation group is nonabelian, gates do not commute in general.

At first glance, we might expect that the set of reversible circuits on $$n$$ wires, $$\mathcal C_n$$, computes all of the $$n$$-bit permutations, $$\mathbb S_{2^n}$$. However, in the general case, reversible circuits only compute the _even_ permutations, $$\mathbb A_{2^n}$$.[^1] These are the permutations with an even number of transpositions.

[^1]: The special case is when we have gates which cover all wires of a circuit. In the standard model, we have three-wire Toffoli gates; thus, $$\mathcal C_3$$ spans $$\mathbb S_8$$. For all larger circuits, no gate can touch all wires.

> Cite Shende

However, we also know that $$\mathbb S_{2^n} \leq \mathbb A_{2^{n+1}}$$: so the set of $$(n+1)$$-wire circuits can compute odd permutations over $$n$$ wires, as well. In fact, we can make a slightly stronger claim: this extra wire, which we call an _ancilla_, always returns to its original value; intuitively, it just acts as a temporary register for the rest of the circuit.

### Complete Gate Sets

All S8, Boolean Control Funcs, CNT, Gate 57

### Gate 57

Cite Salo. Show 3-wire and ancilla constructions for all boolean control functions.

We have confirmed that all permutations $$\pi\in S_8$$ can be computed in $$\leq10$$ gates when restricted to gate 57. In general, however, exponentially many gates are required.

### Computing functions which are not bijections

We can embed non-bijective functions in a reversible circuit by only considering a subset of the bits. Say we want to construct a circuit for some function $$f:\{0,1\}^n\mapsto \{0,1\}$$. We will implement:

$$
C(x) = (f(x) \mid\mid r(x))
$$

where $$r:\{0,1\}^n\mapsto\{0,1\}^{n-1}$$ is the arbitrary function computed by the inner machinery of the circuit. $$(\cdot \mid\mid \cdot)$$ represents concatenation. Observe that reversibility requires that $$C^{-1}(f(x)\mid\mid r(x)) = x$$.

## Our Approach to Obfuscation

[CCMR25] postulates that the following procedure gives an obfuscation scheme for reversible circuits.

> Repeat $$\mathsf{poly}(n)$$ times:
> 
> 1. Choose a small ($$O(1)$$-size) subcircuit $$s$$ of $$C$$.
> 2. Replace $$s$$ with a random functionality equivalent circuit $$s'$$.

Intuitively, we might expect this to reach a stationary distribution after sufficiently many iterations. The interesting questions are how many iterations are needed, exactly how we sample subcircuits, and how we pick replacements. We also care about the blowup of the scheme: if, in expectation, $$\|s'\|/\|s\|=1+\varepsilon$$, then after $$k$$ rounds our circuit will have size $$\|C\|\cdot (1+\varepsilon)^k$$, which might grow with $$O(2^n)$$!

### Perfect Compressors give Perfect Obfuscators


## Next Steps

Try out one of the [challenges](/).

Browse Our github repository at [Local-Mixing/local_mixing](https://github.com/Local-Mixing/local_mixing).