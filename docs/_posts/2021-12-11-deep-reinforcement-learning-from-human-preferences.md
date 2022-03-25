---
layout: article
title: "Deep Reinforcement Learning from Human Preferences (Christiano, 2017)"
sidebar:
  nav: docs-en
aside:
  toc: true
show_tags: true
tags: AI AI-safety papers reinforcement-learning deep-learning
---
# Overview
This is a summary of *[Deep Reinforcement Learning from Human Preferences](https://arxiv.org/abs/1706.03741)*, by Christiano *et al.*

## Summary
Part of the reason why AI alignment is a challenging problem is that we cannot easily write down a function that exhaustively and accurately reflects human values[^1], placing a serious constraint on standard reinforcement learning (RL) techniques. How can we teach RL systems to behave according to our preferences?

What we're looking for is a reward function satisfies several properties: 
1. It helps RL agents solve tasks where humans can recognise good behaviour, without necessarily being able to demonstrate it[^2]
2. It allows for agents that can be taught by non-experts
3. It scales to large deep learning problems
4. It is economical with user feedback

The approach that the authors use is to compare two sequences of agent actions and observations (called *trajectory segments* - these are provided in the form of short video clips) and having a non-expert choose between which is better[^3]. The agent tries to follow trajectories that the human prefers, with as few queries as possible.

Mathematically, this corresponds to running a three-step process asynchronously to update a policy $\pi: \mathcal{O} \to \mathcal{A}$, where $\mathcal{O}$ is the space of observations and $\mathcal{A}$ is the action space. The reward function is estimated by $\hat{r}: \mathcal{O} \times \mathcal{A} \to \mathbb{R}$. 
1. Traditional RL: let the agent interact with the environment via $\pi$ (i.e. at time step $t$, the agent makes observation $o_t$ and takes action $a_t$) while maximising the expected cumulative rewards $r_t = \hat{r}(o_t, a_t)$[^4], producing a set of trajectories $\tau$
2. Select pairs of segments $(\sigma^1, \sigma^2)$ from $\tau$ and let the human choose which one seems better[^5]
3. Train the parameters of $\hat{r}$ on the human comparisons via supervised learning 

The interesting step here is step 2 - unlike in traditional RL where the true reward signal is given purely by a manually defined reward function, here no reward function is explicitly given to the agent. Instead, the agent infers a function $\hat{r}$, essentially predicting what the human's preferences will be. 

This works by slightly modifying the [Bradley-Terry model](https://en.wikipedia.org/wiki/Bradley%E2%80%93Terry_model) for estimating score functions from pairwise preferences. To choose the value of $\hat{r}$, the authors trained an ensemble of predictors based on the human comparisons and averaged the results. 

Step 3 is also interesting from an engineering perspective because $\hat{r}$ changes as more human comparison data is collected, so choosing an appropriate RL algorithm is important (as we'll soon see). 

The authors implement this procedure on two tasks: [Atari games](https://www.cs.toronto.edu/~vmnih/docs/dqn.pdf) and simulated robot locomotion - let's take a look at each of these in turn. 
* **Simulated robotics**: In total, there were eight robotics tasks[^6]. With around 700 levels (from queries to a non-expert human), the agents are almost at the level of standard RL with synthetic feedback on all of tasks. With 1400 labels, the performance *exceeds* performance with synthetic feedback! Famously, in the game of [Hopper](https://gym.openai.com/envs/Hopper-v2/), the robot learns to consistently perform a sequence of backflips with 900 queries and in less than an hour
* **Atari games**: The approach is also tested on seven games in the [Arcade Learning Environment](https://arxiv.org/pdf/1207.4708.pdf)[^7] - in this case matching standard RL is a lot harder, but the method still matches or exceeds RL in some cases. The trained agent even exhibits novel behaviour in the game of [Enduro](https://en.wikipedia.org/wiki/Enduro_(video_game)), where a car learns to stay even with other moving cars for substantial periods of time[^8]

To further test the performance of the procedure, the authors make several modifications. Some notable differences that they found include: 
* The agent performs poorly with offline reward predictor training - in this case the predictor only captures part of the true reward, leading to [strange behaviours on a test distribution](/2021/12/07/concrete-problems-in-ai-safety.html)[^9]
* Performance is much better with short video clips rather than single frames on continuous tasks (i.e. with trajectories than states)

## Thoughts
At least to me, this method of predicting an implicit reward function seems really clever. I'm quite astonished that the agents can be trained with significantly less interaction time (3 orders of magnitude) than full human interaction. I'm quite surprised at how well it performs compared to standard RL, although there are some things that I'm a little unsure about. 

For instance, why does traditional RL do so much better for certain games? One reason I can think of is that the human evaluation simply isn't very good - RL does significantly better than this approach on Breakout, which has become a [classic example of AI finding a better game strategy than humans](https://www.youtube.com/watch?v=TmPfTpjtdgg). In such a scenario, it makes sense to me that the human signal would lose out to RL. 

I also don't currently understand why removing regularisation doesn't usually have a particularly large effect on the performance in most Atari games. 

I think other results were relatively more intuitive, for instance that performance improves quite a lot with short clips as opposed to single frames. Context seems really important, and it can be hard to accurately predict subsequent dynamics from a static image, especially if no other prior information is given[^10].

Overall, I found this paper quite intriguing, and makes me reasonably optimistic about our ability to solve AI alignment. I expect my level of optimism to change somewhat after reading more papers though, so we'll see.

[^1]: This is partially because we don't know what our values are ("Is morality just about reducing suffering?"), and partially because it could be very hard to encode them even if we did ("How do you tell an AI system to 'minimise suffering'?").  

[^2]: For instance, I can't do a backflip (but at least I've tried), so an agent would not be able to learn from me giving demonstrations. Despite my incompetence at *doing* backflips, I can quite easily *recognise* a good one. If we could produce demonstrations, then we could use techniques like [Inverse Reinforcement Learning](https://arxiv.org/pdf/1806.06877.pdf) or [Imitation Learning](https://paperswithcode.com/task/imitation-learning), instead of the method described in the paper. 

[^3]: This behaviour evaluation can be implemented quantitatively (where we compare the sums of rewards over the observation-action pairs of the trajectory segment) or qualitatively (where the human qualitatively determines which trajectory segment is more satisfactory).

[^4]: We can't always assume that the predicted reward only depends on the current observation and action. In a general [partially observed environment](https://en.wikipedia.org/wiki/Partially_observable_Markov_decision_process), we would need to consider the entire history of observations. 

[^5]: A benefit of doing pairwise comparisons rather than asking a human to give absolute evaluations of outcomes is that it's a lot easier for non-expert humans to make accurate evaluations. So they aren't just making things complicated for no reason! In addition, it's possible for the human to decide that the two clips are equally good, as well as incomparable. 

[^6]: These were conducted in a [MuJoCo](https://mujoco.org/) environment, with agents trained using [trust region policy optimisation (TRPO)](https://spinningup.openai.com/en/latest/algorithms/trpo.html). This approach helps generalise training to more complicated tasks. 

[^7]: The agents in Atari games are trained using the [advantage actor-critic (A2C)](https://arxiv.org/pdf/1602.01783v2.pdf) algorithm.

[^8]: For video or GIF demonstrations of these, see [OpenAI's blog post](https://openai.com/blog/deep-reinforcement-learning-from-human-preferences/) about this paper.

[^9]: The authors state that this is due to the "nonstationarity of the occupancy distribution", although at the moment I'm a little puzzled by this.

[^10]: I'm actually slightly confused about how it's even possible to get a decent training signal at all from static images. Unless there's a clear [arrow of time](https://en.wikipedia.org/wiki/Arrow_of_time) (e.g. an ice cube melting), how did the non-experts evaluate which of two static images was "better"? Presumably there are other features that are important, and maybe you don't *need* to know what the future dynamics will be, but I can imagine things getting a lot harder with purely static images. 