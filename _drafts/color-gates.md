---
layout: post
title: "Color Gates"
author: "Solar Grammars"
categories: blog
tags: [blog]
image: arctic-1.jpg
color: "#D95B43"
---

We have explored the relationship of language and colors
from both word character level perspectives, but in 
isolation. Then, the natural question that arises is what 
if subword structures play an important role in color 
grounding?  

When we reviewed the results of the experiments on character 
level description generation, we found quite interesting 
examples where, as we progressed on the sequences of 
characters, the generated colors suddenly changed. That was 
the case of descriptions such as `Strawberry Lipstick` (or 
something like that), where the generation up the character
 `w` was giving us a brown-ish color, naturally associated 
with the word `straw`, which was also present during training. 
Then, suddenly, as the model incorporated the remaining tokens 
sequentially, `b -> e -> r -> r...` the output color was 
gradually changing to red. 

This naturally led us to think, how does the grounding of a 
word or sentence can be understood in terms of the interplay
between its words and tokens? For a given description, is it 
a specific character pattern that impact the most on the 
changes on color  generation? What are the common subword 
structures that command or condition more strongly the color 
generation? 

In order to answer those questions, we need a way to quantify 
the contribution of character and word level representations 
... `together`. Lets say we train a model which learns a 
character based feature vector that is also combined with a 
word vector. Both vectors are combined and then used to generate 
an associated vector. Which one contributes more to minimize 
the loss, lets say, expressed as the MSE between the generated 
color and the ground truth. 

Fortunately, there has been quite interesting research
on the interplay between character and  word level 
representations. One way to represent such relationship is 
through the concept of `gates`, such as in [Balazs et al](https://arxiv.org/pdf/1904.05584.pdf) 
or [Miyamoto et al](https://arxiv.org/abs/1606.01700), which in simple terms means 
to learn a parameter $g$ that we can use to weight the the 
vector representations coming from both char  and word level:

$$w = w_{char} * g +  w_{word} * (1-g)$$ 

Here $$g$$ can be a scalar or a vector. Both cases work almost 
the same, in which the idea is to have a fully connected layer 
that takes as input the word level representation and  outputs 
a scalar or a vector. 

Lets conduct a couple of experiments to see if this way of 
modeling color descriptions, allow us to obtain interesting 
insights. As always, lets start with the data. For the 
purpose of this study, we can reuse the data
data extracted from ColourLovers, which we have used previously. 

One element we can consider is to condition the descriptions by 
certain characteristics such as length or even color. For the 
moment I think grouping them by 
the POS tag could give us a good indicator. This is a sample of the 
descriptions, without considering  any grouping.

TABLE

Now, for each description, let obtain their POS tags and study their
composition. In terms of patterns of POS tags within the
descriptions, we can see something like this:

![](/assets/img/blog/color-gates-img/pos_freqs.png) 

Lets obtain a sample from the top 5 groups of patterns with most 
descriptions. This is what we get:

`('ADJ', 'NOUN')`:
<table>
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
<table>
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
<table>
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


`('ADJ', 'NOUN', 'NOUN')`:
<table>
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
<table>
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
Interestingly, in the case of `(VERB, NOUN)`, it seems the parser
got a bit confused and most of the instances can be regarded as
part of the  `(ADJECTIVE, NOUN)`.


There are several instances where the adjective is quite vague, 
such as in the case of `haunted` in `haunted milk`. In those cases,
we can expect any model to have a hard time trying to figure
out how learn the specific variation of the color. In that example,
the noun associated is `milk`, which we know has the color `white`
as the most probable reference. But then, how to turn a plain 
version of `white` in to `haunted milk`. Here is where it relies
my main hypothesis on the usefulness of the gating mechanism.
I am expecting a model that is only character based word based 
to fail as I consider necessary for the model to play 
a bit with a more flexible representation of the description. 

For the `milk` part of the description, I think , yes, just the 
word level vector might be enough. But for making sense of what is
`haunted`, I think the model should take into account substructures such 
as `haunt`, which can be only incorporated via the character level 
representation. Then, I say that
as `haunt` can be more associated to pale or ghostly concepts (e.g. 
*The haunted house* or this interesting [list](https://www.thesaurus.com/browse/haunted), then
the model could learn to associate the colors in other
ghost-centric descriptions present in the data (thats important 
to highlight, I assume that haunted in the sense ghost related stuff, 
but it could be the case the given the data, we are referring to 
another type of connotation).

In summary, I think a gating mechanism could allow the model
to `learn to choose` from which source (character level or word level)
when it needs to generate a color from a description. Moreover, 
the gates can only help the model to choose one or another, but
as proportion, gracefully combining both representations. 



With this, lets conduct the following  experiment:

- Lets consider 4 models:
  - Just word embedding
  - combine char LSTM + Glove by simple contatenation 
  - combine char LSTM + Glove by gating



![](/assets/img/blog/color-gates-img/losses.png) 
