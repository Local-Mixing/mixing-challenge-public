---
layout: page
title: How To Obfuscate
permalink: /howto
---

{% include math.html %}

This page will describe how our proposed obfuscation scheme works, and give an overview of some related concepts.

Our goal is to construct indistinguishability obfuscation (iO) for _all_ circuits, by way of a special class known as _reversible_ circuits. We'll first discuss what, exactly, iO is, and then explain how reversible circuits work. Finally, we'll (try to) put the pieces together.

<!-- Auto-generated ToC -->
**Table of Contents**
1. toc
{:toc}

## Crash Course on Indistinguishability Obfuscation

In this section, we briefly touch on the definition and properties of iO. For a more formal treatment, see [[SW14]](https://eprint.iacr.org/2013/454) and [[JLS21]](https://eprint.iacr.org/2020/1003), as well as the [latter half of Vinod's crypto course](https://advancedcrypto.github.io/), which focuses on obfuscation.

An indistinguishability obfuscator $$\mathcal O(\cdot)$$ is a randomized algorithm which takes a program (or, here, a circuit) as input, and outputs a functionally-equivalent obfuscated circuit. The salient property is that the obfuscations of two equivalent programs are indistinguishable:

$$
C_1\equiv C_2 \implies \mathcal O(C_1) \approx \mathcal O(C_2)
$$

where $$\equiv$$ means "functional equivalence" ($$\forall x:C_1(x)=C_2(x)$$) and $$\approx$$ means "(computationally) indistinguishable." As a side effect, note that $$C_1 \equiv \mathcal O(C_1)$$, so $$\mathcal O(\mathcal O(C_1))\approx \mathcal O(C_1)$$. This is a surprising statement! It tells us that iO is "as good as it gets" -- more rounds of obfuscation don't make anything harder! Also, iO hides all details of the _implementation_: it only reveals the truth table.[^comp]

[^comp]: Define an iO scheme which, when given some program $$P$$ computing a function $$f$$, outputs the shortest circuit for $$f$$. This would give perfect iO: every such $$P$$ has an _identical_ obfuscation, so is trivially indistinguishable! Unfortunately, such an obfuscator would first have to solve the [_Minimum Circuit Size Problem_](https://mcsp.work) -- that is, just determining how _long_ such a circuit must be -- which we have no idea how to do! However, while we don't know how to build a perfect compressor, we _can_ build an _imperfect_ compressor; this is one practical route towards obfuscation that we have looked into.

**iO is not black-box obfuscation.** The pipe dream of ideal obfuscation -- which renders any program $$P$$ a black box -- is impossible. Even a slight weakening to "virtual black-box obfuscation" (which merely asks for a distinguisher) is also known to be impossible [[BGI+01]](https://eprint.iacr.org/2001/069).

However, this doesn't mean we can't do interesting stuff with iO! Many of the interesting things we can do with it are made even more interesting by the fact that iO exists even if $$\mathbf{P}=\mathbf{NP}$$.

> **Note.** As written, the following does not hold. There's a distributional property we need to account for.

For example, we know that for some pseudorandom function $$F$$, $$F(k_1, \cdot)\approx F(k_2, \cdot)$$ for two distinct keys $$k_1, k_2$$. By the security property of iO,

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

![example circuit](/assets/img/ckt/example.png){: width="75%"}

> TODO:  Show example evaluation in both directions

The reversible gates that we consider are [their own inverse](https://en.wikipedia.org/wiki/Involution_(mathematics)), so for any gate $$g$$, we have that $$gg=1$$, the identity circuit. For example, gates 5 and 6 above cancel. (This is also why $$CC^{-1}=1$$.) We will sometimes write $$1_n$$ to explicitly represent the identity circuit over $$n$$ wires.

However, since the permutation group is nonabelian, gates do not commute in general:

![don't commute](/assets/img/ckt/no-commute.png){: width="50%"}

At first glance, we might expect that the set of reversible circuits on $$n$$ wires, $$\mathcal C_n$$, computes all of the $$n$$-bit permutations, $$S_{2^n}$$. However, in the general case, reversible circuits only compute the _even_ permutations, $$A_{2^n}$$.[^1] These are the permutations with an even number of transpositions. See Shende et al. [[SPMH03]](https://doi.org/10.1109/TCAD.2003.811448) for more.

[^1]: The special case is when we have gates which cover all wires of a circuit. In the standard model, we have three-wire Toffoli gates; thus, $$\mathcal C_3$$ spans $$S_8$$. For all larger circuits, no gate can touch all wires.

However, we also know that $$ S_{2^n} \leq A_{2^{n+1}}$$: the set of $$(n+1)$$-wire circuits can compute odd permutations over $$n$$ wires, as well. In fact, we can make a slightly stronger claim: this extra wire, which we call an _ancilla_, always returns to its original value; intuitively, it just acts as a temporary register for the rest of the circuit.

### Complete Gate Sets

Reversible gates have _control pins_, which they read values from, and _active pins_, which they modify.

TODO:
- Boolean control functions
- CNT, NT libraries
- All of S8

### Gate 57

There is one special gate which can compute all other ones: it is a Toffoli gate with the `NIMPLY` control function. Salo [[Salo20]](https://doi.org/10.1007/s10801-021-01053-7) proved that this gate (as well as its complement) are universal. Per that paper's nomenclature, we also refer to this gate as "rule 57" (under [Wolfram's taxonomy](https://atlas.wolfram.com/01/01/57/)). Elsewhere, we will refer to this gate as `r57`.

The behavior of `r57` on an active pin, positive control, and negative control $$(a, c^+, c^-)$$ is:

$$
\begin{align}
a&\to a\oplus(c^+ \vee \neg\,c^-) \\
 &= a \oplus (\neg\,c^+ \wedge c^-) \oplus 1\\
 &= a \oplus c^+c^- \oplus c^- \oplus 1
\end{align}
$$

That is, the gate flips its active pin when $$c^+=1$$ and $$c^-=0$$.

On three wires, `r57` can emulate all other Boolean control functions with at most eight gates:

> TODO

While with an ancilla wire, six gates suffice:

> TODO

By exhaustive search, we have confirmed that all $$8!=40320$$ permutations $$S_8$$ can be computed in at most ten gates (over three wires) when restricted to gate `r57`. In general, however, exponentially many gates are required to compute an arbitrary permutation. This can be seen by a simple counting argument:

Over $$n$$ wires, there are $$(2^n)!/2=\vert\mathbb A_{2^n}\vert$$ even permutations. With gate `r57`, there are about $$n^{3m}$$ circuits with $$n$$ wires and $$m$$ gates. Thus, we need at exponentially many gates:

$$
\begin{align}
n^{3m} &\geq (2^n)! / 2 \\
3m\log n & \gtrsim \frac{2^n \log 2^n}2\qquad \text{by Stirling}\\
m & \geq \frac{n\cdot 2^n}{6\log n}
\end{align}
$$

Zakablukov [[Zak17]](https://doi.org/10.1016/j.jcss.2016.09.010) provided a constructive proof that at most $$(1+o(1))\cdot 48n2^n / \log n$$ gates are required; thus, most permutations require $$\Theta(n\cdot 2^n / \log n)$$ gates.[^zak]

[^zak]: Note that their result applies to the "CNT" gate set (CNOT, NOT, Toffoli), but we can simulate this gate set with `r57` with a multiplicative blowup of at most six, and in practice probably closer to two.

### Computing functions which are not bijections

We can embed non-bijective functions in a reversible circuit by only considering a subset of the output bits. Say we want to construct a circuit for some function $$f:\{0,1\}^n\mapsto \{0,1\}$$. We will implement:

$$
C(x) = (f(x) \| r(x))
$$

where $$r:\{0,1\}^n\mapsto\{0,1\}^{n-1}$$ is some arbitrary function computed by the inner machinery of the circuit. $$(\cdot \| \cdot)$$ represents concatenation. Observe that reversibility requires that $$C^{-1}(f(x)\| r(x)) = x$$.

- Some functions may require ancilla bits during computation.
- So while $$f$$ might only output a bit, the shortest circuit may require exponentially many gates or many ancillas
- Reversible circuit only has $$n$$ bits of space (logspace?)

## Our Approach to Obfuscation

[[CCMR24]](https://link.springer.com/chapter/10.1007/978-3-031-78023-3_2) conjectures that the following procedure gives an obfuscation scheme for reversible circuits.

> Repeat $$\mathsf{poly}(n)$$ times:
> 
> 1. Choose a small ($$O(1)$$-size) subcircuit $$s$$ of $$C$$.
> 2. Replace $$s$$ with a random, functionality equivalent circuit $$s'$$.

The hope is that this procedure reaches a stationary distribution after sufficiently many iterations. The interesting questions are how many iterations are needed, exactly how we sample subcircuits, and how we pick replacements. We also care about the blowup of the scheme: if, in expectation, $$\|s'\|/\|s\|=1+\varepsilon$$, then after $$k$$ rounds our circuit will have size $$\|C\|\cdot (1+\varepsilon)^k$$, which might grow with $$O(2^n)$$!

While finding functionally-equivalent circuits is hard in general, since $$s$$ is constant-size, we may be able to brute-force all possible circuits. In practice, we build databases of equivalence classes of small circuits.

<!-- **Notation.** In the challenges, we will represent gates as a string of three alphanumeric characters. These represent the active, positive control (??), and negative control, respectively. For example, this circuit

![example circuit](/assets/img/ckt/small-example.png){: width="25%"}

is compactly represented by the string `301;032;102`. We label the wires $$0$$ (top) to $$n-1$$ (bottom). -->

<!-- TODO: define convex subcircuits -->

### Some History

We have spent quite a while experimenting with the general framework, and searching for concrete approaches which (for now, at least empirically) appear secure.

Our first attempt chose subcircuits at random, as initially proposed in the paper. However, we observed that it was hard to efficiently sample "interesting" convex subcircuits, so while we were able to _expand_ circuits substantially, little obfuscation of their functionality appeared to be happening. Indeed, many of our circuits were easily compressed back down to their original implementations (or nearly so).

As a validation technique, we began looking at _heatmaps_ of circuits. Given a circuit $$C$$ and its obfuscation $$\mathcal O(C)$, we tracked the correlations in the intermediate states of both circuits as various random inputs propagated through the circuit. If intermediate values were highly correlated, this suggested that we were merely _stretching_ the circuit, rather than obfuscating it. Indeed, many of our early approaches exhibit very high intermediate state correlations.

Another approach which seemed promising was the insertion of so-called "butterflies." Assume a circuit $$C$$ consists of $$m$$ gates, $$C=g_1g_2\cdots g_m$$. Then, we create a functionally-equivalent stretched circuit:

$$
g_1 R_1R_1^{-1} g_2 R_2R_2^{-1} g_3 \cdots g_{m-1} R_{m-1}R_{m-1}^{-1} g_m
$$

Since all $$R_iR_i^{-1}=1$$ cancel, the functionality of the new circuit does not change. Next, compress each chunk:

$$
\begin{align}
A_1 &= \mathsf{compress}(g_1 R_1) \\
A_2 &= \mathsf{compress}(R_1^{-1} g_2 R_2)\\
&\vdots\\
A_i &= \mathsf{compress}(R_{i-1}^{-1}g_i R_i)\\
&\vdots\\
A_m &= \mathsf{compress}(R_{m-1}^{-1} g_m)
\end{align}
$$

The final obfuscated circuit is $$A_1A_2\cdots A_m$$. Since each $$A_i$$ consists of two uniformly random circuits, the hope was that our randomized compressor would be able to "hide" the functionality of each $$g_i$$ in the compressed $$A_i$$ block. Unfortunately, while this scheme made circuits that were _hard to compress_, it did not significantly mix the internal state of the circuit, so significant correlations with the plaintext circuit persisted.

<!-- Strangely, we observed that the above asymmetric construction performed just as well as a symmetric version, where all $$R_i$$ were identical. -->

**To be discussed.**
- Construction of rainbow tables, randomized compressor.
- Shooting game; single-block replacements, extended collisions.
- Expand-compress loop.
- Insertion of linear gates (`SAMF`): full shuffle of the wires; random NOT gates.
- Secret sharing: virtual wire values and homomorphic gate evaluation. "Bath" of high-degree, pseudorandom wire values.
- SAT hardness.
- Feistel structure. Preventing evaluation in the reverse direction. Construction gives many useful primitives, but may also be important for the obfuscation scheme itself.

<!-- Inserting linear gates (SAMF) gave good heatmaps. Full shuffle of the wires, random NOTs. Push this through to the end -- and undo. In the middle, just remember what we did. Also tried partial linear changes: just touch a few wires at a time. Combination of long-term linear changes seems to be working best.

Question: why only linear ... operations? Arbitrary linear could encode any matrix mult -- but then a huge blowup. Gate becomes a whole matrix, quadratic. Gate itself is non-linear. But swaps & flips don't have the blowup, and are easy to undo: maintain intermediate state easily. -->

<!-- Issue: just linear + local mixing, not actually clear that you've obfuscated anything. Algebraic degree low. This is bad -- can recognize, open to cryptanalysis. Ideally, we want a cut between any two random places to look like a random circuit of that size.

Idea: Virtual wire values - secret sharing - homomorphic gate evaluation. double the wires, say; each old wire is XOR-shared over two new wires. Maybe also three. Rerandomize, cycle the random values. "Heat bath of randomness." All wires are high degree of arbitrary others; secret shared value need not be. -->
<!-- 
> In some sense, this is all that should be necessary... SAT solver.

Very strong attack, and not clear how to defend: not a formal attack framework. SAT hardness. What gives it? Why do we (not) have SAT hardness?

What is it that changed? Initially, SAT solver was unable to work. Used LLM to automatically attack.

Feistelization. How to prevent backward operation. Initial idea. Get Nicholas's gadget. -->

For further technical information, see [the documentation](https://github.com/Local-Mixing/local_mixing/blob/shuffletests/docs/Mixing_Pieces_Documentation.md) for our draft implementation.

### From Reversible Circuits to All Circuits

With the above focus on reversible circuits, it is natural to wonder how we might use such a scheme in practice. If a program is easily represented as a reversible circuit, there is nothing to do, but a more general scheme should support _any_ circuit, including non-reversible ones.

[[CCMR24, Appendix A]](https://link.springer.com/chapter/10.1007/978-3-031-78023-3_2) provides an elegant conversion. Existing synthesis techniques require ancilla wires with _fixed_ inputs (e.g., 0), and may not preserve correctness otherwise. This raises a security concern for an obfuscation scheme; they write:

> Indeed, this under-specification opens the door to situations where even a perfectly obfuscated version of the reversible embedding of a non-reversible circuit C would leak information on the internals of C when the ancilla wires are set to "illegitimate" values.

In short, their proposal is to embed a general circuit $$C(\cdot)$$ into a reversible one, using a technique due to [[Toffoli80]](https://doi.org/10.1007/3-540-10003-2_104), and then add a "wrapped" circuit which _enforces_ that all ancillas have the correct values. This hardened circuit acts as the identity circuit _unless_ all ancillas are correct, in which case it computes the original functionality $$C(x)$$. Once this entire circuit has been obfuscated, the specific implementation of $$C(\cdot)$$ -- and thus any leaky behavior due to invalid ancillas -- is hidden.

## Next Steps

Try out one of the [challenges](/).

Browse our github repository at [Local-Mixing/local_mixing](https://github.com/Local-Mixing/local_mixing).

---

_Footnotes_
