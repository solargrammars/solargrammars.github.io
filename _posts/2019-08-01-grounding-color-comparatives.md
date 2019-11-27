---
layout: post
title: "Grounding Color Comparatives"
author: "Solar Grammars"
categories: blog
tags: [blog]
image: arctic-1.jpg
color: "#D95B43"
---





The methods  we have covered so far allowed  us to learn  a direct mapping between colors and language. One possible limitation of those approaches is that in reality color perception is intrinsically contextual, in the sense that usually we come up with a description for a newly seen color based on a predefined reference. For example, when
we say a given color is `mustard yellow`, we are basically relating our experienced notion of `mustard`  and a reference `yellow` and associating them
to express our perception through language. Another example can be seen when we say `dark green` or `deep blue`. In these cases we are adding an adjective to a color based or our predefined notion of it. In that sense, in order to communicate our perception of  `dark green`, we use our preconceived reference of what we understand as `green` and its `dark` variant. In this process, we are implicitly internalizing that the color we are perceiving is `darker` than 
the reference we have. 

It could be interesting to model or quantify the distance between a predefined reference and a newly perceived color, and how such distance condition the language we use to  describe it.

In an [ACL'18 paper](https://www.aclweb.org/anthology/P18-2125), Winn et al. proposed a way to ground color comparatives as vectors in a color space. Let's assume we have a reference color `yellow` and a given target color `dark yellow`, both expressed as RGB points. The associated comparative in this case is `darker`, as naturally, `dark yellow`
 is *darker* than `yellow` :). The key idea is to represent a comparative between both colors as a vector rooted at the reference  and that points in the direction of the the target. 

