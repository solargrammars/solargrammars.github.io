---
layout: post
title: "Learning Color-Language Representations"
author: "Solar Grammars"
categories: blog
tags: [blog]
image: blog/learn-color-reps/gre_set10.png
color: "#F05343"
---

Grounded language learning tasks allow to characterize the relationships between words semantically (abstracting from their symbolic nature), and based 
on that, contribute to understanding the principles of language 
acquisition. In that sense, color description, or more generally,  the generation of color language, is a relevant aspect for  understanding the emergence of human expression. 

The use of colors as constructs to describe situations can be seen 
in the literature. Some uses are intriguing, for example  
William Gibson's Neuromancer begins with "*The sky above the 
port was the color of television, turned to a dead channel*". What 
exactly did the author want to express here? Of course, there is a 
common agreement on the  purpose of such a sentence as a way of 
setting the mood of a cyberpunk novel, but the perception that each 
reader has certainly varies given the situatedness of language.

Another example is H.P. Lovecraft, who constantly uses the concept of color (or the lack of it) to describe the universes where his solitary characters are transported to:

> A rather large congeries of iridescent, prolately spheroidal bubbles and a very much smaller polyhedron of **unknown colours** and rapidly shifting surface angles (The Dreams of the House of the Witch)

> The vast tomb, or temple, was an **anomalous color** — a nameless blue-violet shade (The Tree on the Hill)

> Light filtered down from asky of **no assignable colour** in baffling, contradictory directions, and played almost sentiently over what seemed to be a curved line of gigantic hieroglyphed pedestals more hexagonal than otherwise and surmounted by cloaked, ill-defined Shapes (Through the Gates of the Silver Key).

For Lovecraft,  the inability to identify or generate a meaningful description of color seems to be the unequivocal sign that something very bad, as usual, is going to happen. Like if being able to define a color meant for an individual to have control over the understanding of the reality.

Besides the literary connotations, from a machine learning point of view, there is a whole line of research dedicated to this problem, which combines natural language processing, psychology and, even art theory.  

