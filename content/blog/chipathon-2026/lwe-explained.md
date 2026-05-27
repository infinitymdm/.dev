---
title: 'Learning With Errors'
date: 2026-05-27T13:49:06-05:00
tags:
    - cryptography
math: true
---

Cryptography is mostly about finding math problems that are really easy if you know some hidden information, but impossibly hard if you don't. Learning With Errors is one such problem.

<!-- more -->

This article is a brief introductory look at the Learning With Errors (LWE) problem, which is the foundation of many developing cryptosystems. I'll try to make it as digestible as possible. But be forewarned: here be equations.

## The Basic Idea

I'm going to come up with a list of secret numbers, and keep them to myself. This is my *private key*, chosen [completely randomly](https://xkcd.com/221/).

> Super secret numbers. Don't tell anyone.$$\begin{aligned} a &= 7 \\\\ b &= 16 \\\\ c &= 12 \end{aligned}$$

Then I'm going to post a [system of linear equations](https://en.wikipedia.org/wiki/System_of_linear_equations) (where my secret list of numbers solve each equation) all over the internet. This is my *public key*.

> $$\begin{aligned} 16a + 2b + 3c &= 180 \\\\ 8a + 18b + 6c &= 416 \\\\ 10a + 19b + 8c &= 470 \\\\ 3a + 4b + 3c &= 121 \\\\ 11a + 6b + 8c &= 269 \end{aligned}$$

Only I have the solution to this linear system, so I'm the only one who can solve these equations. Right?

### The Problem

Now, it might be a little difficult for you or I to find the correct values a, b, and c to solve this system, but you could probably work it out with a little effort. And computers are really good at linear equations. In fact, using only a few of these equations, a computer can find the solution in no time.

We might call this a learning *without* errors problem. The challenge for an attacker is to *learn* my secret key by solving the equations that make up my public key. Unfortunately this isn't much of a challenge, which is not what we want!

### Adding Errors

Ok, so that scheme didn't work very well. At least, not if I want to keep my numbers secret; any hooligan who knows about [Gaussian Elimination](https://en.wikipedia.org/wiki/Gaussian_elimination) can figure out my private key. How can we modify this scheme so it's hard to recover my secret numbers?

This is where the "Error" comes into "Learning with Errors". We're going to make all the equations in our public key *slightly wrong* by adding a small random error to each one.

> $$\begin{aligned} 16a + 2b + 3c &\approx 180 + 1 = 181 \\\\ 8a + 18b + 6c &\approx 416 - 2 = 414 \\\\ 10a + 19b + 8c &\approx 470 + 2 = 472 \\\\ 3a + 4b + 3c &\approx 121 + 2 = 123 \\\\ 11a + 6b + 8c &\approx 269 + 0 = 269 \end{aligned}$$

You might be thinking "Wait, won't that break everything?" And you'd be right! My secret numbers no longer solve the system of equations. But here's the catch: *no one else can solve the system of equations either*. In fact, our public key probably doesn't have any solution at all!

At first glance that might seem like a problem. Our private key is no longer a solution, which seems really bad! But as long as the error for each equation is small, our private key will probably be the closest set of numbers to solving these equations. We can just ignore the errors as random noise.

Picture an analog clock. If it's 10:58 and I ask you what time it is, you'll probably say 11 o'clock. Now *technically* there's some error there, but I'm not going to come back at you with "Wrong! Actually it's 10:58!". I would just look like a jerk. The key idea here is that we can ignore a little bit of error as long as it's less than half an hour. If we can ignore that error for the hour hand of a clock, we can do something very similar to ignore the error we've added into our public key (or other results based on our public key).

Now you might stil be concerned: if I'm posting my public key everywhere, even with these small random errors, won't someone be able to do the same thing as before and come up with a way to learn my private key? Well, it turns out that those small errors get amplified by the process of solving the linear system, and completely throw off the results. And because no one else knows what these random errors are, it's believed to be impossible to recover the secret list of numbers.

## Hard Mathy Stuff

While everything I've said so far is basically accurate, mathematicians like to formalize this stuff using funny symbols and letters. That's what we're going to do in this section. If it's boring, feel free to skip it.

- Our private key \\(\bm{s} \in \mathbb{Z_{q}^{n}}\\) is a vector of \\(n\\) integers randomly selected from \\(\mathbb{Z}_{q}\\) (the integers modulo \\(q\\)).
- We can generate as many equations as we want to form our public key as follows:
    + Choose another random integer vector \\(\bm{a} \in \mathbb{Z_{q}^{n}}\\) to be the coefficients.
    + Choose a random real error \\(e \in \mathbb{T}\\) (between 0 and 1).
    + Compute the real constant on the other side of the equation as \\(t = (\bm{a} \cdot \bm{s}) / q + e\\)
    + The equation can easily be reconstructed from the coefficients \\(\bm{a}\\) and the constant \\(t\\), so those are the only things we'll save for each equation.
- Given a public key which consists of some number of these equations, the LWE problem is to recover the secret vector of randomly selected integers \\(\bm{s}\\).

There are technically two different forms of the LWE problem, but they're mathematically equivalent as long as \\(q\\) is a prime number less than \\(n\\).

## Conclusion

While the math here can look a little scary, the basic ideas aren't very complicated: we've got a list of random integers that's *nearly* a solution to a long list of randomly generated linear equations. There's quite a bit more here that's worth looking into; it turns out that the LWE problem is very closely related to the [shortest vector problem](https://en.wikipedia.org/wiki/Lattice_problem#Shortest_vector_problem_(SVP)) and a number of other lattice problems.

The LWE problem is the basis for several important post-quantum cryptosystems, such as CKKS and TFHE. It's also the basis of the recently-standardized Kyber (aka ML-KEM) algorithm in [FIPS 203](https://csrc.nist.gov/pubs/fips/203/final). My real motivation for explaining this is [this year's Chipathon](../fhe-project-announcement), where I'm working on a coprocessor for TFHE, but I also just think it's pretty cool. Hopefully this helped it make a little bit of sense to you.

### See also
- The original LWE problem (and a small cryptosystem) was proposed by Regev in [this 2005 ACM paper](https://dl.acm.org/doi/10.1145/1060590.1060603), which is more mathy than I can currently understand. I can figure out bits of it, but it's mostly Greek to me (and I'm not just talking about all the \\(\lambda\\)'s and stuff).
- This excellent [video from Chalk Talk](https://www.youtube.com/watch?v=K026C5YaB3A) explains how LWE works very succinctly. If this explanation didn't make it click for you, go there next.
