+++
author = "Anonymous"
title = "How Topology Can Help Inform Autoencoder Hyperparameter Selection"
date = "2020-08-01"
description = "A look at one way that continuity can constrain."
tags = [
    "topology",
    "autoencoder",
]
categories = [
    "math",
    "deep learning",
]
series = ["Abstract Deep Learning"]
aliases = ["migrate-from-jekyl"]
feature_image = "/images/pietro-jeng-n6B49lTx7NM-unsplash.jpg"
+++

Nota Bene: This is the proof-heavy version of this article. To get a more results-dense version, click here.

In a recent post on [StackOverflow](https://datascience.stackexchange.com/questions/80389/what-is-an-autoencoder/80404#80404), I postulate that autoencoders can serve as basically equivalent to a basis-free PCA regime sandwiched between nonlinear automorphisms, where the encoder-side automorphism is trained such that it sends objects to roughly the object in their homotopy class such that the loss introduced by description with N parameters introduces the least loss, where N is the dimensionality of the autoencoder's code (the decoder side automorphism is trained accordingly). That is, I postulate that the training regime is path-isomorphic in expectation ([PIIE]( {{< relref "post/definitions#markdown-header-losspath" >}})) with the following optimization regime: 
$$\max_{P,P'} \text{ s. t.} $$