The first interesting thing about that work is the construction of an ad-hoc dataset. The authors take as starting point  the dataset from [McMahan et al.](https://www.aclweb.org/anthology/Q15-1008) (which in turn is a processed version of the original [XKCD color survey](https://blog.xkcd.com/2010/05/03/color-survey-results/)). This dataset associates a color description with a RGB point.

The authors of this paper noticed that a considerable amount of descriptions contain
adjectives, such as `light green` or `dark purple`. From those adjectives, they obtained the associated comparatives (  `light` -> `lighter`, `dark` -> `darker`) so they can
conform tuples like  `(green, darker, dark green)`, where 
the first element represent the reference color as RGB tuple, the second is the associated comparative, and third element is the target color, also expressed as a  RGB tuple. 

Let's take a look a some tuples and their associated points, by plotting a two dimensional reduction via T-SNE. For example, in the following figure we can see how the RGB points for `lavender` are positioned  in relation to its available comparatives.  

![lavender](/assets/img/blog/color-comparatives/visual_analysis_lavender.png)

One thing we can immediately notice is that ... some does not quite resemble the common notion of `lavender`. But that is an interesting aspect of the dataset, as such categorization comes from a real survey, i.e., it compresses the diversity of perception from a huge number of people.  I added a marker associated to the mean value of comparative (`average` in this case means just plain `lavender`) and we can easy distinguish the defined color zones. Interestingly, the `light lavender` seems to be quite different from the rest, but between elements in this group, we still can see distances quite considerable. If we take a look at a couple more of examples,  we see a consistent distinction: 

`rose`:
![mint](/assets/img/blog/color-comparatives/visual_analysis_rose.png)


`turquoise`:
![mint](/assets/img/blog/color-comparatives/visual_analysis_turquoise.png)


`violet`:
![peach](/assets/img/blog/color-comparatives/visual_analysis_violet.png)


Something worth noticing there could be several ways to structure  a training set. In the case of the paper from Winn et al.,  instead of forming all possible reference-target pairs between RGB points, they assume that the references converge, therefore only the average values of those sets are used. 

The model consists of a linear layer that receives the concatenation of the  reference color $$r_c$$ (as RGB three-dimensional vector) and an embedding associated to the comparative $$w$$ (in the paper the authors used the  Google's 300-dimensional word2vec pretrained vectors). The resulting representation is fed to an additional layer along with the reference color and the output consists of a 3 dimensional vector $$\vec{w_g}$$ that represents the direction in the color space of the vector rooted at $$r_c$$ and the *follows* the direction towards the target $$t_c$$.  

The loss function has two components, the first one computes the cosine distance between the learned vector $$\vec{w_g}$$ and the vector that connects the reference and the target colors ( $$\vec{tr} =  t_c - r_c$$).
The second component compares the size of the learned vector against the size of the vector associated to the target color. With this, it is expected to modulate  the distance to the target color so that the colors in the line of the vector resemble the comparative. 

Let's take a look at some examples of learned comparative vectors.  In the case of the tuple `(lavender, darker, dark lavender)` the first figure shows the differences between the $$\vec{w_g}$$ and $$\vec{tr}$$ as we graph them in a 
three dimensional space. As RGB is not a well defined metric space, the colors were transformed into the LAB
space. The second figure shows the the color gradient associated to the $$\vec{tr}$$ vector and the one associated to $$\vec{v_g}$$. 

![vvis](/assets/img/blog/color-comparatives/vector_visualization_darklavender.png)  ![vvis](/assets/img/blog/color-comparatives/color_gradients_darklavender.png)

From the figures above, we can see that the model actually does a good job, as $$\vec{w_g}$$ follows 
quite close the direction of $$\vec{tr}$$. In terms of magnitude, $$\vec{wg}$$ is longer, which may say something
about the need to adjust the second term of the loss function. But in practical terms, if we look at the color
gradients, we can see that $$\vec{wg}$$ also takes us to a state that we could describe as  `dark lavender`. 

If we inspect other pairs, we can see the a similar behavior. 

For `(yellow, more orange, orange yellow)`, we see a quite close fit in terms of the direction, while at the same time a bigger difference 
in term of the magnitudes. This disagreement seems to not make much difference regarding the colors we can obtain. 
![vvis](/assets/img/blog/color-comparatives/vector_visualization_orangeyellow.png)  ![vvis](/assets/img/blog/color-comparatives/color_gradients_orangeyellow.png)


For `(mint, darker, dark mint)`:
![vvis](/assets/img/blog/color-comparatives/vector_visualization_darkmint.png)  ![vvis](/assets/img/blog/color-comparatives/color_gradients_darkmint.png)

In general,  the method works quite well for tuples on the test set whose elements have appeared during training, which means
RGB  tuples that are in the proximity of colors already seen. Naturally, things get more interesting when we pass a tuple in which  one or more components are totally out of distribution. 

For example, lets take a look at the tuple `(pale blue, more , very pale blue)`. In this case `pale blue` was not present 
on the training data. If we compute the color gradients using 
the grounded comparative vector, we obtain: 

 ![vvis](/assets/img/blog/color-comparatives/unseen_ref_color_gradients_verypaleblue.png)


We can see a clear difference on the ending color both gradients
reach, with the one associated to $$\vec{w_g}$$ getting closer to what we could consider *muddier* rather than *very pale*.  

Another setting we can explore is when the comparative was not seen during training. For example,  for the tuple `(pink, purplish, purplish pink)`, we obtain:

 ![vvis](/assets/img/blog/color-comparatives/unseen_comp_color_gradients_purplishpink.png)

Here we can see how the model is having a really hard time
trying to align we the ground truth. In fact,  we can see the 
ending colors do not resemble at all, and  in the case of
the color gradient associated to $$\vec{w_g}$$, there is no sign 
of  *purplishness* :) 

This tells us a lot about how important is for the model
to handle the comparative representations. As we reviewed above, in the original paper the vectors associated to the comparative
tokens are pretrained vectors from Google Word2vec. Of course replacing those vectors with other, more refined, pretrainied sources could allow more generalization, but I wonder if there is any way to consider out of vocabulary comparatives in a more comprehensive way.

One way to approach such problem could be to estimate the direction of an unseen comparative based on the semantic relationship it may  have with existing comparatives. For example, `brighter` and `lighter` , while not pointing in the exact same direction, probably are close enough. Therefore, if one of them  not appear, probably their RGB points could 
make the model be aware of their relationship, guiding it towards a feasible region in the  color space. Unfortunately, such relationship is in directly encoded in the pretrained vectors so it might be necessary to induce such bias. Besides that, I think the most interesting extension could be 
to incorporate structure  (hierarchies ? ) to the comparative relationships, maybe based on an external source. But lets keep
that as a future work for the reader.  

All the code, data generation and  figures can be found here:


> [http://github.com/solargrammars/comparative_color_descriptions](http://github.com/solargrammars/comparative_color_descriptions)