In general, the main idea is to use a set of (description, color) pairs to train a model able to map between these modalities. The used models differ in their nature but the task remains almost the same.  For example, we can mention the work of [Kawakami et al.](https://aclweb.org/anthology/D16-1202) which uses a character-level recurrent architecture to map a color description to an associated color. This tackles the particular problem that word-level models usually face, which is that color descriptions tend to be short, limiting the expressiveness of the models. [Bhargava et al.](https://web.stanford.edu/class/cs224n/reports/2762012.pdf) propose an autoencoder where the latent feature representation is aimed to align with a three-dimensional color vector. [Moore et al.](https://aclweb.org/anthology/D16-1243) focuses on the compositionality of the color descriptions by combining a neural encoder with a Fourier-based color transformer. The [same author](https://transacl.org/ojs/index.php/tacl/article/view/1142) incorporates a context perspective by modeling the description generation in a speaker-listener setting and also has recently [published](https://arxiv.org/pdf/1803.03917.pdf) an approach that incorporates bilingual data. 

### From language to colors


Let's start analyzing  Kawakami et al. paper, replicating the results and discussing possible directions. 
The first element to consider is the data. In this case, the authors propose to extract color-descriptions pairs from Colourlovers. Fortunately, there is an [API](http://www.colourlovers.com/api) and even a [Python client](https://github.com/elbaschid/python-colourlovers) and the extraction can be done with something like this:

```python
from colourlovers import ColourLovers
cl = ColourLovers()
cont = []
for i in range( 0,1000000,  100):
  time.sleep(3) # take it easy
  col = cl.colors(num_results=100, result_offset=i)
  cont+=[(c.id, c.title, c.rgb.red, c.rgb.green, c.rgb.blue)
    for c in col]
```

After some hours, I was able to get around 800.000 pairs. I applied certain filters to remove duplicates and  characters that were very infrequent, leading to a set with the following characteristics:

![char](/assets/img/blog/learn-color-reps/1.png)


![len](/assets/img/blog/learn-color-reps/2.png)


The associated colors are transformed from RGB to Lab format, which allows us to arrange  in a 3-dimensional space. The spatial disposition of a sample is presented in the following animated graph:

```python
import matplotlib
from IPython.display import HTML

def animate(i):
    ax.view_init(0.5*a[i], a[i])

rgb, lab = zip(*colors[:5000]) # for a sample of 5000 colors
x,y,z = zip(*lab)
rgb = [ (1.0*i[0]/255, 1.0*i[1]/255, 1.0*i[2]/255) for i in rgb]
fig = plt.figure()
ax = Axes3D(fig)
a= range(0, 90) # initial angle
ax.scatter(x, y, z, facecolors=np.asarray(rgb))
ani = matplotlib.animation.FuncAnimation(fig, animate, frames=len(a) )
plt.close()
ani.save('lab_space_90_30fps.gif', writer='imagemagick', fps=30)
```

![3dlab](/assets/img/blog/learn-color-reps/lab_space_90_30fps.gif)

Regarding the model, it consists of two main blocks. The first one, a color description encoder, takes a description of a color in natural language and pass it through a character-level LSTM, from which the last hidden state, $$h \in \mathbb{R}^{300}$$  is used as a description learned feature vector. The second one, a feed-forward layer takes $$h$$ and outputs the associated color in Lab format as $$\hat{y} = \sigma(Wh+ b)$$, with $$W \in \mathbb{R}^{3 \times 300}$$ and  $$ b \in \mathbb{R}^3$$.
The model tries to minimize the mean squared error between the output color and the reference using backpropagation. While I tried several optimizers, Adam was the one with a more consistent performance on  the validation set, as seen in the following comparison:


![curves](/assets/img/blog/learn-color-reps/curves.png)

Initial results show coherence between the generated colors and their references. If we analyze some of the color transitions as the network passes through the descriptions, we can see how intermediate colors are also aligned with the incomplete descriptions. This shows the expressive power of the character level approach, as the generation is a sequential process at a more fine grained granularity. 

Now, let's take a look at  how  the hidden vectors  $$h$$ produced  by the recurrent encoder change as we incrementally add more characters. For example,
```
'c',
'co',
'com',
'comu',
'comua',
'comuan',
'comuanz',
'comuanza',
'comuanza ',
'comuanza g',
'comuanza gr',
'comuanza gre',
'comuanza gree',
'comuanza green'
```
or 

```
'K',
'Ki',
'Kit',
'Kitc',
'Kitch',
'Kitche',
'Kitchen'
```

Let's start with a more qualitative approach, by obtaining the  intermediate feature vectors for each description and visualizing them in a two-dimensional space. Hopefully, we will be able to identify certain trajectories in vector space and their relationship to the associated colors.  

For example, for the color with the description **_strawberry kiss_**, we can obtain the vector from all its subsequences and reduce them altogether via PCA to generate the following figure:

![sk](/assets/img/blog/learn-color-reps/strawberry_kiss.png)

The associated transitions are presented in the following animation, where we can see that as we progress on the addition of characters, the distance between the vectors begin to decrease. This makes sense as at the beginning there is not restriction imposed by the conditional probabilities, but as we progress, the context associated to the previously seen characters narrows
the space of possible next characters. 

![ska](/assets/img/blog/learn-color-reps/sk5.gif)

Interestingly, the colors associated to each sub-sequence also converges. One thing to notice is, for example, in the word **_strawberry_**, when we get to the subsequence **_straw_**,  the color produced is between _light brown_  and _pale yellow_, which is characteristic to a _straw_ from  an agricultural perspective. As we continue adding characters, we can see how the color moves to a more red-ish spectrum, as **_strawberry_** is formed.  The same type of behavior can be seen in most of the descriptions.


For **_Dijon mustard_** :

![mustard](/assets/img/blog/learn-color-reps/dijon_mustard_ii.png)

For **_burnt blueberry_** :

![burnt](/assets/img/blog/learn-color-reps/burnt_blueberry.png)

A more interesting experiment is to visualize how two or more descriptions interact as their respective characters are appended. In that sense, let us assume a subset of descriptions that share a certain substring. As the generation is conditioned by the relative position of the characters, we can study the cases where the substring appears i) at the beginning of the descriptions (e.g. as **_blue_** in **_blueberry_** and **_blue ocean_**),  ii) in the middle or iii) at the end.

