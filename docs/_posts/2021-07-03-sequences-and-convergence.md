---
layout: article
title: Sequences and convergence
sidebar:
  nav: docs-en
aside:
  toc: true
---

## Preliminary definitions
In real analysis we build up theory starting from the fundamental axioms of the real number system $\mathbb{R}$. Now we move on to the next step, which is to look at properties of the reals, starting with sequences. Intuitively these are quite similar to sets of numbers, except that with sequences both order and repetitions matter. 

### Definition 1 (Sequences)
A *sequence of real numbers* is a function $f : \mathbb{N} \rightarrow \mathbb{R}$ going from the naturals $\mathbb{N} = {1, 2, ...}$ to the reals. We'll denote $f(n)$ by $x_n$ and write the sequence as $(x_n)$. 

### Example sequences
There are many examples of sequences that we could come up with: 
- Some can be written in a compact form: $\left( \frac{1}{2^n} \right) = \left( \frac{1}{2} + \frac{1}{4} + \frac{1}{8} + ... \right)$
- The constant sequence $a$: $(x_n) = (a, a, a, ...)$

What can we say about such sequences? One is the behaviour of the terms as $n \rightarrow \infty$. For example, as $n$ gets really large, do the terms tend towards some particular value? This brings us to the important notion of *convergence*, which tells us that as $n$ tends towards infinity, the terms in a sequence $(x_n)$ get arbitrarily close to some value $x$ (pay particular attention to the fact that the definition holds $\forall \epsilon > 0$). 

### Definition 2 (Convergence)
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

## Sequences and limits
Now that we have a basic notion of what sequences are, we can move on to trying to find out more about what makes a sequence convergent, specifically by looking at their limits. One thing we might notice is that if a sequence is convergent, then by definition its terms tend towards some particular value, and so we would expect its terms to all lie within a particular range. 

### Definition 3 (Boundedness)
A sequence $(x_n)$ is *bounded* if $\exists M \geq 0$ s.t. $\lvert x_n \rvert \leq M$ for all $n \in \mathbb{N}$. 

So there is some number that is larger than the absolute value of every term in the sequence. This immediately leads us to an important result. 

### Theorem 1 (Convergence and bounds)
**Theorem statement**: Every convergent sequence is bounded.  

**Proof**: Let $(x_n)$ be a convergent sequence such that $\lim_\limits{x \to \infty} x_n = x \in \mathbb{R}$. From the definition of convergence, there exists some $N \in \mathbb{N}$ such that for all $n \geq N$, $\lvert x_n - x \rvert \leq 1$. So for all such $n \geq N$, we have $\lvert x_n \rvert \leq \lvert x_n - x + x \rvert \leq \lvert x_n - x \rvert + \lvert x \rvert \leq 1 + \lvert x \rvert$. Thus we can define

$ M := \text{max}{\lvert x_1 \rvert, \lvert x_2 \rvert, ..., \lvert x_N \rvert, 1 + \lvert x \rvert},$

which satisfies the property that $\lvert x \rvert \leq M \forall n \in \mathbb{N}$. Notice that a key aspect of this proof is that we're able to define $M$ in this way, and this is because there are only a finite number of terms in the sequence leading up to $x_N$. $\Box$

Given the properties of $\mathbb{R}$ a natural next step to consider is how the properties of convergence hold up under the standard arithmetic operations - this is the motivation for the next theorem. 

### Theorem 2 (Limit properties)
**Theorem statement**: Let $a \in \mathbb{R}$ and $(x_n)$, $(y_n)$ be sequences such that $x_n \to x$ and $y_n \to y$ as $n \to \infty$ (for $x, y \in \mathbb{R}$). Then the following are true: 
1. $x_n + y_n \to x + y$
2. $x_n y_n \to xy$
3. If $y_n \neq 0$ for all $n \in \mathbb{N}$ and $y \neq 0$ then $\frac{1}{y_n} \to \frac{1}{y}$
4. If $y_n \neq 0$ for all $n \in \mathbb{N}$ and $y \neq 0$ then $\frac{x_n}{y_n} \to \frac{x}{y}$
5. $a x_n \to ax$
6. $a + x_n \to a + x$ 
7. If $x_n \leq y_n$ for all $n \in \mathbb{N}$ then $x \leq y$

**Proof**: Left as an exercise. $\Box$

### Example
Theorem 2 is very helpful in allowing us to find the limits of many sequences. As an example, let's try to find the limit of $\frac{3n+7}{2n+1}$ as $n \to \infty$. We would run into problems if we were to try to directly apply theorem 2 to this, because both the numerator and denominator tend to infinity. However, with a bit of manipulation the issue disappears,

$\frac{3n+7}{2n+1} = \frac{3 + 7/n}{2 + 1/n} \to \frac{3}{2}$ as $n \to \infty$,

where you should justify to yourself which statements in theorem 2 we have used. Hence we have $\lim_\limits{n \to \infty} \frac{3 + 7/n}{2 + 1/n} = \frac{3}{2}$. 

### Exercises
1. Prove that $((-1)^n)$ is divergent. 
2. For $a \in \mathbb{R}$, prove that $\lvert a \rvert < 1 \implies (a^n)$ converges to 0, and that $\lvert a \rvert > 1 \implies (a^n)$ tends to $\infty$. (Hint: you might find [Bernoulli's inequality](https://en.wikipedia.org/wiki/Bernoulli%27s_inequality) useful.)
3. Show, from first principles, that $\lim_\limits{n \to \infty} \left( \frac{1}{n} - \frac{1}{n+1} \right) = 0$.
4. Show that $\lim_\limits{n \to \infty} \left( \frac{2^n}{n!} \right) = 0$.
5. Let $(x_n)$ be a convergent sequence. Prove that its limit is unique. 
6. Try and think of examples where theorem 1 would be useful. 
7. Write up the proof of theorem 2 - can you generalise these results? 

## Resources
1. *MT2502 Analysis* 2018 lecture notes by Dr Mike Todd at the University of St Andrews, chapter 2
2. *Introduction to Real Analysis* (4th ed.) by Bartle and Sherbert, chapter 3