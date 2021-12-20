---
layout: article
title: "AI Papers #4: Playing Atari with Deep Reinforcement Learning (DQN)"
sidebar:
  nav: docs-en
aside:
  toc: true
show_tags: true
tags: AI papers reinforcement-learning deep-learning
---

Where I start [Spinning Up in Deep RL](https://spinningup.openai.com/en/latest/spinningup/keypapers.html#a-deep-q-learning), with a really groundbreaking paper in RL...

---

**Epistemic status**: moderate confidence, expect some misconceptions. 

<br />

# Overview
This week I embark on an attempt to level-up in deep RL, through OpenAI's Spinning Up in Deep RL resource. The first paper I'll be reading is a classic - *[Playing Atari with Deep Reinforcement Learning](https://www.cs.toronto.edu/~vmnih/docs/dqn.pdf)*, by Mnih *et al.* (2013). This was the paper that really brought RL back into the limelight, and sparked a bunch of further breakthroughs in following years. As far as I know, progress is still going strong in this area (e.g. see the recent paper on [EfficientZero](https://arxiv.org/pdf/2111.00210.pdf) by Ye *et al.*)

# Summary
One major challenge in reinforcement learning (RL) is to control agents using high-dimensional sensory inputs, like vision. This has become possible due to deep learning (DL), but is still challenging for several reasons: 
* **Sparsity**: DL needs lots of data, but the reward signal in RL is often sparse
* **Delay**: feedback signals in DL are immediate, but can be delayed by thousands of timesteps in RL
* **Non-independence**: most of DL operates under the assumption of data sample *independence*, but in RL the states that an agent encounters are highly correlated
* **Dynamic (non-identically distributed)**: DL usually works with a fixed distribution, but the distribution can change in RL if the algorithm learns new behaviours

How can we overcome these challenges? This paper introduces a model that extends ordinary [Q-learning](https://en.wikipedia.org/wiki/Q-learning), so that it incorporates deep learning methods. 

The setup is an Atari game, where an agent interacts with an environment (i.e. the Atari emulator). At each time step, the agent can only observe the *current* screen and is unable to access the emulator state, so the only way for it to understand its situation sufficiently (and choose strategies) is via a *sequence* of actions and observations. 

For the agent to maximise the expected future return, one approach we could take is to maximise the action-value function (or $Q$-value), iteratively updating this at each time step to reach the optimal value $Q^\*$ (i.e. [value iteration](https://en.wikipedia.org/wiki/Markov_decision_process#Value_iteration)). 

From a computational standpoint, this approach isn't very practical. Instead, we can *estimate* the $Q$-value using a function approximator $Q(s,a;\theta) \approx Q^\*(s,a)$. This can be done using both linear and nonlinear function approximators, like neural networks. Such a network with weights $\theta$ is called a $Q$-network, and can be trained by minimising a sequence of loss functions $L_i(\theta_i)$ that depend on the iteration $i$. Here $L_i$ is taken to be the expected squared error over the **behaviour distribution** (probability distribution over the sequences of states and actions. 

The **$Q$-learning** algorithm consists of optimising the loss functions $L_i$ using stochastic gradient descent. It is **model-free** because the agent learns directly from samples without constructing a model of the environment. It is also **off-policy**, since it learns the optimal policy while its actions are following a different strategy, specifically the $\epsilon$-greedy strategy (follow the greedy strategy with probability $1 - \epsilon$, take a random action with probability $\epsilon$[^1]).

Where deep learning comes in is by training a deep neural network to do these $Q$-value updates. This is done using **experience replay**, where the states, actions and rewards of the agent at each time step is stored into a **replay memory**. Data is then sampled from this memory *at random* to train the algorithm after multiple time steps $T$, rather than after each step[^2]. After this, the agent selects an action using an $\epsilon$-greedy policy[^3]. 

The specific architecture used is a convolutional neural network (this is called the **deep $Q$ network**, or DQN for short), which only takes in the state representation as input, and predicts all the actions corresponding to this state at once[^4]. 

The authors train and test the agents on seven Atari games, using the same hyperparameters, and containing "almost no prior knowledge about the inputs"[^5]. It beats a "human expert" on three games and outperforms previous approaches on six! The authors speculate that the agent performs more poorly on certain games (e.g. space invaders) because these require finding a long-term strategy. 

# Thoughts
I find it interesting that trying to do Q-learning with nonlinear function approximators caused the Q-network to diverge, such that following RL research mostly focused on linear approximators. I wonder what other similar situations like this occur; where a setback in one research direction holds back progress in it for some time.

I suppose I'm not that surprised by the results of this paper because I knew them beforehand, but it was still a very interesting read. Considering the circumstances at the time, I find the approaches used very insightful and the results to be very impressive. I'm very curious to see how they improved upon these techniques in future papers. 

[^1]: This is the author's implementation of the explore/exploit tradeoff. The $Q$ function is only an approximation and is unlikely to be super accurate, to gathering information by taking random actions is critical (to a certain extent). 

[^2]: This approach helps solve the issues with the data not being IID. If instead we were to train from the "last $n$ steps" rather than at random, the states would be highly correlated and the IID assumption fails. 

[^3]: A benefit of off-policy learning is that it prevents the training samples being biased. If on-policy learning is used, then if maximising the return involves going left, the resulting training samples will mostly also be going left. This could lead to problems (e.g. getting stuck in a local minimum, poor generalisability). 

[^4]: This saves on quite a bit of compute (and time) - rather than just predicting $Q(s,a)$ from $s$ for one particular action $a$, this approaches estimates *all* the $Q(s,a_i)$ for *all* possible actions $a_i$ from $s$ in one go. 

[^5]: Considering how hard it was thought to be to train such agents in the past, this is very surprising and impressive. The agents are forced to detect objects on their own, rather than have the abstracted object presented to them as inputs. 