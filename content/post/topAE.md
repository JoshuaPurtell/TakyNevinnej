+++
author = "Anonymous"
title = "How Topology Can Help Inform Autoencoder Hyperparameter Selection, Part 1"
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
feature_image = ""
+++

Nota Bene: This is the proof-heavy version of this article. To get a more results-dense version, click here.

In a recent post on [StackOverflow](https://datascience.stackexchange.com/questions/80389/what-is-an-autoencoder/80404#80404), I postulate that autoencoders can serve as basically equivalent to a basis-free PCA regime sandwiched between nonlinear automorphisms, where the encoder-side automorphism is trained such that it sends objects to roughly the object in their homotopy class such that the loss introduced by description with N parameters is minimized, where N is the dimensionality of the autoencoder's code, and the decoder side automorphism is trained accordingly. That is, I postulate that the training regime is path-isomorphic in expectation ([PIIE]( {{< relref "post/definitions#markdown-header-losspath" >}})) with the following optimization regime: 


$$\max_{P,P'} \Sigma_{i=1}^N \frac{|p_i|}{2^i}$$

where $$ p : AE(P,P',I) \to \mathbb{R}^N \times N$$ is a PCA operator on the code of autoencoder $AE$ evaluated on an input set $I$ and using parameters $P,P'$ for the encoder and decoder, respectively.

This makes sense intuitively, but before going into theory, I want to convince you that there is some empirical evidence for this somewhat out there hypothesis to hold. So, let's dive into a trivial example!

## Linearizing Quadratics

To begin with, let's define a simple model

```
using Flux
using Flux: @epochs, onehotbatch, mse, throttle#, params
using Base.Iterators: partition
using Distributions
##creating simple 2-1-2 AE
function build_train_data()
    #batch size of 2, total size of 10
    function gen()
        x=rand(Uniform(0,1),1,100)
        y=x.^2
        xy = vcat(x,y)
        return xy
    end
    
    train_data=[gen() for i in 1:100]
    return train_data
end

function train()
    train_data=build_train_data()
    encoder=Dense(2,1,leakyrelu)
    decoder=Dense(1,2,leakyrelu)
    model = Chain(
        encoder,
        decoder
    )

    @info("Training model.....")
    loss(x) = mse(model(x),x)
    lr=1e-3
    opt = ADAM(lr)
    evalcb = throttle(() -> @show(loss(train_data[1])), 1)
    @epochs 10 Flux.train!(loss, Flux.params(model), zip(train_data), opt, cb = evalcb)
    return model
end
```

and evaluate it on small sample to make sure nothing went wrong
```
m =train()

x=rand(Uniform(0,1),1,1000)
y=x.^2
sample = vcat(x,y)
estimate=m(sample)

```

Ok, looks good. Now, let's see what's happening under the hood. Specifically, what the code image looks like.

```
p=Flux.params(m)
encoder=Dense(2,1,leakyrelu)
Flux.params(encoder)=p[1]
transform=encoder(sample)
```

Just what we wanted to see! What happens when we remove non-linearities? Well, we can slightly modify the above code to:

```
```

and yield the following code image:

```
```


