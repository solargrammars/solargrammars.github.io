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

One specific element that has attracted my attention is how can we track and understand users learning ability. This is of high relevance , as it touches the core of the platform and impacts directly on the success of the initiative: If we cannot understand users learning ability and design or adapt tools to optimize their performance, the degree of engagement will drop naturally. One way to explore this problem is to analyze the students solutions to the assignments. The online learning initiative  `One Hour Code` <https://code.org/research>, which tries to teach programming skills, has a public dataset consisting on the sequences of partial solutions users submit in order to complete a programming assignment. 

There are two main papers associated to this dataset. The first one , [Autonomously Generating Hints by Inferring Problem Solving Policies](https://stanford.edu/~cpiech/bio/papers/inferringProblemSolvingPolicies.pdf) by Piech et. al., where the dataset is presented, along with a set of heuristics for hint recommendation, and [Learning to Represent Student Knowledge on Programming Exercises Using Deep Learning](http://educationaldatamining.org/EDM2017/proc_files/papers/paper_129.pdf), by Wang et. al., where a recurrent model is used to characterize the student trajectories by learning feature representations of the partial submissions.

The richness  of the data from Hour of Code resides in that, for each users that attempts to solve an assignment, a version of the current work is saved automatically on the system every time the user perform a change. Therefore, we are  able to capture the sequences of steps that a user followed from the beginning  until she reached a final solution. 

The dataset comprises user activity  from December 2013 to March 2014,  conformed by more than 137 million partial solutions for two defined coding problems, [HOC4](https://studio.code.org/hoc/4) and  [HOC18](https://studio.code.org/hoc/18).  `HOC4` requires the users to string together a series of moves and  turns, which is a problem of medium difficulty. `HOC18` required users to efficiently make use of a if-else condition inside a loop, which we can consider a more challenging problem.


Given the restrictions imposed by the platform, a student can produce a finite set of possible states (partial solutions). The dataset provides the set of all possible states and the legal transitions between them. That can be transformed into a graph, for example, here is the corresponding for HOC4:

![](/assets/img/blog/repair-policies/hoc4.png) 

As each state is basically a partial solution, it is actually a program. The dataset provides the actual Abstract Syntax Trees (AST)
associated to the  submissions from real participants. For each problem , we can obtain the list of ASTs with their associated frequency. For example, in the case of HOC4, the AST associated to the id 3 is

```
{'id': '7',
 'children': [{'id': '6',
   'children': [{'id': '0', 'type': 'maze_moveForward'},
    {'id': '2',
     'children': [{'id': '1', 'type': 'turnLeft'}],
     'type': 'maze_turn'},
    {'id': '4',
     'children': [{'id': '3', 'type': 'turnRight'}],
     'type': 'maze_turn'},
    {'id': '5', 'type': 'maze_moveForward'}],
   'type': 'statementList'}],
 'type': 'program'}
```
We can clearly see the hierarchy associated to the instructions this program contains. As HOC4 is rather simple, there is no much complexity. If we visualize an AST from  HOC18 we can see more interesting things, such as the use of an if-else statement.

```
{'id': '9',
 'children': [{'id': '8',
   'children': [{'id': '7',
     'children': [{'id': '6',
       'children': [{'id': '0', 'type': 'isPathForward'},
        {'id': '3',
         'children': [{'id': '2',
           'children': [{'id': '1', 'type': 'turnRight'}],
           'type': 'maze_turn'}],
         'type': 'DO'},
        {'id': '5',
         'children': [{'id': '4', 'type': 'maze_moveForward'}],
         'type': 'ELSE'}],
       'type': 'maze_ifElse'}],
     'type': 'DO'}],
   'type': 'maze_forever'}],
 'type': 'program'}
```

In addition to the frequency for each partial submission, the dataset also provides the results of the running unit tests, which can give a notion of their correctness.

Another key element of the data are the trajectories each user follows when trying to solve the problems. For example, in the case
of HOC4, we can access around 55K  unique trajectories, which have been seen at least once during the period considered. 
Here is a  sample of ten trajectories :

```
{'trajectory': ['6', '17', '1', '59', '22', '22', '32', '0'], 'count': 1}
{'trajectory': ['5', '68', '3302'], 'count': 1}
{'trajectory': ['39', '3', '89', '2', '4', '1', '32', '39', '37', '32', '22', '37', '26', '19', '26', '37', '3', '31', '3', '0'], 'count': 1}
{'trajectory': ['1', '4', '2', '40', '105', '150', '451', '1322', '2342', '3', '22', '45', '138', '108', '284', '316', '2', '18', '2', '0'], 'count': 1}
{'trajectory': ['3', '276', '276', '1', '3', '1', '2', '0'], 'count': 1}
{'trajectory': ['1', '31', '22', '0'], 'count': 2}
{'trajectory': ['103', '8'], 'count': 10}
{'trajectory': ['23', '47', '28', '3', '77', '77', '0'], 'count': 1}
{'trajectory': ['125', '95'], 'count': 3}
{'trajectory': ['10', '20', '1', '2', '0'], 'count': 1}

```
As we can see, each trajectory is represented as a sequence of ASTs ids. The variable `count` expresses how many time
such trajectory has been seen. In terms of the length of the sequences, we can take a look at the following 
histogram (there are trajectories with more than 100 elements, but removed them from the diagram for visualization purposes). 

![](/assets/img/blog/repair-policies/hoc4-hist-truncated.png) 

In addition to the partial solution transitions, this dataset contains a goal standard for both problems that consists of a manually labeled, for each  partial state, which next state the user `should`  follow, based on the expert judgement. 

The interesting part of this dataset is that it contains additionally a `hint` for a given partial state contributed by experts. These hints are aimed at improving the task solving process when a user gets stuck. Moreover, as we can see the full set of trajectories that each user follow on each assignment, we can capture the transitions (code changes) that were most effective, which means, that led to reach a correct solution in the shortest way.

