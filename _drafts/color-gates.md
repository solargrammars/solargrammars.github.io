---
layout: post
title: "Color Gates"
author: "Solar Grammars"
categories: blog
tags: [blog]
image: arctic-1.jpg
color: "#D95B43"
---

We have explored the relationship of language and  colors
from  word-level or character  level perspectives in isolation. But 
what if subword structures play an important role in color grounding?  
When we reviewed the results of the experiments on character level 
description to color model, we found quite interesting  examples where, 
as we progressed on the sequences of characters,  the generated colors 
suddenly changed. That was the case of descriptions such as `Strawberry Lipstick` 
(or something like that), where the generation up the character `w` was giving us 
a brown-ish color, naturally associated with the word `straw`, which was also 
present during training.  Then, suddenly, as the model incorporated the remaining 
tokens sequentially, `b -> e -> r -> r...` the output color was gradually 
changing to red. 

This naturally led us to think, how does the grounding of a word or sentence 
can be understood in terms of the interplay
between its words and tokens? For a given description, is it a specific 
character pattern that impact the most on the changes in color  generation?  
What are the common subword structured that command or condition more strongly 
the color generation? 

In order to answer those questions, we need a way to quantify the contribution of 
character and word level representations. Lets say we train a model which 
learns a character based feature vector that is also
combined with a word vector. Both vectors are combined 
and then used to generate an associated vector. 
Which one contributes more to minimize the loss, lets
say, expressed as the MSE between the generated color and the ground truth. 

Fortunately, there has been quite interesting research
on the interplay between character and  word level representations. One way to 
represent such relationship is through the concept of `gates`, which in simple 
terms means to learn a parameter $g$ that we can 
use to weight the the vector representations coming from both char  and word level:

$$w = w_{char} * g +  w_{word} * (1-g)$$ 

Here $$g$$ can be a scalar or a vector. Both
cases work almost the same, in which the idea is to have a fully connected layer that takes
as input the word level representation and  outputs a scalar or a vector. 

Lets conduct a couple of experiments to see if this way of modeling color descriptions, 
allow us to obtain interesting insights. As always, lets start with the data. For the 
purpose of this study, we can reuse the data
data extracted from ColourLovers, which we have used previously. 


![](/assets/img/blog/color-gates-img/losses.png) 