---
layout: article
title: "AI Papers #3: Double descent"
sidebar:
  nav: docs-en
aside:
  toc: true
show_tags: true
tags: AI papers deep-learning
---
Where I read two papers about the *double descent* phenomenon, a significant challenge to our understanding of statistical learning

---

**Epistemic status**: moderate confidence, expect some misconceptions. 

<br />

# Overview
This week I read two papers about [double descent](https://openai.com/blog/deep-double-descent/), which is the phenomenon where with increasing training (or parameters, or data), test performance first improves, gets worse, then improves again.
1. *[Reconciling modern machine learning practice and the bias-variance trade-off](https://arxiv.org/pdf/1812.11118.pdf)* by Belkin *et al.* (2019)  
2. *[Grokking: Generalization Beyond Overfitting On Small Algorithmic Datasets](https://mathai-iclr.github.io/papers/papers/MATHAI_29_paper.pdf)* by Power *et al.* (2021)

# The bias-variance "trade-off"
A summary of the 2019 paper *[Reconciling modern machine learning practice and the bias-variance trade-off](https://arxiv.org/pdf/1812.11118.pdf)* by Belkin *et al.*

## Summary
In traditional statistical learning, we expect a machine learning model to trade off bias and variance, to prevent underfitting (high bias) and overfitting (high variance). During training, this suggests that we should aim for a "sweet spot' between these two regimes. 

But modern deep learning systems sometimes exhibit behaviours that aren't explained by this framework. For instance, some large neural networks, can have extremely high generalisation performance, even with many parameters and an almost perfect fit to the training set. This supports the modern deep learning approach of choosing really large nets, such that the network can **interpolate** the training data (i.e. effortlessly achieve zero loss on the training set). 

This happens through the phenomenon of double descent. If we increase the number of parameters, we first get a U-shaped curve describing the test error against the model size - characteristic of the bias-variance tradeoff. But this only holds up until we hit an "interpolation threshold", beyond which the test error decreases. 

One way we can interpret this is in terms of three cases: 
1. When the number of parameters is much smaller than the sample size, then the model underfits because the model is unable to capture important patterns in the data
2. When the number of parameters approaches the sample size (i.e. the interpolation threshold), features are *forced* to fit the training data, even if they are only weakly present, leading to overfitting (this happens for a narrow range of parameters)
3. Further increasing the number of parameters beyond the interpolation threshold results in there being a larger class of functions that can be used to fit the data, thus the model becomes better and better at finding good approximations to the data that can generalise

The third point suggests that the number of parameters alone doesn't necessarily tell you how well a model can be applied to a problem. We also need to know what *inductive biases* (i.e. assumptions about properties of the model) are appropriate - so if we consider a larger class of functions, we're more likely to get functions with inductive biases that are compatible with the problem at hand. To test this, the authors successfully demonstrated similar effects with neural networks and decision trees, where enlarging the function class caused generalisation to improve. 

But why haven't we heard about double descent a lot more? There are a few reasons: 
* Observing double descent needs functions of arbitrary complexity, but statistical learning typically deals with a small, fixed set of features
* Regularisation changes the effective capacity of a function class, so that the interpolation threshold becomes a lot less clear and thus masked out
* Optimisation problems can be highly non-convex, and the resulting sensitivity to inputs makes it hard to observe the interpolation threshold (which can only be observed within a range of parameters)
* Things like early stopping prevent us from observing the interpolation peak

Overall, it's possible there are fundamental difference in doing optimisation in the classical statistical regime (i.e. before reaching the interpolation threshold), and in the "over-parameterised regime". Who knows what they may be?

## Thoughts
This seems like a very plausible explanation to me. I'm confused about the how features get forced to fit the training data - does this mean that in some situations, where the parameters are chosen appropriately, then we definitely won't observe double descent?

I also don't really see how this argument (that increasing the number of parameters increases the space of possible functions) works for double descent due to increasing time or data. But I did find the argument itself reasonably compelling.  

# Grokking
A summary of the 2021 paper *[Grokking: Generalization Beyond Overfitting On Small Algorithmic Datasets](https://mathai-iclr.github.io/papers/papers/MATHAI_29_paper.pdf)* by Power *et al.* This paper builds upon previous work done on double descent, and suggests a novel approach for studying how neural networks generalise. 

## Summary
This paper argues that we should use algorithmically generated datasets to study the generalisability of neural networks to different distributions. 

Why? Neural nets trained on these distributions reliably demonstrate some unusual generalisation patterns:
1. **Generalisation beyond overfitting**: the neural networks demonstrate **grokking** - when validation accuracy suddenly increases from chance level to perfect generalisation, *long after severely overfitting*. But why should a neural net suddenly stop overfitting with more training?
2. **Learning time curves**: decreasing the dataset size keeps the generalisation performance high, but the training time needed to reach this performance increases exponentially. In contrast, the *training* performance generally decreases with decreasing dataset size, and the optimisation time doesn't change much. But why doesn't the generalisation performance decrease with smaller dataset size?

These results seemed most prominent with certain types of regularisation, like weight decay (or $L_2$ regularisation).

In addition, the authors also find some interesting patterns. The algorithms generated were based on binary operations (for either the symmetric group $S_5$ or a ring $^{\mathbb{Z}}/_{n\mathbb{Z}}$), but with the group elements relabelled with abstract symbols. 
* Commutative operations required less data to learn, and the network was simply unable to generalise on some other operations. 
* Plotting the network's representations reveals as an embedding algebraic structures (e.g. the elements get clustered together if they are in the same coset)

The hope is that by studying these datasets, we can get a better understanding of how generalisation actually works - this is something that our current understanding of statistical learning and deep learning is unable to explain. 

## Thoughts
I found this paper quite interesting, especially the part about the algebraic structures emerging after doing the [t-SNE embedding](https://en.wikipedia.org/wiki/T-distributed_stochastic_neighbor_embedding). It also raises quite a lot of questions, without providing any answers! 

Here are some things that I don't understand: 
* Why do some operations appear effectively random to the model? 
* It makes sense that the elements of a cyclic group should form circular structures after applying an embedding, but why should it look like the way it did in the paper? How can I know *a priori* where each number might land under the transformation?
* How should I think of double descent in terms of gradient descent in some kind of loss landscape?