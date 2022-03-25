---
layout: article
title: "Prioritized Experience Replay (Schaul, 2015)"
sidebar:
  nav: docs-en
aside:
  toc: true
show_tags: true
tags: AI papers reinforcement-learning deep-learning
---

*[Link to the paper by Schaul et al.](https://arxiv.org/pdf/1511.05952.pdf)*

# Motivation
Online RL often works by observing a stream of experience and discarding data immediately after an update. This has several problems: 
- SGD typically makes an i.i.d. assumption, but data from a stream of experience can be highly correlated
- Rapid forgetting of experiences that would be useful later on
- Poor sample efficiency

One solution to these problems is **experience replay**[^1], which was implemented in the original DQN paper. This had a *sliding window replay memory*, i.e. it stored experience from a fixed number of past timesteps. 

It also replayed transitions uniformly, but some transitions are more useful for learning than others. To make things more efficient, we can *prioritise* which transitions to replay.[^2] 

## Approach
### Blind Cliffwalk
The authors demonstrate the benefits of prioritisation using the "Blind Cliffwalk" environment. This consists of $n$ states, and at each state there is one "right" action (1 reward) and one "wrong" one (0 reward). Taking a "wrong" action terminates the episode, so the odds of getting positive reward via a random sequence of actions is $2^{-n}$. 

Here the useful experience is very sparse, and so prioritised replay can be very useful.

### Prioritising with TD-error
One way of prioritising the importance of a transition is the **expected learning progress** - how much the agent can learn from a transition in its current state. This isn't directly accessible but can be roughly approximated by the TD error $\delta$. 

Implementation: 
- Store the last encountered $\delta$ with each transition in the replay memory
- The transition with the largest $\mid \delta \mid $ is replayed
- $Q$-learning updates are done in proportion to the size of $\delta$
- Newly added transitions are prioritised for replay because they have an unknown TD error

But this has some problems: 
- Transitions with low TD error may not be replayed for a long time
- Prioritisation with $\delta$ is sensitive to noise spikes
- Greedy prioritsation focuses on a small subset of experience, especially because TD errors update slowly and high error transitions are frequently replayed

### Stochastic prioritisation
To fix the issues with TD-error prioritisation, the authors stochastically switch between greedy prioritisation and uniform random sampling. 

This changes the distribution in uncontrolled fashion, leading to a bias. The authors counter this using [importance sampling](https://timvieira.github.io/blog/post/2014/12/21/importance-sampling/), such that the importance weight is annealed over time.[^3] 

# Experiment
The authors compare performance on Atari games with two baseline algorithms - the vanilla DQN and the Double DQN, based on the quality of the best learned policy. 

They find that:
- Prioritised replay substantially improves performance on most games for both the DQN and the Double DQN
- Prioritised replay helps replay states shortly after being encountered, helping the policy learn from more recent experience

## Extensions
- **Prioritised supervised learning**:  
The authors apply a similar approach to standard supervised learning, by removing 99% of the samples of digits 0,1,2,3,4 from MNIST. Here they find that prioritised sampling is much better than uniform sampling, and approaches the standard baseline performance with the full dataset. 
- **Off-policy replay**:  
If transitions can be stored in a replay buffer, then applying these techniques can also work with off-policy RL algorithms
- **Feedback for exploration**:  
The number of times that a transition is replayed can be used to inform the exploration strategy, generating more useful experiences
- **Prioritised memories**:  
We can reduce the required memory capacity by prioritising experience that is more likley to be useful

[^1]: Experience replay can reduce sample efficiency, while increasing computation and memory, which are often cheaper resources to acquire than data. 

[^2]: We can also improve efficiency by choosing which experiences to *store*, but this paper only focusses on prioritising which experiences to *replay*.

[^3]: In nonlinear function approximators, this approach further allows for improved performance in the nonlinear loss landscape.