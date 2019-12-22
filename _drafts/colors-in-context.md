---
layout: post
title: "Palettes and  graphs as color context"
author: "Solar Grammars"
categories: blog
tags: [blog]
image: arctic-1.jpg
color: "#C02942"
---

Colors are usually perceived in as part of a composition. It is rare that we can 
find them in isolation in nature. In that sense,  probably what colors evoke is 
tightly associated to the perception of a colorful `context`. In other words, how we perceive 
a certain type of red, does not depend only on such color, but also on the colors
that surround it. 

As we are interested in learning feature representations of colors, we could
try to come up with a setting where colors appear as a group and then find associations
and, hopefully, patterns of co-occurrence between them. 

How can we find groups of colors we can work on? On simple way is to
collect a large amount of RGB images and  from each of them extract a color palette. A color
palette basically compresses an image into the most representative colors. We could say 
it is the most primitive way to do compression. While a palette cannot
provide a representation of the semantics of the image, at least it can inform us about
the main colors used and their distribution. 

Lets assume we have a set of images from which we can extract a color palette of $$n$$ colors, $$p_i = \{ c_1, ..., c_n \}$$. 
With such dataset, we could conduct a simple experiment: We know that in natural language, 
we can assume that words that occur in the same contexts tend to have the similar meaning, i.e.,
*you can know a word by the company it keeps*, which is known as the distributional hypothesis. 
Can we transfer such idea to colors, using the palettes as context?  In other words, 
can  we make the same analogy, and  say that colors that tend to co-occur  share certain
semantics? For example, let's say we collect thousands of pictures of bathrooms. In most
of them `white`, `light blue` and `pink` will appear quite frequently, because there seems to be 
a common agreement on  how bathrooms look like, like a pattern in which such colors represent the
`cleanness` and `calmness` a bathroom should have. Same if we analyze pictures of forests, probably
`greens` and `browns` will be predominant, while `pinks` will be very unlikely to appear. Can 
we leverage such regularities to learn dense feature vectors for each color?

Lets start with a very simple approach, using very well known tools to see if we are heading
in a feasible direction. Firstly, we need some data.  For simplicity, lets use the validation set 
of the Microsoft COCO dataset. For each image, let's capture a color palette of 6 colors.

<img src="/assets/img/blog/colors-in-context-img/p1.jpg" width="400px">  

For the above image, let's compute a incremental palettes. As we can see, the colors are appended
in terms of their absolute proportion on the image.  


<img src="/assets/img/blog/colors-in-context-img/p21.png" style="height:50px;">   
<img src="/assets/img/blog/colors-in-context-img/p22.png" style="height:50px;">   
<img src="/assets/img/blog/colors-in-context-img/p23.png" style="height:50px;">   
<img src="/assets/img/blog/colors-in-context-img/p24.png" style="height:50px;">   
<img src="/assets/img/blog/colors-in-context-img/p25.png" style="height:50px;">   
<img src="/assets/img/blog/colors-in-context-img/p26.png" style="height:50px;">   

With this data, we can test a direct analogy from natural language BOW models.
Lets recall such formulation as , lets say we have a sentence $$s$$ composed
by $$n$$ words , $$s  = \{  w_1, w_2 , ... , w_n \}$$, then a CBOW approach
learn vectors $$v_i$$ for a word $$w_i$$ as maximizing the likelihood 
of $$w_i$$ given its neighborhood  $$\{ w_{i-c} , ... , w_{i+c} \}$$. 

One problem we face at this moment is that each exact color probably appear only once
in one palette in all the dataset.  In other words, while we can
have two red colors that  look quite similar, in reality their RGB
representation will be different and therefore a model will perceive
them as different. To solve this issue, we can discretize their representation by clustering them.

Then, for example, we can expect  all variations of `scarlet` color to be grouped on a single 
cluster. Therefore, on a palette that contains one of these scarlet variations, instead of 
using the actual color, we can use the centroid/medoid of the associated cluster. 

Lets see first if clustering colors makes sense. For simplicity we can take the RGB or LAB 
tuples as feature vectors an apply K-means. In order to get a
rough estimation of the optimal number of clusters, we can take a look at both 
 distortion ( the average of the squared distances from the cluster centers of the respective 
 clusters ) and   inertia  (the sum of the squared distances to the closest cluster center). For 
 a sample of the data, we get the following: 
![num_clusters_study](/assets/img/blog/colors-in-context-img/num_clusters_study2.png)  

