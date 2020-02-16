---
layout: post
title: "Combining Word and Character level Representations from Color Descriptions"
author: "Solar Grammars"
categories: blog
tags: [blog]
image: arctic-1.jpg
color: "#D95B43"
---

We have explored the relationship between language and colors
from both word and character level perspectives, but in 
isolation. Then, the natural question that arises is what 
if subword structures play an important role in color 
grounding?  In other words, what if we combine character
and word level representations when modeling color-language
relationships?

When we reviewed the results of the experiments on color generation
from descriptions using a character 
level recurrent model, we found quite interesting 
examples where, as we progressed on the sequences of 
characters, the generated colors suddenly changed. That was 
the case of descriptions such as `Strawberry lipstick` (or 
something like that), where the color generation up the character
 `w` (from strawberry) was giving us a brown-ish color, naturally 
associated with the word `straw`, which was also present during training. 
Then, suddenly, as the model incorporated the remaining tokens 
sequentially, `b -> e -> r -> r...` the output color was 
gradually changing to something more reddish (associated to
strawberry). 

This led us to think, how does the grounding of a 
word or sentence can be understood in terms of the interplay
between its words and characters? For a given description, is it 
a specific character pattern that impact the most on the 
changes at color generation? or what are the common subword 
structures that command or condition more strongly the color 
generation? 

In order to answer those questions, we need a way to quantify 
the contribution of character and word level representations 
together. Lets say we have two ways to obtain a feature vector
for each word present in a description. The first one: we use a RNN  which learns a 
character based feature vector from the word, just like we did before.
The second one:
we directly query a source of pretrained vectors, such as Word2vec
or Glove. In this setting, which one contributes then most to minimize 
the loss, lets say, expressed as the MSE between the generated 
color and the ground truth? What type of combination is the optimal ?


