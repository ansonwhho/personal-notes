---
layout: article
title: "Deep Recurrent Q-Learning for Partially Observable MDPs (Hausknecht, 2015)"
sidebar:
  nav: docs-en
aside:
  toc: true
show_tags: true
tags: AI papers reinforcement-learning deep-learning
---

*[Link to paper by Hausknecht and Stone.](https://arxiv.org/pdf/1507.06527.pdf)*

# Motivation
If we look back to the [original paper on DQNs](https://www.cs.toronto.edu/~vmnih/docs/dqn.pdf), we notice some problems:
- **Limited history**: the DQN is trained on stacks of 4 consecutive frames - if we need more than 4 frames to capture all the necessary information for learning, then the [MDP](https://en.wikipedia.org/wiki/Markov_decision_process) becomes partially observable. But vanilla DQNs only work in a [POMDP](https://en.wikipedia.org/wiki/Partially_observable_Markov_decision_process) if its observations accurately reflect the true state
- **Fixed-length inputs**: we can't learn from sequences of varying lengths

These problems make vanilla DQNs really bad in POMDPs, which is annoying because most real-world situations aren't fully observable or Markovian. So what do we do?

## The key idea
These problems are reminiscent of standard sequence modelling, and fortunately we've had tools for solving these problems for some time - recurrent networks[^1]!

The big idea of this paper is thus to combine recurrent networks with DQNs to form a new network - the **Deep Recurrent $Q$-Network (DRQN)**. 

# Experiment
## Architecture
This is basically the same as for the original DQN paper, but instead of going directly from convolutional layers to full layers, we pass through an [LSTM](https://colah.github.io/posts/2015-08-Understanding-LSTMs/) layer. 

The convolutional layers are chosen in classic 2015 fashion - in successive layers, the kernel size decreases, the stride decreases, and the number of channels increases.[^2] 

## Training
### Data
The authors select Atari game episodes randomly from [replay memory](https://paperswithcode.com/method/experience-replay), and generate targets using a target network. They also do two kinds of updates:
- **Bootstraped Sequential Updates**: 
  - Updates occur throughout the episode from start to finish
  - The LSTM's hidden state is carried forward throughout the episode
- **Bootstrapped Random Updates**: 
  - Updates begin at random times in the episode, and proceed during unroll iterations of the LSTM
  - This is better for randomly sampling experience, but this requires that the hidden state is zeroed at the start of each update, making it harder for the LSTM to learn functions spanning long times

Some games that are originally POMDPs become MDPs due to sampling consecutive frames, so if we want to test the performance of the DRQN on POMDPs, we need to introduce partial observability carefully. 

The authors do this using *flickering Atari games* - at each time step, the screen is fully obscured with 50% probability, yielding a POMPDP.

### Learning
Training is done using standard [$Q$-learning](https://en.wikipedia.org/wiki/Q-learning) and [BPTT](https://en.wikipedia.org/wiki/Backpropagation_through_time).

## Evaluation
- The authors first test the DRQN on nine Atari games, all of which are MDPs
  - Performance is better than the DQN on five games, but significantly worse on others
- They then try and generalise a DRQN trained on an MDP to a POMDP, by playing the flickering equivalents of the games
  - The DRQN performance suffers due to this, but not as much as a DQN

### Convolutional layers
Later convolutional layers get better at detecting the relevant features of the input, as one might expect:
- Conv1: detects the paddle
- Conv2: jointly tracks the ball and paddle (sometimes)
- Conv3: tracks ball and paddle interactions (almost always)

Overall, it seems that the DRQN is more robust against missing information than a DQN, and can be very useful for handling partially observable environments. 

[^1]: This was before transformers were a thing. Recurrent networks are still pretty good though!

[^2]: 2015 before [ResNet](https://arxiv.org/abs/1512.03385), anyhow! That was published in December, but this DRQN paper was published in July. 