Which roughly tells us a good number of clusters is around 25. This makes sense as we are working 
with very standard natural images from COCO, so the colors they present should be quite compact. 
This result was obtained using a sample from 4000 images from which palettes of six colors were 
extracted, conforming a group of around 22.000 total colors that we clustered using their RGB 
representation. Changing  the amount of images or using the LAB format as feature vector, did 
not change considerably  the results. 

Lets take a look at the clusters visually. With $$k$$ = 25, we can see something like this:

<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/0.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/1.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/2.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/3.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/4.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/5.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/6.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/7.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/8.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/9.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/10.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/11.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/12.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/13.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/14.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/15.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/16.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/17.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/18.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/19.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/20.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/21.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/22.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/23.png" style="float:left;">  
<img src="/assets/img/blog/colors-in-context-img/cluster_palettes/24.png">  

Here we have 20 random elements from each cluster. As we can see, 
the groups tend to make sense, as there is a clear distinction between them and also a inherent cohesion within each cluster. Of course we 
can refine and adapt the number of clusters  based on the requirements of any  downstream task, but let's use this result for the moment. 

With this, we can  associate each color with its cluster id so 
we can generate  the reduced palettes. Now, lets learn a vector representation from each element in the reduced
palettes, i.e., a dense vector for each cluster id. We want to exploit the co-occurrence of the colors in the palettes
making a direct analogy as how words co occur in a set of sentences. In that sense we can train the reduced palettes in
the same way sentences are used in CBOW: 

![diagram1](/assets/img/blog/colors-in-context-img/diagram2.png)  

As stated above, the idea is to learn a feature vector for each color $$c_i$$ as maximizing the likelihood 
of $$c_i$$ given its neighborhood  $$\{ c_{i-\lambda} , ... , c_{i+\lambda} \}$$:

$$P(c_i | \{ c_{i-\lambda} , ... , c_{i+\lambda} \} )$$
 

CBOW serves as a nice inspiration, as there is no dependency between the elements in the context window. 
In our setting,  we also don't want to enforce any ordering, as in the palette, we cannot assume the colors 
conform a sequence. Something interesting to notice is that when we obtain the original palettes, for each 
color we can obtain its proportion ( between 0 and 1) . This value
could be incorporated as a weight when we sum context vectors.  

One of the parameters that impacts the most on the performance of CBOW is actually the size of the context 
window. In NL, larger windows tend to favor more topical similarities, while shorter ones  privileges more 
functional similarities. In the context of this problem, we don't know in advance such effect, therefore it 
will be necessary to explore empirically if there is any difference.

For palettes of six colors, and considering 25 clusters,  moving the window context size from 2 to 5 really 
does not any noticeable impact. But when we start using  longer palettes, the differences are visible.
For example, here we show a small experiment with palettes of 20 colors and a number of clusters equal to 400. In 
the following figures, each learned representation is reduced to a two dimensional point space using TSNE . I
have associated to each point the original RGB color, so we can visualize the vectors in terms of their colorful nature. 


| | | |
|<img src="/assets/img/blog/colors-in-context-img/window_sizes/context_size_2.png">|<img src="/assets/img/blog/colors-in-context-img/window_sizes/context_size_3.png">|<img src="/assets/img/blog/colors-in-context-img/window_sizes/context_size_4.png">|
|<img src="/assets/img/blog/colors-in-context-img/window_sizes/context_size_5.png">|<img src="/assets/img/blog/colors-in-context-img/window_sizes/context_size_6.png">|<img src="/assets/img/blog/colors-in-context-img/window_sizes/context_size_7.png">|
|<img src="/assets/img/blog/colors-in-context-img/window_sizes/context_size_8.png">|<img src="/assets/img/blog/colors-in-context-img/window_sizes/context_size_9.png">|<img src="/assets/img/blog/colors-in-context-img/window_sizes/context_size_10.png">|


Based on the above, we can start question ourselves a bit and 
try to ask what would make a good context-informed color
representation? Is it ok if we obtain vectors that group 
similar colors  into well defined groups? In some sense yes, 
but for such thing probably just using the RGB or LAB 
representation of the colors is enough. I think while it is
important  that the learned representations allow us to relate
similar colors, at the same time  I find it equally important that
these representation also encode the closeness of the colors 
in the context of the images the come from. Going back 
to the example we talked before, if we analyze images
of landscapes of forests, we probably can see groups of greens (lets
say , representing trees , vegetation ) closer to groups
of light blues and whites on one side (the clouds and sky)
and also some groups of brown and grays (the ground). 
It is not entirely clear which configuration is better, but
from the results, I would probably choose something between 4 and 5
for context window. 


