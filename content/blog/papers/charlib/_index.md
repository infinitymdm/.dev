---
title: 'Laymanized: "CharLib: An Open Source Standard Cell Library Characterizer"'
date: 2024-08-07T14:16:52-05:00
categories:
    - laymanized
    - paper review
tags:
    - open source
    - standard cell
    - characterization
    - tools
    - FOSS
    - EDA
---

In this first entry in the [Laymanized]({{< ref "/categories/laymanized" >}}) series, we'll take a
look at my first publication and dive into the topic of standard cell characterization.

<!--more-->

### First, a brief Disclaimer

I do NOT intend for this site to be exclusively a showcase for my own work. I want to talk about
all sorts of stuff from all sorts of people, like [David Harvey's optimizations to NTT](https://www.sciencedirect.com/science/article/pii/S0747717113001181)
or [David Harris's taxonomy for parallel prefix networks](https://ieeexplore.ieee.org/abstract/document/1292373).
(Also, I'm not just going cover works from people named David H. This was a coincidence, I
promise). However, if I'm going to try to explain difficult and complex VLSI topics, it's nice
to start with stuff I know.

Ok, disclaimer over. Let's get into this.

### Organization

This entry is broken up into three parts.
- In the [first part](./background), we'll go over background information you'll need in order to
understand the paper.
- In the [next part](./operation), we'll look at how CharLib works under the hood.
- In the third and final part (coming soonish), we'll compare CharLib to other standard
cell characterization tools, and discuss future goals for the project.

CharLib is an open source project and can be found [here on GitHub](https://github.com/stineje/charlib).
I wrote the original code over the course of about a year, building on ideas from several existing
works and with lots of input from my advisor, Dr. James Stine.

[You can find the paper here in the proceedings of the 2024 IEEE Midwest Symposium on Circuits and Systems.](https://ieeexplore.ieee.org/abstract/document/10658687)
