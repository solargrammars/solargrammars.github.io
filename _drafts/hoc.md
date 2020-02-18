---
layout: post
title: "Program Repair by Policy Learning (in progress)"
author: "Solar Grammars"
categories: blog
tags: [blog]
image: blog/learn-color-reps/gre_set10.png
color: "#751071"
---

Repairing a bug is essentially a search problem. At source code level, it is necessary to find the combination of structural and syntactic changes that lead us obtain a version of the code on which the reported fault is not present anymore. Therefore, the code needs to be transformed, from a defective state to a healthy one. We assume this transformation can be achieved by adding, modifying or even deleting certain portions of the code. Most existing techniques for repair take the problem only from the source code perspective, and therefore, only consider the syntactic aspects of the program. While this configuration appear as the most natural approach, I think we can explore a more top-down way, in the sense of explicitly consider how humans find their way to fix a bug.

In that sense, we can model the bug fixing dynamics as a sequence of progressive modifications that conform a trajectory across the space of feasible solutions followed by a human when trying to repair a program. The goal is to understand the transitions between code level changes that lead to reliable and effective solution.

Therefore, we firstly need to let a program `observe` how real faults are fixed by humans, by means of considering editing activity. After capturing this transitions, our model could be able to extract the distributional patterns necessary to carry out a new fix.

Instead of using supervised learning approach, where we assume the existence of a training set of the form $$(X,y)$$, where $$X$$ represents a set of features and $$y$$ a defined label associated to each instance, here we are more interested on obtaining sequential data, in the form of chain of consecutive changes from which we can learn how the trajectory of partial changes impact the overall solution.

In terms of how to model the problem, I think  that reinforcement learning fits well our requirement: instead of learning from a fixed mapping between the dependent variable feature representations and a label, the learning process in this case is cast as sequential decision making, where the learner understand its current state and choose the action that will most likely maximize the future rewards. Therefore, the first step is to obtain data that represents real transitions developers follow when they build or modify a program. To this end, we can rely on data from Massive Open Online Courses (MOOCs).

MOOCs represent one of the most interesting initiatives aimed at spreading high quality education content, with the goal of democratizing education. As the impact of that goal could be huge, the amount of complexity involved is also considerable, in both social and technical dimensions.

One the key elements that a massive online system needs to address is related to the diversity of students. Contrary to a standard teaching setting, where we can expect certain level of homogeneity of skills and cultural background, the openness and accessibility of massive online classes is characterized by the diversity of the participants. Given the low entry barriers, we can expect users coming from the most diverse cultural backgrounds, geographical locations, etc. While diversity represents a positive aspect, as it is in fact one of the desired elements these systems seek, at the same time it represents challenge, in terms of the volume and the variability of the content that students generate.

In terms of understanding participants, literature has focused on several aspects, such as motivations, incentives, among others, which are key to optimize the user experience and retention rate, similarly to the research in other collaborative systems such Open Source software development.

One specific element that has attracted my attention is how can we track and understand users learning ability. This is of high relevance , as it touches the core of the platform and impacts directly on the success of the initiative: If we cannot understand users learning ability and design or adapt tools to optimize their performance, the degree of engagement will drop naturally. One way to explore this problem is to analyze the students solutions to the assignments. The online learning initiative  `One Hour Code` <https://code.org/research>, which tries to teach programming skills, has a public dataset consisting on the sequences of partial solutions users submit in order to complete a programming assignment. The interesting part of this dataset is that it contains additionally a `hint` for a given partial state contributed by experts. These hints are aimed at improving the task solving process when a user gets stuck. Moreover, as we can see the full set of trajectories that each user follow on each assignment, we can capture the transitions (code changes) that were most effective, which means, that led to reach a correct solution in the shortest way.