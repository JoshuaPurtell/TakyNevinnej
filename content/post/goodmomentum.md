+++
author = "Anonymous"
title = "What is a Good Momentum?"
date = "2020-08-01"
description = "Optimizing optimization paths."
tags = [
    "deep learning",
    "hyperparameters"
]
categories = [
    "math",
    "deep learning",
]
series = ["Abstract Deep Learning"]
aliases = ["migrate-from-jekyl"]
feature_image = "/images/gemma-evans-LTEo69JUv7o-unsplash.jpg"
+++

Nota Bene: This is the proof-heavy version of this article. To get a more results-dense version, click here.

In terms of hyperparameters, momentum may well inspire the most curiousity. Can the right momentum circumvent convergence to local minima? Could it improve convergence time? Where increasing the dimensionality of a parameter space improves performance via convexification at great computational expense, a good momentum implicitly convexifies the loss landscape at the cost of a cheap algorithmic overhead. At least, that's the hypothesis I'd like to test in this article. We'll look at representations of momentum-aided optimization, some results that can steer us away from very bad momentum regimes, and some results that can help us choose a good — if not the best — momentum regime.

First, let's introduce some structure to question. 
### DEFINITIONS NEED WORK
**Definition** Loss Landscape

A *Loss Landscape* is a functor $\lambda$ such that : $\lambda: X \times L \times \Pi \times Q \to \\{ (Q(x),\ell(\pi,x)) \forall x \in X, \pi \in \Pi,$ $\ell \in L\\}$ where $ \Pi \cong \mathbb{R}^N$ is the space of parameters, $L=\\{\ell: \Pi \times X \to \mathbb{R}\\}$ is a space of loss functions, $X$ is the input space (for the sake of convenience, if SGD uses N inputs, we define $X:=Y^N$, where $Y$ is the natural input space), and $Q$ is a distribution over $X$. 

**Definition** Path Space

A *path space* $P$ subject to $\lambda$ is a set of paths such that: $P_{\lambda} := \\{ p \mid p: I \to \lambda(\Pi,X,L,Q), p \in C^1 \\}$ 

**Definition** Momentum 

Momentum is a functor $ \mu_m$ such that $\mu_m $: $(\Pi,X,Q,L) \to \\{ (Q(x)R(p),m(\pi,x,p,\ell)) \forall x \in X, \pi \in \Pi,$ $ \ell \in L,p \in AP_{\lambda},A \in \text{Aut}(P_{\lambda})\\}$, where $R$ is a distribution over $p$ and $m: \Pi \times X \times P \times L \to \mathbb{R}^{|X|}$ is the associated momentum function. We denote the element $AP_{\lambda}$ as $P_{\mu_m}$.

To formulate our question using the above structure, we wish to identify $m^* $ such that $\text{argmax}_{m \in M} |\\{ x \in M \text{ such that } x \precsim m\\}| =m^* $, where $\succ$ is a total order on path distributions, instances of which we will now elaborate on.

**Definition** Plausible

An element $v \in P$ is *plausible* under the tuple $(X,Q,\pi,\ell)$ iff $\int_I \mathbb{I} [ |1 - \int_Q \mathbb{I} \\{ m(\pi,v[t],v,\ell) /|m(\pi,v[t],v,\ell)| - dv[t]/|dv[t]|\\} dq ] dt=0$

An element $v \in P$ is *finite plausible* if it is plausible and $\limsup_{t \to 1} v[t] = \liminf_{t \to 1} v[t]$.

Having formalized the set of paths which could be traversed via SGD, we are now able to introduce an ordering on momenta.

**Theorem** Plausible Orderability

The binary order $\succsim$ defined on the space of momenta $M$ as:
$m_1 \succsim m_2 \text{ iff } W(m_1)=|1-\mathbb{I}\\{\int_{(V \subset A_{\mu_{m_1}}P) \times Q} \Pi_1 \mu_{m_1} dQdU\\}|$ $\int_{(U \subset A_{\mu_{m_1}}P) \times Q} \Pi_1 \mu_{m_1} dQdU $ < $|1-\mathbb{I}\\{\int_{(V \subset A_{\mu_{m_2}}P) \times Q} \Pi_1 \mu_{m_2} dQdU\\}|$ $\int_{(U \subset A_{\mu_{m_2}}P) \times Q} \Pi_1 \mu_{m_2} dQdU = W(m_2)$
* where $U$ are *finite plausible*
* where $V$ are *infinite plausible*

is a total order on $M$.

**Proof**

**Theorem** 
