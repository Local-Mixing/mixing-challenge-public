---
layout: page
title: How To Obfuscate
permalink: /howto
---

{% include math.html %}

This page will describe how our proposed obfuscation scheme works.

## Crash   Course on Indistinguishability Obfuscation

Our goal is to construct indistinguishability obfuscation (iO) for circuits, by way of _reversible_ circuits (defined next). In this section, we briefly touch on the definition and properties of iO. For a more formal treatment, see [SW14] [JLS21].

An indistinguishability obfuscator $$\mathcal O(\cdot)$$ is a randomized algorithm which takes a program (or, here, a circuit) as input, and outputs a functionally-equivalent obfuscated circuit. The salient property is that the obfuscations of two equivalent programs are indistinguishable:

$$
C_1\equiv C_2 \implies \mathcal O(C_1) \approx \mathcal O(C_2)
$$

where $$\equiv$$ means "functional equivalence" ($$\forall x:C_1(x)=C_2(x)$$) and $$\approx$$ means "(computationally) indistinguishable." As a side effect, note that $$C_1 \equiv \mathcal O(C_1)$$, so $$\mathcal O(\mathcal O(C_1))\approx \mathcal O(C_1)$$.

**iO is not black-box obfuscation.** The pipe dream of obfuscation would be something known as "virtual black-box obfuscation," which has been proven impossible. [cite] VBB would require...

However, this doesn't mean we can't do interesting stuff with iO.

For example, we know that for some pseudorandom function $$F$$, $$F(k_1, \cdot)\approx F(k_2, \cdot)$$ for two distinct keys $$k_1, k_2$$ (tighten up, on different inputs, distributionally, etc.). By the security property of iO,

$$
\mathcal O(F(k_1, \cdot)) \equiv F(k_1, \cdot) \approx F(k_2, \cdot) \equiv \mathcal O(F(k_2, \cdot))
$$

so we cannot tell which key has been "embedded" into the obfuscated program. Thus we can do neat things like build public-key encryption out of symmetric key cryptography:

$$
\begin{align}
k&\overset $ \gets \{0,1\}^\lambda\\
\mathsf{sk}&:=k\\
\mathsf{pk}&\gets \mathcal O(\mathrm{AES}(k, \cdot))\\
\mathsf{Enc}(\mathsf{pk}, m)&:=\left(r\overset $ \gets \{0,1\}^\lambda,\mathsf{pk}(m\oplus r) \equiv \mathrm{AES}(k, m\oplus r)\right)\\
\mathsf{Dec}(\mathsf{sk}, (r, c))&:=\mathrm{AES}(k, c)\oplus r
\end{align}
$$

(or choose your favorite block cipher mode of operation.) It is safe to publish $$\mathsf{pk}$$ here because iO guarantees that all such public keys are indistinguishable from each other.

## Crash Course on Reversible Circuits

A _reversible circuit_ $$C$$ computes a bijective function over a fixed number of wires. $$C$$ consists of a sequence of $$m$$ gates $$g_1g_2\cdots g_m$$ where each $$g_i$$ applies a permutation to a subset of the wires. We call such circuits _reversible_ because they can be run forwards or backwards; the inverse is always computable. Since each $$g_i$$ computes a permutation, so does the entire circuit: $$C^{-1}=g_mg_{m-1}\cdots g_2g_1$$. We represent the number of _wires_ in the circuit as $$n$$.

> Insert diagram of a circuit labeled. Show example evaluation in both directions