Fortunately, there has been quite interesting research
on the interplay between character and  word level 
representations (besides just concatenating both vectors) 
One way to represent such relationship is 
through the concept of `gates`, such as in 
[Balazs et. al.](https://arxiv.org/pdf/1904.05584.pdf) 
or [Miyamoto et. al.](https://arxiv.org/abs/1606.01700), which 
in simple terms means to learn a parameter $g$ that we can use 
to weight the vector representations coming from both char 
and word levels:

$$w = w_{char} * g +  w_{word} * (1-g)$$ 

Here $$g$$ can be a scalar or a vector. Both cases work almost 
the same, in which the idea is to have a fully connected layer 
that takes as input the word level representation and  outputs 
a scalar or a vector. 

Lets conduct a couple of experiments to see if this way of 
modeling color descriptions allow us to obtain interesting 
insights. As always, lets start with the data. For the 
purpose of this study, we can reuse the data
data extracted from ColourLovers, which we have used previously. 
This source of data is quite interesting given the variability
and abstractness in the use of language. Additionally, while 
they tend to be short (usually no more than five tokens) their
composition is quite diverse, as we can see if we extract
POS patterns. 

![](/assets/img/blog/color-gates-img/pos_freqs.png) 

Lets obtain a sample from the top 5 groups of patterns with most 
descriptions. This is what we get:

`('ADJ', 'NOUN')`:
<table class="table_colors">
<tr> <td>dutch teal</td><td><div style='float:left;width:100px; height:20px;background:#1a6470;'>  </div>  </td>  </tr>
<tr> <td>hot pink</td><td><div style='float:left;width:100px; height:20px;background:#ff67b4;'>  </div>  </td>  </tr>
<tr> <td>mighty slate</td><td><div style='float:left;width:100px; height:20px;background:#556270;'>  </div>  </td>  </tr>
<tr> <td>certain frogs</td><td><div style='float:left;width:100px; height:20px;background:#c3ff68;'>  </div>  </td>  </tr>
<tr> <td>millennial blue</td><td><div style='float:left;width:100px; height:20px;background:#374bde;'>  </div>  </td>  </tr>
<tr> <td>black tulip</td><td><div style='float:left;width:100px; height:20px;background:#340943;'>  </div>  </td>  </tr>
<tr> <td>lipgloss boost</td><td><div style='float:left;width:100px; height:20px;background:#d7217e;'>  </div>  </td>  </tr>
<tr> <td>green tea</td><td><div style='float:left;width:100px; height:20px;background:#edc003;'>  </div>  </td>  </tr>
<tr> <td>cheery pink</td><td><div style='float:left;width:100px; height:20px;background:#fa3a7f;'>  </div>  </td>  </tr>
<tr> <td>juicy pink</td><td><div style='float:left;width:100px; height:20px;background:#dd037e;'>  </div>  </td>  </tr>
</table>

`('NOUN', 'NOUN')`:
<table class="table_colors">
<tr> <td>vanilla cream</td><td><div style='float:left;width:100px; height:20px;background:#fbe6da;'>  </div>  </td>  </tr>
<tr> <td>vitamin c</td><td><div style='float:left;width:100px; height:20px;background:#d26f36;'>  </div>  </td>  </tr>
<tr> <td>orange icing</td><td><div style='float:left;width:100px; height:20px;background:#ea3556;'>  </div>  </td>  </tr>
<tr> <td>barbie world</td><td><div style='float:left;width:100px; height:20px;background:#e20092;'>  </div>  </td>  </tr>
<tr> <td>sugar champagne</td><td><div style='float:left;width:100px; height:20px;background:#f9cdad;'>  </div>  </td>  </tr>
<tr> <td>metro blue</td><td><div style='float:left;width:100px; height:20px;background:#2c89e6;'>  </div>  </td>  </tr>
<tr> <td>brides blush</td><td><div style='float:left;width:100px; height:20px;background:#ea69ae;'>  </div>  </td>  </tr>
<tr> <td>moon warrior</td><td><div style='float:left;width:100px; height:20px;background:#51445f;'>  </div>  </td>  </tr>
<tr> <td>love affair</td><td><div style='float:left;width:100px; height:20px;background:#856cc9;'>  </div>  </td>  </tr>
<tr> <td>heirloom hydrangea</td><td><div style='float:left;width:100px; height:20px;background:#b954f2;'>  </div>  </td>  </tr>

</table>


`('VERB', 'NOUN')`:
<table class="table_colors">
<tr> <td>haunted milk</td><td><div style='float:left;width:100px; height:20px;background:#cdd7b6;'>  </div>  </td>  </tr>
<tr> <td>raspberry lemonade</td><td><div style='float:left;width:100px; height:20px;background:#d2c2f1;'>  </div>  </td>  </tr>
<tr> <td>minted peas</td><td><div style='float:left;width:100px; height:20px;background:#c8f0b2;'>  </div>  </td>  </tr>
<tr> <td>strawberry lipgloss</td><td><div style='float:left;width:100px; height:20px;background:#ce5d9b;'>  </div>  </td>  </tr>
<tr> <td>irish sea</td><td><div style='float:left;width:100px; height:20px;background:#13a445;'>  </div>  </td>  </tr>
<tr> <td>poached ivory</td><td><div style='float:left;width:100px; height:20px;background:#d8d8c0;'>  </div>  </td>  </tr>
<tr> <td>wedded passion</td><td><div style='float:left;width:100px; height:20px;background:#630947;'>  </div>  </td>  </tr>
<tr> <td>imagine roses</td><td><div style='float:left;width:100px; height:20px;background:#fe016b;'>  </div>  </td>  </tr>
<tr> <td>rosen maiden</td><td><div style='float:left;width:100px; height:20px;background:#ffa4a0;'>  </div>  </td>  </tr>
<tr> <td>powdered sugar</td><td><div style='float:left;width:100px; height:20px;background:#bddebe;'>  </div>  </td>  </tr>
</table>

Hmm it seems here  Spacy tagged as verbs things that are passive voice adjectives. It's fine, as descriptions are short, probably 
Spacy didn't have much context to work with when deciding the tag. 

`('ADJ', 'NOUN', 'NOUN')`:
<table class="table_colors">
<tr> <td>hawaiian lava fields</td><td><div style='float:left;width:100px; height:20px;background:#4f4e57;'>  </div>  </td>  </tr>
<tr> <td>new yolk city</td><td><div style='float:left;width:100px; height:20px;background:#ffa111;'>  </div>  </td>  </tr>
<tr> <td>white chocolate fill</td><td><div style='float:left;width:100px; height:20px;background:#f8ecc9;'>  </div>  </td>  </tr>
<tr> <td>unreal food pills</td><td><div style='float:left;width:100px; height:20px;background:#fa6900;'>  </div>  </td>  </tr>
<tr> <td>red velvet cake</td><td><div style='float:left;width:100px; height:20px;background:#6e2e37;'>  </div>  </td>  </tr>
<tr> <td>royal passion rc</td><td><div style='float:left;width:100px; height:20px;background:#4b1139;'>  </div>  </td>  </tr>
<tr> <td>pure sun tanning</td><td><div style='float:left;width:100px; height:20px;background:#eb053f;'>  </div>  </td>  </tr>
<tr> <td>soft lace pink</td><td><div style='float:left;width:100px; height:20px;background:#ffebeb;'>  </div>  </td>  </tr>
<tr> <td>sexy police officer</td><td><div style='float:left;width:100px; height:20px;background:#f7374b;'>  </div>  </td>  </tr>
<tr> <td>cold morning dew</td><td><div style='float:left;width:100px; height:20px;background:#c8ebfe;'>  </div>  </td>  </tr>

</table>

`('ADJ','ADJ','NOUN')`:
<table class="table_colors">
<tr> <td>parttime super girl</td><td><div style='float:left;width:100px; height:20px;background:#e45635;'>  </div>  </td>  </tr>
<tr> <td>expensive black wool</td><td><div style='float:left;width:100px; height:20px;background:#0f0a06;'>  </div>  </td>  </tr>
<tr> <td>white kitten nose</td><td><div style='float:left;width:100px; height:20px;background:#ffd0d4;'>  </div>  </td>  </tr>
<tr> <td>typical light grey</td><td><div style='float:left;width:100px; height:20px;background:#e7e7e7;'>  </div>  </td>  </tr>
<tr> <td>old fine paper</td><td><div style='float:left;width:100px; height:20px;background:#f0eee1;'>  </div>  </td>  </tr>
<tr> <td>rich red wedding</td><td><div style='float:left;width:100px; height:20px;background:#88090b;'>  </div>  </td>  </tr>
<tr> <td>hawaiian blue light</td><td><div style='float:left;width:100px; height:20px;background:#66aef3;'>  </div>  </td>  </tr>
<tr> <td>little blue eyes</td><td><div style='float:left;width:100px; height:20px;background:#486060;'>  </div>  </td>  </tr>
<tr> <td>vintage new york</td><td><div style='float:left;width:100px; height:20px;background:#90797e;'>  </div>  </td>  </tr>
<tr> <td>blue eyed girl</td><td><div style='float:left;width:100px; height:20px;background:#2b70bb;'>  </div>  </td>  </tr>

</table>

In general we can see a good correlation between what each description 
expresses and the colors it has associated. 

There are several instances where the adjective is quite vague, 
such as in the case of `haunted` in `haunted milk`. In those cases,
we can expect any model to have a hard time trying to figure
out how to learn the specific variation of the color. In that example,
the noun associated is `milk`, which we know has the color `white`
as the most probable reference. But then, how to turn a plain 
version of `white` in to `haunted milk`. Here is where it relies
my main hypothesis on the usefulness of the gating mechanism.
I expect models that are only character-based or only  word-based 
to perform poorly as it seems to be necessary  to consider 
a bit  more flexible representation of the description. 

For the `milk` part of the description, I think , yes, just the 
word level vector might be enough. But for making sense of what is
`haunted`, I think the model should take into account substructures such 
as `haunt`, which can be only incorporated via the character level 
representation. Then, 
as `haunt` can be more associated to `pale` or `ghostly` terms (as in 
`The haunted house`, or based on this interesting [list](https://www.thesaurus.com/browse/haunted) ), 
the model could learn to associate the colors from other
ghost-centric descriptions present in the data (thats important 
to highlight, I assume that `haunted` in the sense ghost related stuff, 
but it could be the case the given the data, we are referring to 
another type of connotation. Anyway, [here](https://dictionary.cambridge.org/dictionary/english/haunt) 
is a good definition of what `haunt` means).

In summary, I think a gating mechanism could allow the model
to `learn to choose` from which source (character level or word level)
when it needs to learn to  generate a color from a description. Moreover, 
the gates does not provide a binary value, but
a weight we can later inspect to give to obtain more explanatory insights. 


In terms of how to operationalize a gating mechanism, we can follow
the standard approach from the Balazs et. al. paper, as seen in the following diagram:
![](/assets/img/blog/color-gates-img/gating-model.png) 

Here, we have the description `vanilla cream`. The ultimate
goal is to obtain a vector representation for the whole
description that can be fed into a linear layer to generate the 
color tuple (as a RGB or Lab color vector). We can do that by processing  each word at a time and 
then aggregate the word level features vectors. That can be done 
summing or averaging them, or, if we care about the ordering and
dependency between tokens (we should), we could use a word level
RNN over the word-level representations and use the final hidden vector
as a description-level vector.

But the core part is how to obtain a word-level representation. Here 
is where the gating mechanism comes to play. For each word in the description, 
we follow previous work and run two simultaneous processes. The first one
consists of obtaining a character-level feature vector of the word by means
of a LSTM that runs over the sequence of characters. This representation accounts
for the morphology aspects of the word and allow us to give more 
flexibility to the final representation. 

The second process is to query
a lookup table and obtain a pretrained feature vector associated to the word,
for example Glove. This vector is used to compute the gate weight $$g$$ 
by means of being fed directly to a linear layer. It is not direct that a 
pretrained source such as Glove could benefit this specific task. Given 
the nature of color description, and how words are associated in this context,
it could be the case that we could be actually sabotaging ourselves. 
In color descriptions, nouns serve usually as pivot from where an inherent color
is referenced. For example, when we describe a color as "sad banana" we know 
we are referring to a type of yellow. Same with something than contains nouns
such  as sun, school bus, vanilla, etc. 

Glove vectors were obtained through a task that is different to 
color description grounding, which is basically training a language model, where
we try to maximize the likelihood of a word given a defined context. In that sense,
if we assume two yellow things, such as `banana` and `taxi`, their vectors in Glove
will probably be quite distant, as those words do not co-occur so frequently. In 
Glove, the closest vectors to `banana` are probably are associated to other fruits, 
such as apple or orange, and the closest vectors to `taxi` will be probably 
associated to other transportation mediums. 
Glove is not color aware, or more generally, pretrained 
vectors are not attribute-aware. That does not mean we cannot use then as 
an initial representation and condition them based on additional information, such
as, for example, WordNet, or set them as trainable, which will naturally 
modify the spatial position. That could be an interesting experiment: to check how 
much the task changes de vectors we associate, in advance, to a defined color. 

Ok, having such model ready, and the data we discussed above, we can start our main
experiment. Lets consider the following models:


- Character-based LSTM (char-only)
- Combine Character-level LSTM  and Word-level by concatenation
  - word level vectors initialized randomly (concat-random)
  - word level vectors initialized with Glove (concat-glove)
- Combine Character-level LSTM  and Word-level by gating
  - word level vectors initialized randomly (gating-random)   
  - word level vectors initialized with Glove (gating-glove)


Let's take a look at the results from this five models and see 
what interesting stuff we can get.

Something important to notice is that models that combine
representations tend to be harder to train than just character 
level based model. In, in the exploratory runs, it was
kind of  difficult to find  a configuration that make 
the training stable. The most promising optimizer across
models was Adam, with a very low initial learning  of 0.0001. 
Overfitting was a huge issue, therefore using decay was essential
(more important that Dropout I would say )  to let the training and 
dev loss to achieve  a quasi-stable state.  

Regarding the use of pretrained vectors, does Glove help? 
Contrary to my initial assumption, it kinda help XD:  

![](/assets/img/blog/color-gates-img/loss1clean.png) 

The above graph says a lot. In the first place we can see that 
when we use glove, in both glove and random models we can achieve
a lower dev loss. The difference is not that big but it is considerable
specially as both models reached a stable state in terms of their
training loss. The second interesting thing here is  
that doe this dataset, basically gating does not give any 
advantage over the simple concatenation of word representations 
when we use glove, and when we use random vectors, clearly concatenation
beats gating by a respectable margin.  

Now, if we take the two best performing models and compare against
a char-only model, we can see a clear advantage:


![](/assets/img/blog/color-gates-img/loss2clean.png) 

The char-only model needs to learn from  scratch more fine-grained
relationships, therefore it is natural for it to be slower. Having 
access to a word level representation to encapsule characters as 
cohesive entities, seems to really speed up the training, and allows
to model to achieve a lower dev loss.


Having visualized the losses, let's take a look a the actual 
generated colors for a sample from  the unseen dataset.  

<table class="table_colors" ><tr><th>Descripton</th><th>gating random</th><th>gating glove</th><th>concat random</th><th>concat glove</th><th>only char</th><th> Ground truth</th></tr><tr><td>bat ears</td><td> <div style='width:60px; height:30px;background:#5b484a'> </div> </td><td> <div style='width:60px; height:30px;background:#794946'> </div> </td><td> <div style='width:60px; height:30px;background:#4f3f52'> </div> </td><td> <div style='width:60px; height:30px;background:#7e4945'> </div> </td><td> <div style='width:60px; height:30px;background:#989782'> </div> </td><td> <div style='width:60px; height:30px;background:#8b7c83'> </div> </td></tr><tr><td>sequoia glitter</td><td> <div style='width:60px; height:30px;background:#90736a'> </div> </td><td> <div style='width:60px; height:30px;background:#8d6169'> </div> </td><td> <div style='width:60px; height:30px;background:#92776c'> </div> </td><td> <div style='width:60px; height:30px;background:#936362'> </div> </td><td> <div style='width:60px; height:30px;background:#856a7a'> </div> </td><td> <div style='width:60px; height:30px;background:#c46411'> </div> </td></tr><tr><td>romantic music</td><td> <div style='width:60px; height:30px;background:#a47868'> </div> </td><td> <div style='width:60px; height:30px;background:#a7667a'> </div> </td><td> <div style='width:60px; height:30px;background:#b0637a'> </div> </td><td> <div style='width:60px; height:30px;background:#ab687c'> </div> </td><td> <div style='width:60px; height:30px;background:#927881'> </div> </td><td> <div style='width:60px; height:30px;background:#bbb9df'> </div> </td></tr><tr><td>kitty espionage</td><td> <div style='width:60px; height:30px;background:#ac8372'> </div> </td><td> <div style='width:60px; height:30px;background:#958288'> </div> </td><td> <div style='width:60px; height:30px;background:#b7926a'> </div> </td><td> <div style='width:60px; height:30px;background:#958184'> </div> </td><td> <div style='width:60px; height:30px;background:#8f787a'> </div> </td><td> <div style='width:60px; height:30px;background:#fac162'> </div> </td></tr><tr><td>first peach</td><td> <div style='width:60px; height:30px;background:#eea480'> </div> </td><td> <div style='width:60px; height:30px;background:#f7a076'> </div> </td><td> <div style='width:60px; height:30px;background:#f0a57f'> </div> </td><td> <div style='width:60px; height:30px;background:#f6a67c'> </div> </td><td> <div style='width:60px; height:30px;background:#e3ab7f'> </div> </td><td> <div style='width:60px; height:30px;background:#eabc9d'> </div> </td></tr><tr><td>sunny eyes</td><td> <div style='width:60px; height:30px;background:#b69582'> </div> </td><td> <div style='width:60px; height:30px;background:#b5b36d'> </div> </td><td> <div style='width:60px; height:30px;background:#b0b66c'> </div> </td><td> <div style='width:60px; height:30px;background:#aab16e'> </div> </td><td> <div style='width:60px; height:30px;background:#b2a271'> </div> </td><td> <div style='width:60px; height:30px;background:#ff0579'> </div> </td></tr><tr><td>black diamond eyes</td><td> <div style='width:60px; height:30px;background:#434b59'> </div> </td><td> <div style='width:60px; height:30px;background:#4a4c68'> </div> </td><td> <div style='width:60px; height:30px;background:#364567'> </div> </td><td> <div style='width:60px; height:30px;background:#233b5c'> </div> </td><td> <div style='width:60px; height:30px;background:#4e554a'> </div> </td><td> <div style='width:60px; height:30px;background:#31161c'> </div> </td></tr><tr><td>empty canvas</td><td> <div style='width:60px; height:30px;background:#ab9e91'> </div> </td><td> <div style='width:60px; height:30px;background:#9aa292'> </div> </td><td> <div style='width:60px; height:30px;background:#b1ad90'> </div> </td><td> <div style='width:60px; height:30px;background:#98a79a'> </div> </td><td> <div style='width:60px; height:30px;background:#a47e71'> </div> </td><td> <div style='width:60px; height:30px;background:#cedd90'> </div> </td></tr><tr><td>tangerine fade</td><td> <div style='width:60px; height:30px;background:#ec7e55'> </div> </td><td> <div style='width:60px; height:30px;background:#ea904b'> </div> </td><td> <div style='width:60px; height:30px;background:#ed8f51'> </div> </td><td> <div style='width:60px; height:30px;background:#eb8b51'> </div> </td><td> <div style='width:60px; height:30px;background:#a28d76'> </div> </td><td> <div style='width:60px; height:30px;background:#efa04c'> </div> </td></tr><tr><td>witchy rendezvous</td><td> <div style='width:60px; height:30px;background:#785a57'> </div> </td><td> <div style='width:60px; height:30px;background:#64405c'> </div> </td><td> <div style='width:60px; height:30px;background:#825560'> </div> </td><td> <div style='width:60px; height:30px;background:#603255'> </div> </td><td> <div style='width:60px; height:30px;background:#b08876'> </div> </td><td> <div style='width:60px; height:30px;background:#762075'> </div> </td></tr></table>


Well, the above experiment led us with a weird feeling. 
I was expecting that a gating mechanism really improved
the color generation over just a simple concatenation
of the word representations. But let's not give up yet. 
Lets use a different scenario to 
to see if the gating mechanism can shine. 

In the previous experiment, for simplicity, we prepared the data
in a way that all words have an associated
pretrained vector (i.e., we discarded all the 
descriptions that contain at least one word
which do not appear in Glove). Thats a strong
assumption. In reality, in color description, as subjective as it is,
people can use any word they consider appropriate
to describe a color. Some words could be quite unique
or rare, tightly associated to the person's cultural 
background or experiences. Most likely those words
will not have a pretrained vector available. 

Thats an interesting scenario to assess the usefulness 
of the gating mechanism. When a word we use has not 
an associated pretrained vector, what we basically do 
is to assign a random vector...which can be quite useless
in most cases. 

In that sense, we can see a good opportunity
for the gating mechanism: if we are currently trying 
to obtain a vector representation for a word in a sentence,
lets say the adjective `breakable`
but such world does not have a pretrained vector associated, 
would it be great if the gating mechanism put more 
weight into the character level representation, which could
have possibly already learned good representations of words
that are associated to the target word, such as  `broken` or
`breaking` and their relationship with color generation.

In order to test this new hypothesis, we need a new dataset
that actually contains words without a proper pretrained
representation and even noisier color descriptions. Lets 
generate such dataset and inspect its characteristics.

Lets relax a little bit the descriptions we are allowing words
that do not appear initially in Glove, but still only
allowing words whose frequency is higher than two and also 
descriptions that are at least two word long. We can regard
this dataset as a bit noisier than the original one, but
not super noisy. If we repeat the experiment with
the same models, we obtain the following results. 

![](/assets/img/blog/color-gates-img/loss1medium.png) 
![](/assets/img/blog/color-gates-img/loss2medium.png) 

<table class="table_colors" ><tr><th>Descripton</th><th>gating random</th><th>gating glove</th><th>concat random</th><th>concat glove</th><th>only char</th><th> Ground truth</th></tr><tr><td>bat ears</td><td> <div style='width:60px; height:30px;background:#5b484a'> </div> </td><td> <div style='width:60px; height:30px;background:#794946'> </div> </td><td> <div style='width:60px; height:30px;background:#4f3f52'> </div> </td><td> <div style='width:60px; height:30px;background:#7e4945'> </div> </td><td> <div style='width:60px; height:30px;background:#989782'> </div> </td><td> <div style='width:60px; height:30px;background:#8b7c83'> </div> </td></tr><tr><td>sequoia glitter</td><td> <div style='width:60px; height:30px;background:#90736a'> </div> </td><td> <div style='width:60px; height:30px;background:#8d6169'> </div> </td><td> <div style='width:60px; height:30px;background:#92776c'> </div> </td><td> <div style='width:60px; height:30px;background:#936362'> </div> </td><td> <div style='width:60px; height:30px;background:#856a7a'> </div> </td><td> <div style='width:60px; height:30px;background:#c46411'> </div> </td></tr><tr><td>romantic music</td><td> <div style='width:60px; height:30px;background:#a47868'> </div> </td><td> <div style='width:60px; height:30px;background:#a7667a'> </div> </td><td> <div style='width:60px; height:30px;background:#b0637a'> </div> </td><td> <div style='width:60px; height:30px;background:#ab687c'> </div> </td><td> <div style='width:60px; height:30px;background:#927881'> </div> </td><td> <div style='width:60px; height:30px;background:#bbb9df'> </div> </td></tr><tr><td>kitty espionage</td><td> <div style='width:60px; height:30px;background:#ac8372'> </div> </td><td> <div style='width:60px; height:30px;background:#958288'> </div> </td><td> <div style='width:60px; height:30px;background:#b7926a'> </div> </td><td> <div style='width:60px; height:30px;background:#958184'> </div> </td><td> <div style='width:60px; height:30px;background:#8f787a'> </div> </td><td> <div style='width:60px; height:30px;background:#fac162'> </div> </td></tr><tr><td>first peach</td><td> <div style='width:60px; height:30px;background:#eea480'> </div> </td><td> <div style='width:60px; height:30px;background:#f7a076'> </div> </td><td> <div style='width:60px; height:30px;background:#f0a57f'> </div> </td><td> <div style='width:60px; height:30px;background:#f6a67c'> </div> </td><td> <div style='width:60px; height:30px;background:#e3ab7f'> </div> </td><td> <div style='width:60px; height:30px;background:#eabc9d'> </div> </td></tr><tr><td>sunny eyes</td><td> <div style='width:60px; height:30px;background:#b69582'> </div> </td><td> <div style='width:60px; height:30px;background:#b5b36d'> </div> </td><td> <div style='width:60px; height:30px;background:#b0b66c'> </div> </td><td> <div style='width:60px; height:30px;background:#aab16e'> </div> </td><td> <div style='width:60px; height:30px;background:#b2a271'> </div> </td><td> <div style='width:60px; height:30px;background:#ff0579'> </div> </td></tr><tr><td>black diamond eyes</td><td> <div style='width:60px; height:30px;background:#434b59'> </div> </td><td> <div style='width:60px; height:30px;background:#4a4c68'> </div> </td><td> <div style='width:60px; height:30px;background:#364567'> </div> </td><td> <div style='width:60px; height:30px;background:#233b5c'> </div> </td><td> <div style='width:60px; height:30px;background:#4e554a'> </div> </td><td> <div style='width:60px; height:30px;background:#31161c'> </div> </td></tr><tr><td>empty canvas</td><td> <div style='width:60px; height:30px;background:#ab9e91'> </div> </td><td> <div style='width:60px; height:30px;background:#9aa292'> </div> </td><td> <div style='width:60px; height:30px;background:#b1ad90'> </div> </td><td> <div style='width:60px; height:30px;background:#98a79a'> </div> </td><td> <div style='width:60px; height:30px;background:#a47e71'> </div> </td><td> <div style='width:60px; height:30px;background:#cedd90'> </div> </td></tr><tr><td>tangerine fade</td><td> <div style='width:60px; height:30px;background:#ec7e55'> </div> </td><td> <div style='width:60px; height:30px;background:#ea904b'> </div> </td><td> <div style='width:60px; height:30px;background:#ed8f51'> </div> </td><td> <div style='width:60px; height:30px;background:#eb8b51'> </div> </td><td> <div style='width:60px; height:30px;background:#a28d76'> </div> </td><td> <div style='width:60px; height:30px;background:#efa04c'> </div> </td></tr><tr><td>witchy rendezvous</td><td> <div style='width:60px; height:30px;background:#785a57'> </div> </td><td> <div style='width:60px; height:30px;background:#64405c'> </div> </td><td> <div style='width:60px; height:30px;background:#825560'> </div> </td><td> <div style='width:60px; height:30px;background:#603255'> </div> </td><td> <div style='width:60px; height:30px;background:#b08876'> </div> </td><td> <div style='width:60px; height:30px;background:#762075'> </div> </td></tr></table>

Hmmm, even with a noisier dataset, the results are quite
similar. No problem. Let's force the model to use the 
raw dataset: straight out of the Colourlovers crawler, 
without any special filtering besides removing alphanumerical
symbols.

Let us repeat the experiment with the same models, and now:
![](/assets/img/blog/color-gates-img/loss1noise.png) 
![](/assets/img/blog/color-gates-img/loss2noise.png) 

<table class="table_colors" ><tr><th>Descripton</th><th>gating random</th><th>gating glove</th><th>concat random</th><th>concat glove</th><th>only char</th><th> Ground truth</th></tr><tr><td>bat ears</td><td> <div style='width:60px; height:30px;background:#5b484a'> </div> </td><td> <div style='width:60px; height:30px;background:#794946'> </div> </td><td> <div style='width:60px; height:30px;background:#4f3f52'> </div> </td><td> <div style='width:60px; height:30px;background:#7e4945'> </div> </td><td> <div style='width:60px; height:30px;background:#989782'> </div> </td><td> <div style='width:60px; height:30px;background:#8b7c83'> </div> </td></tr><tr><td>sequoia glitter</td><td> <div style='width:60px; height:30px;background:#90736a'> </div> </td><td> <div style='width:60px; height:30px;background:#8d6169'> </div> </td><td> <div style='width:60px; height:30px;background:#92776c'> </div> </td><td> <div style='width:60px; height:30px;background:#936362'> </div> </td><td> <div style='width:60px; height:30px;background:#856a7a'> </div> </td><td> <div style='width:60px; height:30px;background:#c46411'> </div> </td></tr><tr><td>romantic music</td><td> <div style='width:60px; height:30px;background:#a47868'> </div> </td><td> <div style='width:60px; height:30px;background:#a7667a'> </div> </td><td> <div style='width:60px; height:30px;background:#b0637a'> </div> </td><td> <div style='width:60px; height:30px;background:#ab687c'> </div> </td><td> <div style='width:60px; height:30px;background:#927881'> </div> </td><td> <div style='width:60px; height:30px;background:#bbb9df'> </div> </td></tr><tr><td>kitty espionage</td><td> <div style='width:60px; height:30px;background:#ac8372'> </div> </td><td> <div style='width:60px; height:30px;background:#958288'> </div> </td><td> <div style='width:60px; height:30px;background:#b7926a'> </div> </td><td> <div style='width:60px; height:30px;background:#958184'> </div> </td><td> <div style='width:60px; height:30px;background:#8f787a'> </div> </td><td> <div style='width:60px; height:30px;background:#fac162'> </div> </td></tr><tr><td>first peach</td><td> <div style='width:60px; height:30px;background:#eea480'> </div> </td><td> <div style='width:60px; height:30px;background:#f7a076'> </div> </td><td> <div style='width:60px; height:30px;background:#f0a57f'> </div> </td><td> <div style='width:60px; height:30px;background:#f6a67c'> </div> </td><td> <div style='width:60px; height:30px;background:#e3ab7f'> </div> </td><td> <div style='width:60px; height:30px;background:#eabc9d'> </div> </td></tr><tr><td>sunny eyes</td><td> <div style='width:60px; height:30px;background:#b69582'> </div> </td><td> <div style='width:60px; height:30px;background:#b5b36d'> </div> </td><td> <div style='width:60px; height:30px;background:#b0b66c'> </div> </td><td> <div style='width:60px; height:30px;background:#aab16e'> </div> </td><td> <div style='width:60px; height:30px;background:#b2a271'> </div> </td><td> <div style='width:60px; height:30px;background:#ff0579'> </div> </td></tr><tr><td>black diamond eyes</td><td> <div style='width:60px; height:30px;background:#434b59'> </div> </td><td> <div style='width:60px; height:30px;background:#4a4c68'> </div> </td><td> <div style='width:60px; height:30px;background:#364567'> </div> </td><td> <div style='width:60px; height:30px;background:#233b5c'> </div> </td><td> <div style='width:60px; height:30px;background:#4e554a'> </div> </td><td> <div style='width:60px; height:30px;background:#31161c'> </div> </td></tr><tr><td>empty canvas</td><td> <div style='width:60px; height:30px;background:#ab9e91'> </div> </td><td> <div style='width:60px; height:30px;background:#9aa292'> </div> </td><td> <div style='width:60px; height:30px;background:#b1ad90'> </div> </td><td> <div style='width:60px; height:30px;background:#98a79a'> </div> </td><td> <div style='width:60px; height:30px;background:#a47e71'> </div> </td><td> <div style='width:60px; height:30px;background:#cedd90'> </div> </td></tr><tr><td>tangerine fade</td><td> <div style='width:60px; height:30px;background:#ec7e55'> </div> </td><td> <div style='width:60px; height:30px;background:#ea904b'> </div> </td><td> <div style='width:60px; height:30px;background:#ed8f51'> </div> </td><td> <div style='width:60px; height:30px;background:#eb8b51'> </div> </td><td> <div style='width:60px; height:30px;background:#a28d76'> </div> </td><td> <div style='width:60px; height:30px;background:#efa04c'> </div> </td></tr><tr><td>witchy rendezvous</td><td> <div style='width:60px; height:30px;background:#785a57'> </div> </td><td> <div style='width:60px; height:30px;background:#64405c'> </div> </td><td> <div style='width:60px; height:30px;background:#825560'> </div> </td><td> <div style='width:60px; height:30px;background:#603255'> </div> </td><td> <div style='width:60px; height:30px;background:#b08876'> </div> </td><td> <div style='width:60px; height:30px;background:#762075'> </div> </td></tr></table>

Here we can see that the gating model is able to reach
a lower test loss than the rest of alternatives.
But at the same time, we can see that the character-based
model is now able slightly outperform the other 
methods.


Well, as a result, after taking into consideration the experiments and
the different datasets,  we cannot directly conclude that a gating mechanism 
always provide a better 
performance than the rest of configurations. We are considering a generic 
architecture for the specific problem of color grounding. My guess is
that we should design a more ad-hoc mechanism that takes into consideration
the unique characteristics of the problem. In the next post, lets keep 
exploring this problem and testing  some alternatives. 
