---
layout: post
title: "Autoencoding Color-Language Representations"
author: "Solar Grammars"
categories: blog
tags: [blog]
image: blog/learn-color-reps/gre_set10.png
color: "#000000"
---

Autoencoding color representations


In this post, we will analyze and replicate an interesting report that appeared few years
ago, in which authors make use of an autoencoder to learn color-language representations. 

This idea is interesting as the autoencoder operates over the color descriptions by encoding
and decoding them while at the same time forcing the latent space to
be as close as possible to an actual colors associated to the descriptions. 

The report uses the data from the MacMahan paper, which we have been using extensively in this
blog, and that originally came from a survey on XCXCXC. 

The paper follows a standard AE architecture (is not a variatonal, as in the standard VAE formulation)
Given a pair (color, description), the encoder takes the sequence of tokens from the description and 
produce a vector representation. For this purpose, a GRU net is used. Again, as with all the papers
we have reviewed so far that use that dataset, I think using a recurrent neural network to encode
a description consisting of at most three tokens could be a little bit overkill, but lets stick to
the paper to see if we can get closer to the results reported.

So, a GRU rnn encodes the description into a vector representation, which is passed to a linear
layer to obtain latent representation, a three dimensional vector with
values between 0 and 1, which is set to match that normalized representation of the associated color.

The decoder takes the latent representation and map it back to the description representation space.
Such resulting vector is fed into another GRU rnn which generates one token at a time, hoping
to reconstruct the original description by softmaxing over the training vocabulary. 

In terms of the loss function, it is formalized as the sum of the `reconstruction` loss, 
basically the cross entropy between the generated and ground truth description, and the `visual` loss,
which the authors relate to how far is the learned latent representation associated to a description
to the actual color (expressed as a RGB tuple).

Such `visual` loss can be formulated in many ways, including cosine distance or simply euclidean distance.
The authors proposed the use of several alternatives, such as max-margin distance. The reported results
are obtained using an ad-hoc loss of the form $$ D_{RGB}(enc(x), v(x)) = || enc(x)2 - v(x)2||2 $$, where $$x$$
represents the description, $$enc(x)$$ the latent representation obtained from the encoder ,and 
$$v(x)$$ is the color representation (RGB vector).

There is an hyperparameter $$\mu$$ that controls the trade off betweeen the two losess, which I found
quite escesstual to obtain good results. 

