---
layout: article
title: Sequences and convergence
sidebar:
  nav: docs-en
aside:
  toc: true
---

In real analysis we build up theory starting from the fundamental axioms of the real number system $\mathbb{R}$. Now we move on to the next step, which is to look at properties of the reals, starting with sequences. Intuitively these are quite similar to sets of numbers, except that with sequences both order and repetitions matter. 

## Definition 1 (Sequences)
A *sequence of real numbers* is a function $f : \mathbb{N} \rightarrow \mathbb{R}$ going from the naturals $\mathbb{N} = {1, 2, ...}$ to the reals. We'll denote $f(n)$ by $x_n$ and write the sequence as $(x_n)$. 

### Example sequences
There are many examples of sequences that we could come up with: 
- Some can be written in a compact form: $\left( \frac{1}{2^n} \right) = \left( \frac{1}{2} + \frac{1}{4} + \frac{1}{8} + ... \right)$
- The constant sequence $a$: $(x_n) = (a, a, a, ...)$

What can we say about such sequences? One is the behaviour of the terms as $n \rightarrow \infty$. For example, as $n$ gets really large, do the terms tend towards some particular value? This brings us to the important notion of *convergence*, which tells us that as $n$ tends towards infinity, the terms in a sequence $(x_n)$ get arbitrarily close to some value $x$ (pay particular attention to the fact that the definition holds $\forall \epsilon > 0$). 

## Definition 2 (Convergence)
Let $(x_n)$ be a sequence and let $x \in \mathbb{R}$. 
#### Convergence 
$(x_n)$ *converges* to $x$ if  

<center>$\forall \epsilon > 0, \exists N \in \mathbb{N}$ such that $\forall n \geq N, \lvert x_n - x \rvert \leq \epsilon$</center>

$x$ is called the *limit* of the sequence, and we'll write $\lim\limits_{n \to \infty} x_n = x$ to denote this. 
#### Divergence  
If $(x_n)$ doesn't tend to some limit, then it is said to be *divergent*. A specific case of divergence is if the terms *tend to infinity*, where there is no upper bound to the terms in the sequence:  

<center>$\forall K \in \mathbb{R}, \exists N \in \mathbb{N}$ s.t. $\forall n \in \mathbb{N}, n \geq N \implies x_n \geq K$,</center> 

so no matter what $K$ you choose there is always an $x_n$ that is larger than it. In limit notation we can write this as $\lim\limits_{n \to \infty} x_n = \infty$. We can say something similar if the sequence *tends to minus infinity*, where instead

<center> $\forall K \in \mathbb{R}, \exists N \in \mathbb{N}$ s.t. $\forall n \in \mathbb{N}, n \geq N \implies x_n \leq K$, </center>

and we write this as $\lim\limits_{n \to \infty} x_n = - \infty$. 

### Convergence of $\left( \frac{1}{n^2} \right)$
Let's take a look at an example to see how this works.  
**Claim**: $\left( \frac{1}{n^2} \right)$ is convergent.  
**Proof**:  Let's return to the definition that we've just seen - the goal is to find some $N$ such that the terms $x_n = \frac{1}{n^2}$ get arbitrarily close to a limit (which we intuitively know to be 0) for all $n \geq N$. First, let $\epsilon > 0$, and for now let's just consider $N \in \mathbb{N}$ to be given. Then if $n \geq N$,   

<center> $\lvert x_n - x \rvert = \left\lvert \frac{1}{n^2} - 0 \right\rvert = \frac{1}{n^2} \leq \frac{1}{N^2}$. </center>

Hence we can choose $\frac{1}{N^2} < \epsilon \implies N > \frac{1}{\sqrt{\epsilon}}$, and we get that $\lvert x_n - x \rvert \leq \epsilon$, as desired. 

### The convergence game
In some sense you can think of testing for the convergence of a sequence to be like a game between two players, $A$ and $B$:  
1. Player $A$ claims that $x$ is the limit of a sequence $x_n$
2. Player $B$ challenges this by setting some specific $\epsilon > 0$
3. Player $A$ finds some $N$ such that $\lvert x_n - x \rvert \leq \epsilon$ for all $n \geq N$  

If player $A$ is always able to find some $N$ regardless of the choice of $\epsilon$ given by player $B$, then the sequence is convergent. If instead player $B$ finds some $\epsilon$ that $A$ isn't able to respond to, then the sequence is instead divergent. 

## Exercises
1. Prove that $((-1)^n)$ is divergent. 
2. For $a \in \mathbb{R}$, prove that $\lvert a \rvert < 1 \implies (a^n)$ converges to 0, and that $\lvert a \rvert > 1 \implies (a^n)$ tends to $\infty$. (Hint: you might find [Bernoulli's inequality](https://en.wikipedia.org/wiki/Bernoulli%27s_inequality) useful.)