For the  fist case, let us select the substring **_gre_** and sample ten descriptions from our test set the begin with such pattern: 

```
'gre',
'greased lips',
'greasy spoon',
'great',
'great dark spot',
'great fright',
'great green',
'great lover',
'great party favors',
'great wide tea'
```

In this case, the resulting visualization is the following. 
(It must be noted that for the case of  two or more descriptions, the images generated by PCA are not expressive enough,
as most of the points are overlapped. For these cases, t-SNE performs much better.)

![gre10](/assets/img/blog/learn-color-reps/gre_set10.png)

And the corresponding animation:

![gre10ani](/assets/img/blog/learn-color-reps/gre5.gif)

For the second case, if we select the substring **_ran_**, we obtain following sample set:

```
'Orange Creamsicle',
'arugula granita',
'orange scribbles',
'Light Amaranth',
'goranluppo',
'my random',
'orange celosia',
'Violet Fragrance',
'Pomegranate',
'red orange'
```

![gre10](/assets/img/blog/learn-color-reps/ran.png)

For the last case, if we sample descriptions that end in **_nge_**, we obtain:

```
'Fringe',
'Lounge',
'Orange',
'avenge',
'change',
'fringe',
'lounge',
'orange',
'sponge',
'tounge'
```

![end_nge](/assets/img/blog/learn-color-reps/end_nge.png)

![end_nge_ani](/assets/img/blog/learn-color-reps/end_nge_ani2.gif)

Visualizing the results, we can see the most interesting trajectories are generated when we define a substring  at the end, in the sense of obtaining trajectories that are more linear and encapsulated in certain zones of the vector space than in other cases.