At this point, we have a vector representation the can 
be associated to the whole cluster. Thats fine, but what we really
want is a vector for each color.  What we can do is to assume
that the learned vector is actually associated to the cluster
centroid (or medoid), then extrapolate the vector associated to 
each color in the cluster in terms of its distance to the centroid.


![diagram1](/assets/img/blog/colors-in-context-img/diagram1.png)  

In the above figure, lets say for cluster $$k$$  we obtained a vector representation
$$v_k$$. Let $$c_i$$ be the medoid color and we arbitrarily associate $$v_k$$ to that point.
Then, we need to find a way to obtain a vector to all the other $c_j$ members of the cluster, taking
into account $$v_k$$ as reference. A natural way to proceed would be to use the distance  $$d_{ij}$$ between $$c_i$$ and
$$c_j$$ as a way to weight $$v_k$$, assuming that if the entities keep a certain relationship 
in the the RGB space, such relationship should be also present, to some extent,  in a learned 
feature space. Then, to produce a feature vector associated to $$c_j$$, such representation
will have to keep certain direct proportion such as:

$$dist(c_i, c_j) \propto dist(v_k, v_j)$$


Such  relationship probably is not linear as we are trying to relate  two dissimilar  feature spaces : the color one  (roughly assuming that RGB or LAB  spaces can be metric space ... which strictly speaking is not correct), and the one defined by the feature representations from the co occurrence in the palettes. Finding  a factor that generalizes well across examples probably is hard. One way to proceed is parametrize such relationship via a small neural network. In that sense, we take all the pairs (centroid, vector)
and train a network. Then, at inference time, we pass all the real colors from to obtain a dense representation.


Once we have obtained a vector representation for all colors present in our dataset, we can 
start visualizing them. The first thing we can do is to reduce the 300-dim vectors into two dimensions and plot them.  In the following figure, we can see each point to which we have associated its original color in the visualization. 

The first thing we can notice is the smooth transition across colors in most of the graph. Remember each point location is not
their RGB , but the learned dense vector. In that sense we can see that the greens seems to be quite close the 
certain variation of light brown/ orange, probably representing the co occurrence of landscapes. The same phenomena can be seen with the light blues and whites, whose closeness probably come from the disposition of those colors
in the sky. Interestingly, the fact that the blacks and browns appear in the center, means the that they tend to co occur with all other groups, which makes sense as in natural images darker colors are associated
with transversal elements such as shadows that appear across all images.

![color_vectors_2d](/assets/img/blog/colors-in-context-img/color_vectors_2d.png)  

![color_vectors_2d](/assets/img/blog/colors-in-context-img/3d.png)  

## Possible Improvements/ Optimizations

- *Single model:* Our model is currently split into two main blocks. The fact the we have to perform a clustering
to reduce the palettes, while can be seen as a limitation, it also allows us to have more control, like a real pipeline. What
we can do  is try to merge the steps into one single model, by means of working directly with the original colors
from the palettes, without having to apply any reduction. 