The reversible gates that we consider are [their own inverse](https://en.wikipedia.org/wiki/Involution_(mathematics)), so for any gate $$g$$, we have that $$gg=1$$, the identity circuit. (This shows that $$CC^{-1}=1$$, as well.) We will sometimes write $$1_n$$ to explicitly represent the identity circuit over $$n$$ wires.

However, since the permutation group is nonabelian, gates do not commute in general.

![don't commute](/assets/img/ckt/no-commute.png){: width="50%"}

At first glance, we might expect that the set of reversible circuits on $$n$$ wires, $$\mathcal C_n$$, computes all of the $$n$$-bit permutations, $$\mathbb S_{2^n}$$. However, in the general case, reversible circuits only compute the _even_ permutations, $$\mathbb A_{2^n}$$.[^1] These are the permutations with an even number of transpositions.

[^1]: The special case is when we have gates which cover all wires of a circuit. In the standard model, we have three-wire Toffoli gates; thus, $$\mathcal C_3$$ spans $$\mathbb S_8$$. For all larger circuits, no gate can touch all wires.

> Cite Shende

However, we also know that $$\mathbb S_{2^n} \leq \mathbb A_{2^{n+1}}$$: so the set of $$(n+1)$$-wire circuits can compute odd permutations over $$n$$ wires, as well. In fact, we can make a slightly stronger claim: this extra wire, which we call an _ancilla_, always returns to its original value; intuitively, it just acts as a temporary register for the rest of the circuit.

### Complete Gate Sets

All S8, Boolean Control Funcs, CNT, Gate 57

### Gate 57

Cite Salo. Show 3-wire and ancilla constructions for all boolean control functions.

This gate computes the NIMPLY control function; on wires $$(a, c^+, c^-)$$, that is:

$$
\begin{align}
a&\to a\oplus(c^+ \vee \neg\,c^-) \\
 &= a \oplus (\neg\,c^+ \wedge c^-) \oplus 1\\
 &= a \oplus c^+c^- \oplus c^- \oplus 1
\end{align}
$$

The gate flips its active pin when $$c^+=1$$ and $$c^-=0$$.


We also refer to this gate as "rule 57" beacuse it can also be viewed as that numbered elementary cellular automata (under [Wolfram's taxonomy](https://atlas.wolfram.com/01/01/57/)). Elsewhere, we will refer to this gate as `r57`.

By exhaustive search, we have confirmed that all $$8!=40320$$ permutations $$S_8$$ can be computed in at most ten gates when restricted to gate `r57`. In general, however, exponentially many gates are required. This can be seen by a simple counting argument:

Over $$n$$ wires, there are $$(2^n)!/2=\vert\mathbb A_{2^n}\vert$$ even permutations. With gate `r57`, there are about $$n^{3m}$$ circuits with $$n$$ wires and $$m$$ gates. Thus, we need at exponentially many gates:

$$
\begin{align}
n^{3m} &\geq (2^n)! / 2 \\
3m\log n & \gtrsim \frac{2^n \log 2^n}2\qquad \text{by Stirling}\\
m & \geq \frac{n\cdot 2^n}{6\log n}
\end{align}
$$

Zakablukov[^zak] provided a constructive proof that at most $$(1+o(1))\cdot 48n2^n / \log n$$ gates are required; thus, most permutations require $$\Theta(n\cdot 2^n / \log n)$$ gates.

[^zak]: Zak 2016. Note that their result applies to the "CNT" gate set (CNOT, NOT, Toffoli), but we can simulate this gate set with `r57` with a multiplicative blowup of at most six, and in practice probably closer to two.

### Computing functions which are not bijections

We can embed non-bijective functions in a reversible circuit by only considering a subset of the output bits. Say we want to construct a circuit for some function $$f:\{0,1\}^n\mapsto \{0,1\}$$. We will implement:

$$
C(x) = (f(x) \mid\mid r(x))
$$

where $$r:\{0,1\}^n\mapsto\{0,1\}^{n-1}$$ is the arbitrary function computed by the inner machinery of the circuit. $$(\cdot \mid\mid \cdot)$$ represents concatenation. Observe that reversibility requires that $$C^{-1}(f(x)\mid\mid r(x)) = x$$.

Some functions require ancilla bits during computation. So while $$f$$ 

## History?

J's notes: https://hackmd.io/@gausslabs/ryOlMIWR0

First, tried to choose at random. Hard to sample convex subcircuits. Expansion possible, but compression, correlations, heatmaps

Making things incompressible was not easy. 

Butterflies. $$AgB$$

Symmetric ($$RgR^{-1}$$) and asymmetric (random each time) and compress. Surprisingly, minor differences between these. Incompressible, but hard to mix: doesn't really move state around.

Rainbow tables. Build compressor.

Shooting game. Instead of single-block replacements, extending collisions to create larger...

Expand-compress loop.

Inserting linear gates (SAMF) gave good heatmaps. Full shuffle of the wires, random NOTs. Push this through to the end -- and undo. In the middle, just remember what we did. Also tried partial linear changes: just touch a few wires at a time. Combination of long-term linear changes seems to be working best.


Question: why only linear ... operations? Arbitrary linear could encode any matrix mult -- but then a huge blowup. Gate becomes a whole matrix, quadratic. Gate itself is non-linear. But swaps & flips don't have the blowup, and are easy to undo: maintain intermediate state easily.

Issue: just linear + local mixing, not actually clear that you've obfuscated anything. Algebraic degree low. This is bad -- can recognize, open to cryptanalysis. Ideally, we want a cut between any two random places to look like a random circuit of that size.

Idea: Virtual wire values - secret sharing - homomorphic gate evaluation. double the wires, say; each old wire is XOR-shared over two new wires. Maybe also three. Rerandomize, cycle the random values. "Heat bath of randomness." All wires are high degree of arbitrary others; secret shared value need not be.

> In some sense, this is all that should be necessary... SAT solver.

Very strong attack, and not clear how to defend: not a formal attack framework. SAT hardness. What gives it? Why do we (not) have SAT hardness?

What is it that changed? Initially, SAT solver was unable to work. Used LLM to automatically attack.

Feistelization. How to prevent backward operation. Initial idea. Get Nicholas's gadget.

[Nicholas documentation](https://github.com/Local-Mixing/local_mixing/blob/shuffletests/docs/Mixing_Pieces_Documentation.md)


## Our Approach to Obfuscation

[CCMR25] postulates that the following procedure gives an obfuscation scheme for reversible circuits.



> Repeat $$\mathsf{poly}(n)$$ times:
> 
> 1. Choose a small ($$O(1)$$-size) subcircuit $$s$$ of $$C$$.
> 2. Replace $$s$$ with a random functionality equivalent circuit $$s'$$.

Intuitively, we might expect this to reach a stationary distribution after sufficiently many iterations. The interesting questions are how many iterations are needed, exactly how we sample subcircuits, and how we pick replacements. We also care about the blowup of the scheme: if, in expectation, $$\|s'\|/\|s\|=1+\varepsilon$$, then after $$k$$ rounds our circuit will have size $$\|C\|\cdot (1+\varepsilon)^k$$, which might grow with $$O(2^n)$$!

**Notation.** In the challenges, we will represent gates as a string of three alphanumeric characters. These represent the active, positive control (??), and negative control, respectively. For example, this circuit

![example circuit](/assets/img/ckt/small-example.png){: width="25%"}

is compactly represented by the string `301;032;102`. We label the wires $$0$$ (top) to $$n-1$$ (bottom).

### Perfect Compressors give Perfect Obfuscators

## From Reversible Circuits to All Circuits

CCMR appendix


## Next Steps

Try out one of the [challenges](/).

Browse our github repository at [Local-Mixing/local_mixing](https://github.com/Local-Mixing/local_mixing).

---

_Footnotes_
