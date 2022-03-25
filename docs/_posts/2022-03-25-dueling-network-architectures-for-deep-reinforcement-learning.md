---
layout: article
title: "Dueling Network Architectures for Deep Reinforcement Learning (Wang, 2015)"
sidebar:
  nav: docs-en
aside:
  toc: true
show_tags: true
tags: AI papers reinforcement-learning deep-learning
---

*[Link to paper by Wang et al.](https://arxiv.org/pdf/1507.06527.pdf)*

# Motivation
Thus far (2015), most approaches to RL use standard neural networks like LSTMs. Why not take a different approach that is more suited to RL methods?

One observation we might make is that in many states, estimating the value of each action choice is unnecessary.

The authors give the example of Enduro - here moving left or right typically doesn't matter, *unless a collision is eminent*.

Situations with bootstrapping are different, where estimating state values is very important for every state. 

## Key idea
The dueling architecture thus learns which states are valuable, without needing to learn the effect of each action for each state. 

This is done by getting a standard DQN and splitting it into two streams of fully connected layers after a single sequence of convolutional layers, to help separately estimate the value and advantage functions. These two functions are combined to give $Q$ values. 

# Approach
## Obtaining $Q$ estimates
How do we implement this dueling network module? If we have convolutional layer parameters $\theta$, and parameters $\alpha$ and $\beta$ for the two streams of fully-connected layers, then we might try to construct the $Q$ function as follows:

$$Q(s, a; \theta, \alpha, \beta) = V(s; \theta, \beta) + A(s, a; \theta, \alpha)$$

But this is problematic for two reasons: 
- The parameterisations of $A$ and $V$ may not be good estimates of the true functions, so the resulting $Q$ value may be inaccurate
- Given $Q$, we cannot uniquely recover $V$ and $A$, which leads to poor practical performance (i.e. the equation is *unidentifiable*)

Thus the authors suggest two possible adjustments to equation (1)[^1]:
1. Force the advantage function estimator to be zero at the *optimal action*[^2]
$$Q(s, a; \theta, \alpha, \beta) = V(s; \theta, \beta) + (A(s, a; \theta, \alpha) - \max_{a' \in \mathcal{A}} A(s, a'; \theta, \alpha))$$
2. Replace the $\max$ with an average over actions
$$Q(s, a; \theta, \alpha, \beta) = V(s; \theta, \beta) + (A(s, a; \theta, \alpha) - \frac{1}{\mid \mathcal{A} \mid} \sum_{a' \in \mathcal{A}} A(s, a'; \theta, \alpha))$$

Both of these help with identifiability, and make it clear that one stream is estimating the value function and the other stream learns the advantage function. 

Neither of these equation change the relative rank of $A$ or $Q$ values, and so any ($\epsilon$-)greedy policy gets preserved relative to equation (1). 

# Experiments
The authors evaluate performance of the dueling architecture by doing policy evaluation, following an $\epsilon$-greedy behaviour policy.

They then test the architecture on Atari games, and compare their results with DQN and DDQN approaches. 

Findings: 
- The dueling network does substantially better than an equivalent single stream network, as well as the previous SoTA
- The dueling network achieves human-level performance on 42 of 57 games

## Extensions
### Robustness to human starts
The previous tests the authors did don't test generalisability, and so they try performing these tests on random starting points in an episode, based on a human expert's trajectory. In these scenarios, the dueling network still performs very well, mostly outperforming the single stream baseline.[^3] 

### Combining with prioritised experience replay
Applying prioritised replay yields an agent that does significantly better than the prioritised baseline agent and the dueling agent alone. 

### Saliency maps
The authors also use saliency maps to visualise the roles of the value and advantage streams. The value stream appears to pay more attention to general features that are always present, whereas the advantage stream focusses on things that are immediately important. 

[^1]: In my opinion, the following two equations are a dubious abuse of notation. The *definition* of the advantage is the difference between $Q$ and $V$. If I'm not mistaken, the suggested adjustments are not really $Q$ functions, but are "other things" that allow us to still learn the optimal policy. It's possible that I've misunderstood the paper though!

[^2]: For reasons I don't understand, the authors say that we "force the advantage function estimator to have zero advantage at the *chosen* action" (emphasis mine). 

[^3]: This is very similar to the approach taken in *[Deep Reinforcement Learning with Double Q-learning](/personal-notes/2022/03/25/deep-reinforcement-learning-with-double-Q-learning.html)*.