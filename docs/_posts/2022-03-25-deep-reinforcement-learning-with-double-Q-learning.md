---
layout: article
title: "Deep Reinforcement Learning with Double Q-learning (van Hasselt, 2015)"
sidebar:
  nav: docs-en
aside:
  toc: true
show_tags: true
tags: AI papers reinforcement-learning deep-learning
---

*[Link to paper by van Hasselt, Guez, and Silver](https://arxiv.org/pdf/1509.06461.pdf)*

# Motivation
Standard $Q$-learning has a tendency to result in overoptimistic $Q$ values. This can happen for several reasons (e.g. noise), but the important thing is that *any* estimation errors can cause an upward bias in $Q$-learning. 

To see why, consider that the $Q$-value update consists of changing the parameters via

$$\pmb{\theta}_{t+1} = \pmb{\theta}_t + \alpha (Y_t^Q - Q(S_t, A_t; \pmb{\theta}_t)) \nabla_{\pmb{\theta}_t} Q(S_t, Q_t; \pmb{\theta}_t),$$

where $Y_t^Q \equiv R_{t+1} + \gamma \max_a Q(S_{t+1}, a; \pmb{\theta}_t)$ is the *target*.

The culprit is the $\max$ operation in the target - since we're using the same values both to select and evaluate an action, we're more likely to choose overestimated $Q$ values. If these estimates are not *uniformly* overoptimistic, then this may result in suboptimal policies even in theory.

The goal is thus to empirically investigate these overestimations, and provide a solution that overcomes these issues. 

# Approach
## Double Q-learning
To avoid this overestimation problem, the authors use the Double $Q$-learning algorithm; i.e. they use two separate $Q$ functions with different sets of weights $\pmb{\theta}$ and $\pmb{\theta}'$ - one for policy evaluation, and another for policy improvement. 

This just involves changing the target from normal $Q$-learning ever so slightly, but making all the difference:
- **$Q$-learning**: $Y_t^Q = R_{t+1} + \gamma Q(S_{t+1}, \arg \max_a Q(S_{t+1}, a; \pmb{\theta}_t); \pmb{\theta}_t)$
- **Double $Q$-learning**: $Y_t^{DoubleQ} = R_{t+1} + \gamma Q(S_{t+1}, \arg \max_a Q(S_{t+1}, a; \pmb{\theta}_t); \pmb{\theta}_t')$

Thus we choose the action using the online weights $\pmb{\theta}_t$ and evaluate the policy using $\pmb{\theta}_t'$. 

## Double DQN
As you might expect, the authors combine double $Q$-learning with DQNs to give the **Double DQN** algorithm. This incorporates the idea of periodically updating the target (rather than at every step), in order to make learning more stable. 

# Evaluation
## Task
The authors then compare the estimates and performance of the DQN and the Double DQN on six Atari 2600 games, following essentially the same architecture as in the [standard DQN](https://www.nature.com/articles/nature14236). They compare these with "ground truth" average values, which are determined by running the best learned policies and computing the *true* returns. 

To test whether learned solutions generalise, they also try testing testing the agents from different starting points in an episode (based on a human expert's trajectory). 

## Results
- The DQN is consistently overoptimistic, and the Double DQN is a lot closer to "ground truth"
- Learning is much more stable with the Double DQN, suggesting that the instability of DQNs is due to overoptimism
- Overoptimism *doesn't always* adversely affect the quality of the learned policy (e.g. Pong), but reducing overestimations significantly improves stability of learning
- Double DQN seems more robust to starting from various points in an episode

Overall this suggests that Double DQN finds better policies and is more stable than standard DQNs. It also doesn't require anything more than the original DQN architecture, which is very nice!