If you want to train your own model, the source for transferring from color names to LAB format can be found [here](https://github.com/pabloloyola/name-color).


Anyway, in this post we analyzed  the problem of producing a color given  a specific description of it. There are several extensions we can think of, but that is left to the reader's imagination. 


### From colors to language


Now we will try to do the inverse task, which means, give a color, expressed as a three-dimensional vector, obtain its description in natural language. To do that, we will take as inspiration  the approach described in [Monroe et al.](https://arxiv.org/abs/1606.03821) and try to obtain a working  version in Pytorch. The current implementation is available here: https://github.com/pabloloyola/color-description 


The data we will use in the first place is the dataset from [Munroe](https://blog.xkcd.com/2010/05/03/color-survey-results/), which are the results of a survey where participants were asked to provide a short description of a given color. 
The actual version of the dataset is the one processed by [McMahan et al.](https://aclweb.org/anthology/Q15-1008) and that is available for download [here](http://paul.rutgers.edu/~bcm84/rugstk_v1.0.tar.gz)

The original representation of the colors is the HSV format, which stands for Hue, Saturation and Value. This format is intended to more closely resemble how humans perceive color attributes. Here is a small sample from the training set:

![munroe-data](/assets/img/blog/learn-color-reps/munroe-data.png)

Given the nature of the data collection, there are cases where one color description is associated to several --very similar-- color vectors, and also cases  where one color vector is related to more than one color description.

As described in the original paper, the model consists of an LSTM decoder that receives as input a color in the HSV format and the associated description, as a sequence of words.

At each time step the LSTM is fed with a vector resulting from concatenating the vector representation of the color  we want to model, and the embedding vector associated to the word predicted by the model in the previous step. The output state from the LSTM is fed into a softmax layer from which the next word is selected. 

One interesting element to consider is  how to treat the color representation. We can just pass the raw three-dimensional vector as it is, assuming the the HSV space is sufficient to provide expressive representations. Another alternative is presented in the original paper, where it is proposed to transform the HSV vectors into a Fourier basis representation, resulting in  a 54-dimensional vector. 

An initial parameter search led to feasible results. Probably because the vocabulary size is not so big (as well as the sequences not so long). This is a random list showing a subset of the test set. The first column is the actual generated color description while the second and  third column represents that reference, in terms of the description and the color in RGB format. 

![atomic](/assets/img/blog/learn-color-reps/atomic.png)

As we can see, the model tends to output descriptions that while do not match exactly with the reference, they clearly resemble the target color. It is interesting to see the small perceptual differences between `orange` and `salmon` and  `pink` and `magenta`. 
One thing to notice in this case, is that the color descriptions are mostly formed by one word (maybe we should just call them color names in that case). This is a natural consequence of the nature of the dataset, where people were asked to be specific in their descriptions. 

If we filter a bit and collect results  where the generated sequences have length more than one (ignoring  `pad`, `bos` and `eos` special tokens), we can see something like this:


![not-atomic](/assets/img/blog/learn-color-reps/not-atomic-blue.png)



This a sample mostly from the `blue` spectrum, where can see two interesing things. Firstly, the accompanying words, usually adjectives, such as `dark` or `light`, are a bit repetitive, but effective. For example, in the third instance, we can see that the reference is just  `blue`, but if we look at the actual color, the generated description, `light blue` seems to fit better. Of course,  we do not have a quantitative way to measure this as we do not have a point of reference of what is actually `blue`, but the model seems to learn globally such characteristics.

If we filter a  bit more and consider only the generated descriptions and the references have more than one token, we obtain the following list. 


![2-2](/assets/img/blog/learn-color-reps/2-2-green-blue.png)

In this case, we see a more uniform matching between generated and reference descriptions. 

One interesting aspect to study is the impact of the Fourier transformation of the color vectors. The following graph compares the training and testing losses, showing that in fact expanding to a 54 dimensional representation improves model performance.

![losses_color_desc](/assets/img/blog/learn-color-reps/losses_color_desc.png)

Having explored the word-level approach, the natural next step is to
study if we can go a more fine grained level, for example, study if we can generate description character by character.
While this could represent an extra challenge for the model, I think it could also open the door to obtain more 
diverse descriptions. Let's see. 

The most straightforward way to obtain a character level approach is to reuse the exsiting code and just change the tokenizer function to split the description into a sequence of characters. 

When I  was playing with the hyperparameters, something that caught my attention was how to structure the input vector for the decoder. As we saw above, this vector is the concatenation of the color representation (raw three-dimensional vector or an expansion based on the Fourier mapping) and the embbeded vector of the word predicted in the previous step.  For the word-based level, changing the proportion of the two components did not impact on the results. Something different I experienced in the character-level method. For example, the following figure compares the training and testing losses between three runs, using vectors of 50, 100 and 200 respectively. Again, the first column represents the generated description, and the second and third , the references. 

![losses_embedding_size_character_level](/assets/img/blog/learn-color-reps/losses_embedding_size_character_level.png)

Such difference also impact on the generated descriptions, for example:

Sample of generated descriptions with embedded vector of size 50:

![size50](/assets/img/blog/learn-color-reps/size50.png)

Sample of generated descriptions with embedded vector of size 100:

![size100](/assets/img/blog/learn-color-reps/size100.png)

Sample of generated descriptions with embedded vector of size 200:

![size200](/assets/img/blog/learn-color-reps/size200.png)


As we can see, the size of the character embedding indeed impacts on the generation results. A vector of size 50 seems to generate incomplete descriptions, meaning the it is not expressive enough to guide the generation to known descriptions.  A size of 200 totally makes the model collapse, generating the same pattern for the majority of the instances on the test set. Finally, a size of 100 seems to  provide much reliable results, in this case we can see the both the signal from the color and the embedded vector play nicely together, such as in the cases of `red` for `orange-brown` and `brown`  for  `khaki`, which means, while the model cannot always produce  the exact description, at least it is able to use in a effective way the color representation to find a feasible alternative.


All the code and data generation scripts can be found here:

>  [https://github.com/solargrammars/name_color](https://github.com/solargrammars/name_color)


> [https://github.com/solargrammars/color_descriptions](https://github.com/solargrammars/color_descriptions)