- *Vector arithmetics:* if we combine yellow and green in the RGB space,  we probably will obtain [some kind of blue](https://youtu.be/0fC1qSxpmKo)... most likely.  What if we combine the learned vectors associated to yellow and green? Will the resulting vector be similar to the learned vector of blue too?  
- *Segmenting COCO or using other sources of images:* COCO provides a set of very heterogeneous images: people, landscapes, sports etc. What if we run this pipeline on a subset of images that share the same semantics? Lets say, only on images of landscapes, or only images of restaurants. In those cases, for a given color, how the learned
representations obtained from different sets relate? 


## Incorporating Natural Language

So far, we have been considering only the images and their colors. But the COCO dataset
provides a set of captions for each image, a source of data that looks quite irresistible.
Then the question is, how can we incorporate such information? How the representations of the colors
could be `improved` by informing our model/pipeline about the caption associated to the images we
use to obtain the palettes. I don't know the answer yet, but lets explore a bit to see in which 
direction we should move.

Lets not make things difficult and keep working with the validation part of COCO. For each image, we 
can obtain one or more captions.  The from each image $$i \in I$$, we can obtain a 
pair $$(p_i, s_i)$$, where $$p_i = \{ c_1, ..., c_N \}$$ is  the associated palette of $$N$$ colors, 
and $$s_i = \{  w_1, ..., w_M \}$$ the caption expressed as a sequence of  words. For each 
of these captions we can use GLOVE (or any other source of pretrained vectors) to
obtain  a good dense representation for each word.

![diagram3](/assets/img/blog/colors-in-context-img/diagram3.png)  

Given this extra source of information, how could we use the co occurrence patterns associated to 
the words to support the context based representation learning of colors? We know there is a one 
to one relationship between the sentences and the palettes: 

![diagram4](/assets/img/blog/colors-in-context-img/diagram4.png)  

Hmmm it seems there is going to be necessary to obtain a representation of the sentence, $$s_i$$. That 
is not difficult, as we can just do a plain average of the word vectors in $$s_i$$, use something 
like [Doc2vec](http://proceedings.mlr.press/v32/le14.pdf) or if we care about the dependencies (we probably should), 
we can use an RNN to encode the sentence and  capture the last hidden state as a representation 
of the sentence.  In any case, lets assume that for each sentence $$s_i$$ we obtain a dense sentence vector $$sv_i$$. 

Lets start with a very simple experiment. For each pair $$(p_i, sv_i)$$ lets use kNN to obtain the $$k$$ 
closest pairs, based on the cosine similarity between sentence vectors 

$$p_i, sv_i    \rightarrow kNN_{sv_i} = \{  sv_1, sv_2 , ... , sv_K\}$$  

Each of these sentence vectors have in turn associated a palette. Therefore, we can , 
transitively, associate the palette $$p_i$$ with a set of $$K$$ palettes based on the 
underlying similarity of their associated sentences. 

![diagram6](/assets/img/blog/colors-in-context-img/diagram6.png)  




For example, for the following  instance:

`a man in mid air attempting to catch a frisbee`  
![pcq](/assets/img/blog/colors-in-context-img/palette-caption-q.png)  


the closest five pairs are:

`a man reaching out to catch a frisbee`  
![pcq](/assets/img/blog/colors-in-context-img/pc1.png)  
`a man jumps to catch a frisbee flying through the air`  
![pcq](/assets/img/blog/colors-in-context-img/pc2.png)  
`there is a dog in the air going to catch a frisbee`  
![pcq](/assets/img/blog/colors-in-context-img/pc3.png)  
`a man in a grassy field about to catch a frisbee`  
![pcq](/assets/img/blog/colors-in-context-img/pc4.png)  
`a man is in mid air doing a skateboard trick`  
![pcq](/assets/img/blog/colors-in-context-img/pc5.png)  


We can see that while simple, the sentence embedding is able 
to capture quite relevant semantic relationships between sentences. Of course,
it degrades quite quickly, but that a is characteristic of the data under 
study. 

More interesting is the relationship between the colors. At first glance
I cannot see any direct similarity at palette level on this example. Actually, it looks
pretty random. I think this is because for this sample, the 
core word is `frisbee`, which inherently does not have an associated color, right?
What if we sample the captions by a word that can be easily associated to
a color, for example **forest**, which I expected to be linked to greens.

For a given sample containing the **forest** word:


`a bench in the middle of a lush green forest`  
![pcq](/assets/img/blog/colors-in-context-img/pc6.png)  

We obtain the following five closest:


`a tree sitting in the middle of a lush green field`  
![pcq](/assets/img/blog/colors-in-context-img/pc7.png)  

`a road in the middle of a beautiful green forest`  
![pcq](/assets/img/blog/colors-in-context-img/pc8.png)  

`a large brown cow standing on the side of a lush green hill`  
![pcq](/assets/img/blog/colors-in-context-img/pc9.png)  

`a man and child standing on top of a lush green field`  
![pcq](/assets/img/blog/colors-in-context-img/pc10.png)  

`a photo of a bench in the middle of the beach`  
![pcq](/assets/img/blog/colors-in-context-img/pc11.png)  


While there is still quick degradation on the semantic 
similarity across sentences, at least we can see a bit
more cohesion at color level. 

This is getting more interesting. We could then  TODO

## More Structure through graphs

We have explored the feature learning  of colors by means of
capturing their co-occurrence using palettes. While palettes are 
easy to obtain, they do not encode any kind of structural information. 
In contrast, if we rely on graph we could have a richer setting
that could allow a more comprehensive learning. 

How could we extract a color graph from an image? Actually, there 
are families of techniques from image processing that compute an
intermediate graph between image segments, for example, when 
performing image compression.

Lets see some examples of color graphs obtained from both COCO dataset
samples as well from art pieces from the WikiArt website:

`COCO`: 


| | |
|:-------------------------:|:-------------------------:|
<img src="/assets/img/blog/colors-in-context-img/image_graphs/coco1.png"> | <img src="/assets/img/blog/colors-in-context-img/image_graphs/coco1_graph.png">| 
<img src="/assets/img/blog/colors-in-context-img/image_graphs/coco2.png"> | <img src="/assets/img/blog/colors-in-context-img/image_graphs/coco2_graph.png">|  
<img src="/assets/img/blog/colors-in-context-img/image_graphs/coco3.png"> | <img src="/assets/img/blog/colors-in-context-img/image_graphs/coco3_graph.png">|
<img src="/assets/img/blog/colors-in-context-img/image_graphs/coco4.png"> | <img src="/assets/img/blog/colors-in-context-img/image_graphs/coco4_graph.png">|
<img src="/assets/img/blog/colors-in-context-img/image_graphs/coco5.png"> | <img src="/assets/img/blog/colors-in-context-img/image_graphs/coco5_graph.png">|


`WIKIART`:

| | |
|:-------------------------:|:-------------------------:|
<img src="/assets/img/blog/colors-in-context-img/image_graphs/wiki1.png"> | <img src="/assets/img/blog/colors-in-context-img/image_graphs/wiki1_graph.png">| 
<img src="/assets/img/blog/colors-in-context-img/image_graphs/wiki2.png"> | <img src="/assets/img/blog/colors-in-context-img/image_graphs/wiki2_graph.png">|  
<img src="/assets/img/blog/colors-in-context-img/image_graphs/wiki3.png"> | <img src="/assets/img/blog/colors-in-context-img/image_graphs/wiki3_graph.png">|
<img src="/assets/img/blog/colors-in-context-img/image_graphs/wiki4.png"> | <img src="/assets/img/blog/colors-in-context-img/image_graphs/wiki4_graph.png">|
<img src="/assets/img/blog/colors-in-context-img/image_graphs/wiki5.png"> | <img src="/assets/img/blog/colors-in-context-img/image_graphs/wiki5_graph.png">|

We can see that the resulting graphs represent a good discretization 
of the images, as they seems to cover the color spectrum  quite well.
The nodes position are not aligned with the actual image 
composition, but that can be added. 

Having computed these graphs for a set of images, how can we learn
feature representations that encode the relationships between colors?
In other words, not only learn a dense vector for each node, but also 
for the edges that connect them. 

One easy way would be to reuse most of the code we wrote before and 
re-apply a CBOW-like approach by sampling random walks from the graph
This is the idea behind existing approaches such  as [Node2vec](https://arxiv.org/abs/1607.00653).

![](/assets/img/blog/colors-in-context-img/node2vec.png) 

One nice thing about this idea is that the random walks could allow us
to visualize the progression of colors in certain zones. But at the same time,
as we need to define in advance a window, that could be restricting 
long term color dependencies present on the images. 


Lets explore another alternative that is provided  by the family of 
graph neural networks. This family of models are based on a 
message passing schema, where we iteratively learn node representations
by propagating information through the edge structure. In that sense,
the representation for a given node at a given time is the result 
of the aggregation function on all its neighbor nodes. 

Recently, there has been an extensive list of publications, 
each of them proposing a specific variation in terms of the how the
representations are computed. A paper by [Gilmer et. al.](https://arxiv.org/abs/1704.01212) 
does a great job (basically showing 
that the myriads of papers that have appeared in the last years are ... basically
the same) by
summarizing all the literature and providing a description of the core set of 
functionalities underneath the learning process. In short, if we assume
an undirected graph  $$G$$, node features $$x_v$$ and edge features $$e_{vw}$$, 
we will have a message passing phase that runs for $$T$$ time steps, 
defined in terms of a message function $$M_t$$ and a vertex update function $$U_t$$. 
Hidden states $$h_v^t$$ at each node are updated based on messages $$m_v^{t+1}$$,
which in turn is defined as $$m_v^{t+1} = \sum_{w \in N(v)}M_t(h_v^t,h_w^t, e_{vw})$$
and with  $$h_v^{t+1} = U_t(h_v^t,m_v^{t+1})$$. 

A subsequent readout phase can compute feature vector for the whole graph, using readout, 
function $$R$$, $$\hat{y} = R(\{ h_v^T|v \in G  \})$$. 
Here, as you can image, $$M_t, U_t, R$$ are learnable functions.

In our case, we can easily use this framework in a semi supervised learning setting.
For each graph associated to each image, we can let the model know the labels (RGB values)
only from a portion of the nodes. Then, the goal is to infer the RGB vector
associated to each each node, including both labeled and  unlabeled.  The assumption
behind this is that the message passing mechanism will eventually be able to 
propagate information from the seen to the unseen nodes, taking into account 
the locality of the color composition expressed by the edge structure.

<img src="/assets/img/blog/colors-in-context-img/message_passing.png" width="200px">

In the above figure, we expect the unlabeled node, represented in white, iteratively
receives a color signal from its neighborhood in the form of a node vector, 
which can be used to learn the correct RGB tuple. 
