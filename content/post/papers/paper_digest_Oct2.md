+++
author = "Anonymous"
title = "Paper Roundup Week of October 2"
date = "2020-08-01"
description = "Paper Roundup"
tags = [
    "Spectral Graph Theory",
]
categories = [
    "math",
    "algebra and group theory",
]
series = ["Paper Roundups"]
aliases = ["migrate-from-jekyl"]
feature_image = ""
+++

<script type="text/javascript"
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

It's time for the weekly paper roundup! My hope is for this segment to roughly reduplicate the write-ups I build on [Obsidian](https://obsidian.md/). Generally speaking, the inclusion of a paper on the roundup is not an endorsement, and the amount of text I devote to a given entry is only a noisy indicator of the value I ascribe to it. With that being said, let's jump in.

[On Spectral Clustering: Analysis and an Algorithm](https://papers.nips.cc/paper/2092-on-spectral-clustering-analysis-and-an-algorithm.pdf)

The main conceit of this paper is to cast data $S$ into a normalized affinity space (where affinity decays according to some hyperparameter $\sigma^2$ such that $A_{ij}=e^{\frac{-\left\lVert s_i-s_j \right\rVert^2}{2\sigma^2}}$). One does this by constructing $X=[x_1,...,x_k]$ where $x_i$ is the $i^{th}$ eigenvector of $L=D^{-1/2}AD^{-1/2}$, normalizing row norm to 1, and clustering in the ambient space $Y$.

In terms of performance, the algorithm seems to perform well enough for the first trivial non-convex clustering that I tried, concentric circles (this isn't very original — Ng et al use it in the paper).

Here is result I get using a naive initialization of knn (code [here](https://github.com/JoshuaPurtell/code_vignettes/blob/master/10_2_2020/spectral_clustering.jl)):

<figure class="video_container">
<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vQbBri54KfQ5bEeekq-CJY40nMQOzs9QInC46tdUADzlDa10rgfQD-sw39OKBxRzNBfCblA6YQpvNDT/embed?start=false&loop=false&delayms=5000" frameborder="0" width="480" height="299" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>
</figure>

A few comments on the above: firstly, the only reason why there is a natural (2D) representation for $X,Y$ is that I chose $k=2$, as $X,Y$ are $|S| \times k$ arrays and the rowwise correspondence means that each $s \in S$ corresponds with a $k-vector$ in $X,Y$ space. Furthermore, while the transformation $A \to L$ appears to draw the clusters nearer together, it doesn't appear to impact the eigenvector space ($X$) much at all. Indeed, there were some runs where the populations in $A$ did not overlap, did so in $L$, and yet were clearly separated in $X,Y$. My hunch is that this would be really easy to prove using either Matrix Perturbation Theory or Spectral Graph Theory, but since I've yet to establish a firm foundation in either (although I've begun on both at the behest of this paper), this is yet to be seen. [If this is obvious to you, please contact me and I will both recognize your contribution here (if you'd like), and let you recommend one of the successive week's papers! Just find my email in the contact page]. Anyways, the next note is that it doesn't appear that the normalization step from $X \to Y$ really did much. I coded the step as 

<img src="/post/papers/images/Y_to_X.png" alt="drawing" width="200"/>


which appears to be correct; given the circumstances, though, it's entirely possible that I've made a really embarrassing and obvious error. Hopefully, it's an artifact of the problem set up, or the eigensolver used? The greatest piece of evidence to support this hope is the fact that the data in $X$ is indeed distributed as one would expect a row-normalized sample would be, with the seeming variance difference between clusters in the horizontal direction rationalizable as being an artifact of the large natural separation between clusters in the vertical direction.

Speaking of said artificial variance discrepancy, it has caused non-trivial problems. If you refer to the [code](https://github.com/JoshuaPurtell/code_vignettes/blob/master/10_2_2020/spectral_clustering.jl), it's clear that I "cheat" by restarting the final knn step for any convergence which assigns clusters that vary greatly in size. The motivation being that the conditional problem of the random seeding having chosen one seed from each natural population given that the identified clusters are close in size is quite high. Using any of [Clustering.jl's](https://juliastats.org/Clustering.jl/stable/index.html) out of the box seeding algorithm alternatives was not fruitful, both in the case of kmeans and kmediods (somehow, the latter was even more pathological than the former). My sense is that these seeding algorithms are failing mainly because of the variance difference between the clusters in $X,Y$ space; in order to repair this, I attempted to reduce the natural separation (and thereby the artificial variance discrepancy, per the above observation) by varying $\sigma^2$ from {$5e7,5e6,...,5e{-7}$}. It did not have the intended effect, which seems to be a real problem for this implementation of the algorithm. A reliable way to circumvent this would be to use a Gaussian Mixture Model to cluster the populations in $Y$ space, which so obviously sidesteps the issue of random seeding that implementing it here didn't seem particularly worthwhile.

For those wishing to implement Ng's algorithm with knn, the above implementation leaves much to be desired. Because Ng's paper features example populations which are much more tightly clustered in $Y$ space, I suspect that either this problem is completely explainable by a (very) possible error in the row-normalization step, or that it could be circumvented by using an automatic $\sigma^2$ search method.

Which provides a natural segue back to the paper; Ng et al recommend selecting a $\sigma^2$ value by minimizing the induced cluster "tightness." They don't offer a specific approach to doing so, but my guess is that such an algorithm would solve a relaxation of the following optimization problem:

$$\min_{\sigma^2} \max_i(\ell) \text{ s. t. }\ell_i = k_{Ch}(C_i)$$ where $k_{Ch}(C_i)$ is the Cheegar constant of cluster $C_i$.

Of course, finding the Cheegar constant is an NP-hard problem (classically), so this doesn't exactly make for the fastest-converging approach. Alternatively, Ng suggests that simply bounding the mixing time from above ought to equivalently induce the relaxed condition; such a bound can be applied via an upper bound on the second eigenvalue of the transition matrix corresponding to $A$, $M$ ($M_{ij}=\frac{2\sigma^2}{A_{ij}}, \forall i \neq j; M_{ii}=0$). That is, by solving:

$$\min_{\sigma^2} \max_i(\ell) \text{ s. t. } \ell_i = \lambda^{M^i}_2, M^i \text{ is the transition matrix for cluster }i$$ 

I did so, and the results are ... mixed. Here are some plots of the losses found at $\sigma^2=5*10^x$ for different values of $x$ and $|S|=50,500$.

<figure class="video_container">
<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vSMSj36VBn2uiPXMKXcfgVMGpXsz79eeOaehbF8i_vQbyt8Z87FeLIpa4p2TLKYxIt7ROYIrLzldniU/embed?start=false&loop=false&delayms=5000" frameborder="0" width="480" height="299" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>
</figure>

I imagine that the mode collapse owes much both to the low number of natural clusters and to the seeding issue mentioned above (crucially, while the conditional probability of getting a "good" seeding with the restart mechanism used is high, it's not particularly high). If applied to an example on a richer underlying domain space, with more numerous and less separated clusters, using gmm in lieu of knn, I imagine the loss image may look a bit more interesting. But, this seems to be a natural place to conclude my empirical treatment of this paper.

On the the theory and methods side of things, Prop. 1 gives a somewhat trivial result on the "ideal" case, where clusters are infinitely mutually distant, and Theorem 2 uses some assumptions on the clusters (how "tight" they are relative to their mutual distance) to get a result that bounds how "loose" clusters in $Y$ space can be in the best case. This is powerful but not particularly groundbreaking; it's clear that this paper serves to formalize a folk belief on the efficacy of a general class of spectral methods, and elaborate a particularly useful member of that class. What may be a bit more noteworthy is the specific approach Ng et al take; they elaborate the result for a simple, perfect case and then find the assumptions necessary to extend the reasoning to a much larger subset of the case space. Of course, developing intuition with a trivial example is not a technique uniquely employed by Ng et al; what is remarkable is the elegance which accompanies the employ. I recommend reading this (short) paper mostly on this basis.

Overall ratings: Theory-2/5,Results-3/5,Methods-4/5


[A Polynomial Time Algorithm for Rayleigh Ratio on Discrete Variables](https://pubsonline.informs.org/doi/pdf/10.1287/opre.1120.1126)


