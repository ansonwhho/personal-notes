---
layout: article
title: "Concrete problems in AI safety (Amodei, 2016)"
sidebar:
  nav: docs-en
aside:
  toc: true
show_tags: true
tags: AI AI-safety papers
---

# Overview
*[Concrete Problems in AI Safety](https://arxiv.org/pdf/1606.06565.pdf)* is a paper published in 2016 by Dario Amodei *et al*, one of the first papers to bring broader awareness to the issue of AI safety[^1]. The paper focuses on harmful societal impacts due to the poor design of AI systems, and proposes relevant research problems. 

I first heard about this paper several years ago through [a video on Computerphile](https://www.youtube.com/watch?v=AjyM-f8rDpg). At the time I didn't give it much attention, but recently I've become a lot more interested in AI safety and decided that it was high time I read it!

# Summary
AI systems have been growing more powerful and successful, with a lot of excitement about their potential benefits, as well as concern about their harmful side effects. As systems and environments become more complex and autonomous, it seems plausible that unforeseen side effects are becoming both more likely and harder to prevent.

This paper focuses on one particular source of negative impacts from AI: *accidents*, which the authors define as "unintended and harmful behaviour emerging from machine learning systems". 

These negative effects occur due to an engineering problem, where the designer of the AI system had a certain objective in mind, but when implemented led to harmful effects. The authors distinguish between three types of such situations: 
1. The designer does not know the correct formal specification of the objective function, and thus implements it incorrectly
2. The designer knows the correct formal specification or is able to evaluate it, but it is too expensive to implement it frequently
3. The designer correctly specifies the formal objective, but problems arise when due to a lack of good data, or an insufficiently expressive model

## Case 1: Incorrect formal specification of objectives
When the designer doesn't know how to formally specify the objective, it can lead to an AI system (like a [reinforcement learning (RL)](https://en.wikipedia.org/wiki/Reinforcement_learning) agent) taking unrelated and destructive actions. 

### Avoiding negative side effects 
As a contrived but illustrative example, suppose I tell a robot to make me a cup of coffee, and there happens to be a child on the straight-line path between the robot and the coffee machine. Since the robot knows nothing about my [common-sense preferences](https://en.wikipedia.org/wiki/Commonsense_knowledge_(artificial_intelligence)), the robot might push the child out of the way so that it can now walk directly towards the coffee machine. 

The problem here is that while I specified the objective as "make me a cup of coffee", what I *actually* wanted was "make me a cup of coffee while respecting my set of preferences", such as "it's bad to push children for no good reason". More generally, we might ask an AI system to do a task $X$, but what we really want is for it to do task $X$ *given certain constraints*. 

Unfortunately, it's very difficult to specify such preferences or constraints precisely. A naive approach is to make a really [large database of common-sense concepts](https://en.wikipedia.org/wiki/Open_Mind_Common_Sense), but this is intractable in practice, and it's hard to tell if we've missed something important[^2]. 

What we need to figure out is how to avoid these negative side effects. The approaches below are hopefully general enough to be transferable across tasks (with potentially different side effects):
* **Define an impact regulariser**: i.e. something that penalise "changes to the environment", e.g. relative to a [policy](https://spinningup.openai.com/en/latest/spinningup/rl_intro.html#key-concepts-and-terminology) that is known to be safe, to disincentivise harmful actions
* **Learn an impact regulariser**: since formalising "changes to the environment" is challenging, we can also try to *learn* a generalised impact regulariser instead
* **Penalise influence**: disincentivise the agent from getting into positions where it could easily do things with side effects
* **Multi-agent approaches**: rather than avoiding *all* side effects, we can focus on avoiding *[negative externalities](https://en.wikipedia.org/wiki/Externality#:~:text=A%20negative%20externality%20is%20any,an%20indirect%20cost%20to%20individuals.)* that harm the interests of other actors. For instance, one could try making the goals of an RL agent transparent, allowing a human to penalise actions with many negative side effects
* **Reward uncertainty**: in an environment which is already fairly good according to our preferences, random changes to it are more likely to be bad than good. We can reflect this property in a [prior probability distribution](https://en.wikipedia.org/wiki/Prior_probability) over possible reward functions, effectively making the agent uncertain about what the correct reward function is, and causing it to avoid actions that significantly change the environment

Ideally, these approaches would place bounds on how much harm an agent can do.

### Avoiding reward hacking
Another way in which things could go wrong is reward hacking, where the agent exploits a property of the reward function to get extremely high reward in unintended fashion. If a clearning robot gets reward for not seeing any messes, then all it has to do is close its eyes[^3]. 

There are many ways in which reward hacking can occur, which we'll need to understand in order to prevent: 
* **Partially observed goals**: real-world tasks are usually only partially observable, leading to imperfect measures of task performance (e.g. we may not be able to see every corner of the room from where we are standing, so it's hard to verify that the whole room is clean), often resulting in hackable reward functions
* **Complicated systems**: reward hacking is more likely in more complicated agents with more available strategies, similar to how bugs are more likely in more complicated programs
* **Abstract rewards**: complicated reward functions need to refer to abstract concepts, which can be vulnerable to hacking
* **Goodhart's Law**: the designer chooses an objective that is highly correlated with accomplishing the task, but this correlation no longer holds when it is being aggressively optimised for[^4]
* **Feedback loops**: a positive feedback loop may distort what the objective function was meant to represent (e.g. [content selection algorithms](https://en.wikipedia.org/wiki/Social_media_reach) can make certain ads more popular and appear more prominent, further accentuating their popularity, and so on...)
* **Environmental embedding**: some agents may be able to tamper with the implementation of the reward function, to maximise their score

These issues are general causes for choosing the wrong objective function, and suggest that reward hacking isn't just due to incompetence in individual domains. While these issues are mostly small in current systems, they may get more critical as AI systems become more powerful. 

Some approaches to mitigate reward hacking include: 
* **Adversarial reward functions**: the current issue is that the agent finds an exploit in the reward function. Doing this could be a lot harder if the reward function is able to behave like and agent and respond to attempts to game it - for instance, such a "reward agent" could try to act as an adversary to the AI system, finding situations which the AI system predicts high reward but the human labels as low reward[^5]
* **Model lookahead**: we can prevent environmental embedding by looking at anticipated future states instead of the present one - although we cannot control rewards once the agent replaces the reward function, we can penalise *plans* to replace the reward function
* **Adversarial blinding**: adversarial techniques can blind a model from variables and thus prevent it from understanding things like "how reward is generated", thus deterring reward hacking
* **Careful engineering**: [sandboxing](https://en.wikipedia.org/wiki/Sandbox_(computer_security)) and [formal verification](https://en.wikipedia.org/wiki/Formal_verification) of parts of the AI system can guarantee reasonable behaviour
* **Reward capping**: placing a maximum bound on possible reward prevents some types of reward hacking, like very low-probability, high-payoff strategies
* **Counterexample resistance**: techniques like [weight uncertainty](https://arxiv.org/pdf/1505.05424.pdf) can counter abstract rewards (which we can think of as a problem of [adversarial examples](https://openai.com/blog/adversarial-example-research/))
* **Multiple rewards**: having multiple rewards can make it generally harder to do reward hacking
* **Reward pretraining**: we can pretrain a fixed reward function using supervised learning (without interaction with the environment), so as to prevent the agent from influencing it
* **Variable indifference**: try to get the agent to optimise for certain variables of interest, while not optimising for things it is indifferent about
* **Trip wires**: use diagnostic tools to spot if an agent is trying to hack its reward function

These techniques have their shortcomings, but they hopefully provide some concrete approaches to reducing risks from reward gaming. 

## Case 2
### Scalable oversight 
One cause of safety issues is that we can't always give fully representative reward signals to the AI agent. For instance, giving a good reward signal might require several hours of analysis which we generally don't have (i.e. we have a limited "oversight budget"), and we thus use cheaper methods for evaluating agent performance during training. However, these cheaper methods don't fully capture our preferences, and lead to side effects. How can agents efficiently achieve goals when feedback is very expensive?

One approach to tackle this is through the framework of [semi-supervised reinforcement learning](https://ai-alignment.com/semi-supervised-reinforcement-learning-cf7d5375197f), where the agent is only allowed to see its rewards on a subset of episodes or timesteps. One particular case of this is active learning, where the agent requests to see the reward at certain episodes or timesteps, with the aim of being economical in feedback requests and total training time. 

In this case the agent needs to optimise its return, but with less information compared to ordinary episodic RL. With this constraint, the agent is incentivised to be transparent, since it wants to get as much feedback as possible about whether it has accurately predicted when the human will give it high reward for its actions. 

Some approaches to semi-supervised RL include: 
* **Supervised reward learning**: predict the reward on a per-episode basis, to estimate the payoff of unlabelled episodes
* **Semi-supervised or active reward learning**: use active learning to efficiently learn an estimator for the human-provided reward
* **Unsupervised value iteration**: using the observed transitions of unlabelled episodes to improve [value iteration](https://artint.info/2e/html/ArtInt2e.Ch9.S5.SS2.html)
* **Unsupervised model learning**: using the observed transitions of the unlabelled episodes to improve quality of the RL model

It seems likely that an effective approach to semi-supervised RL would help mitigate many AI safety problems, because of these incentives towards transparency. Other approaches to scalable oversight include: 
* **Distant supervision**: provide information about the agent's actions in aggregate, e.g. using statistical data
* **Hierarchical reinforcement learning**: this works by taking a having a top-level agent take high-level actions over large time scales, and delegating tasks to sub-agents (which it incentivises using a synthetic reward signal). Agents at the lowest level directly take primitive actions in the environment, and the top-level agent needs to be able to learn from sparse rewards[^6]. 

## Case 3
### Safe exploration
Exploration is both necessary and potentially dangerous, especially in real-world scenarios where the outcomes can be terminal (e.g. autonomous vehicles can only do so much trial and error). Current RL uses a manually specified avoidance of catastrophic behaviours, but this is very hard to implement in general domains. How can an agent explore safely?

Some popular approaches to safe exploration include: 
* **Risk-sensitive performance criteria**: change the optimisation criteria to prevent catastrophic events
* **Demonstrations**: learn from expert demonstrations of near-optimal behaviour to create a baseline policy and reduce the need for exploration
* **Simulated exploration**: it is safer to explore in simulated environments rather than the real world, although this is bottlenecked by the accuracy of the simulator
* **Bounded exploration**: we can let agents freely explore in parts of the state space that we know to be safe or boundedly harmful
* **Trusted policy oversight**: similarly, we can limit the actions of an agent to a trusted policy that only consists of actions that can be recovered from
* **Human oversight**: check potentially unsafe actions with a human - this has the issue of scalable oversight

### Robustness to distributional shift
Suppose a machine learning model is deployed on a test distribution that differs from its training distribution. Will its predictions remain *robust* to this distributional change (i.e. perform well on the test distribution, and know when it is unlikely to make good predictions)?

This problem of out-of-distribution (OOD) robustness is well known in statistics, and many approaches exist. One set of approaches looks at how to *detect* when a model will not do well in a novel distribution: 
* **Well-specified models**: for a prediction task with input $x$ and target $y$, one approach is to make the [robust statistics](https://en.wikipedia.org/wiki/Robust_statistics) assumption of [covariate shift](https://www.stat.berkeley.edu/~jsteinhardt/stat260/notes/lect20.pdf). Under this assumption, the only thing that changes between the test and train distributions is the distribution of [covariates](https://en.wikipedia.org/wiki/Dependent_and_independent_variables#Statistics_synonyms), while the output distribution remains the same. Making this assumption allows us to implement techniques like [importance sampling](https://en.wikipedia.org/wiki/Importance_sampling) to help make more accurate estimates of the test distribution
* **Partially specified models**: many assumptions need to be made for the case of well-specified models, and relaxing some of these brings us into the regime of other methods, like the [method of moments](https://en.wikipedia.org/wiki/Method_of_moments_(statistics)) (which estimates distributional parameters) and [unsupervised risk estimation](https://www.stat.berkeley.edu/~jsteinhardt/publications/risk-estimation/preprint.pdf) (tell whether a model is performing well by looking at its distribution of errors)
* **Training on multiple distributions**: if a model works well on multiple distributions, (hopefully) it also performs well on new ones

Other approaches instead focus on how to *respond* when OOD:
* **Asking humans for information** after detecting that a model is unlikely to make good predictions
* **Explore** to gather information (for agents that act in an environment)

We can think about OOD prediction in terms of [counterfactual reasoning](https://www.lesswrong.com/tag/counterfactuals) ("what would have happened in another distribution?") and machine learning with contracts ("how can we design models that behave according to a contract, with certain conditions like 'the training and test distributions are equal'?"). 

## Conclusion
Takeaway: problems in AI safety (not limited to *accidents*) are important, and there are many concrete problems for people to work on!

---

[^1]: When I refer to AI safety, I am specifically (and almost tautologically) referring to the problem of building safe AI systems. However I do believe this is an important point to raise, because "AI safety" is often construed as being equivalent to the [AI Alignment Problem](https://en.wikipedia.org/wiki/AI_control_problem#Alignment). 

[^2]: In fact a large part of the reason why machine learning and AI exists is because it's hard to specify exactly what we care about, [and what we want machines to do for us.](https://www.deeplearningbook.org/contents/intro.html) 

[^3]: Many real-life examples of reward hacking can be found at Victoria Krakovna's post, *[Specification gaming examples in AI](https://vkrakovna.wordpress.com/2018/04/02/specification-gaming-examples-in-ai/)*.

[^4]: [Goodhart's law](https://en.wikipedia.org/wiki/Goodhart%27s_law) says that "When a measure becomes a target, it ceases to be a good measure". The [h-index](https://en.wikipedia.org/wiki/H-index) is a correlated measure of research productivity, but it this correlation can break down if researchers focus on maximising their h-index. [Terence Tao discusses something similar](https://terrytao.wordpress.com/career-advice/theres-more-to-mathematics-than-grades-and-exams-and-methods/) when learning mathematics. 

[^5]: A related architecture is the [Generative Adversarial Network (GAN)](https://en.wikipedia.org/wiki/Generative_adversarial_network), which consists of two mutually adversarial neural networks that "try to fool each other" and produce realistic images. In the case of adversarial reward functions, the AI system is instead trained to better predict situations where humans would give high reward.

[^6]: This sounds similar to [Ought's Factored Cognition project](https://ought.org/research/factored-cognition), based on Paul Christiano's [Iterated Amplification (IDA)](https://www.lesswrong.com/s/EmDuGeRw749sD3GKd) proposal. I'm not sure if hierarchical RL was part of the inspiration for IDA (my understanding is that IDA was in part motivated by how [AlphaGo Zero](https://deepmind.com/blog/article/alphago-zero-starting-scratch) works). 