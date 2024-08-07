---
title: 'Laymanized - CharLib: An Open Source Standard Cell Library Characterizer'
date: 2024-08-07T14:16:52-05:00
draft: true
categories:
    - laymanized
    - paper review
tags:
    - open source
    - standard cell
    - characterization
    - eda tool
    - foss
---

In this first entry in the [Laymanized]({{< ref "/categories/laymanized" >}}) series, we'll take a
look at my first publication and dive into the topic of standard cell characterization.


<!--more-->

### First, a brief Disclaimer

I do NOT intend for this site to be exclusively a showcase for my own work. I want to talk about
all sorts of stuff from all sorts of people, like [David Harvey's paper on fast NTT](https://www.sciencedirect.com/science/article/pii/S0747717113001181)
or [David Harris's taxanomy for parallel prefix networks](https://ieeexplore.ieee.org/abstract/document/1292373). 
(Also, I'm not just going cover works from people named David H. This was a coincidence, I 
promise). However, if I'm going to try to explain VLSI concepts as difficult and complex as
characterization, it's nice to start with stuff I know. Ok, disclaimer over. Let's get into this.
We've got a lot of ground to cover.

### Some Assumptions

I'm going to assume that you, as an intellectual, have some knowledge going into this. Probably a
big part of why you clicked on this article, right?

{{< figure src="https://imgs.xkcd.com/comics/when_you_assume.png" link="https://xkcd.com/1339/" caption="Yes, I know what happens when I assume. / xkcd.com" >}}

Just to be sure, before we get started:

- You should have at least a basic understanding of Boolean logic
- You should know your logic gates (NOT, AND, OR, XOR, etc.)
- You should be able to identify the components of a circuit diagram (including logic gates)
- You should know how to read a line graph
- You should be aware of hardware descriptons languages such as verilog or VHDL (but don't worry,
we won't be working with any code here)

Now that that's out of the way...

## Let's Start with a bit of Background

Even if you've studied electronics at a university level, odds are pretty good that this is your
first time hearing the term "standard cell", let alone paired with "library" and "characterizer".
We'll begin by breaking down what those terms mean.

### What are Standard Cells?

In the simplest terms, **standard cells are digital building blocks (such as logic gates)
implemented in a consistent, standardized way**. Perhaps an analogy will help.

> Imagine you're building a house. You've got the framing up and the roof on, and now it's time to
cover up the exterior walls with siding. You can hire an expert stonemason who will build you a
beautiful natural stone veneer, but the job will take a month, and they charge expensive rates for
their expertise. Or instead, you can hire a bricklayer who will have the job done by the end of the
week for a fraction of the cost. 
>
> See the difference here? The stonemason has to work with the shapes of the stone he or she is
given. The bricklayer doesn't have to worry about that at all - all the bricks have the same
dimensions, so it doesn't matter that brick A is next to brick C instead of brick B. As a result,
the brickwork is quick and cheap by comparison, and the bricklayer doesn't have to be a master of
his craft (though it certainly helps).

It's the same way with electronics. Standard cells (like bricks) have a fixed height, so we can
place them in rows (like bricks) to get a clean, organized design. A master engineer (like the
stonemason) could likely do a better job without those rows, but it would probably take them so
long that it isn't worth the cost and effort. Besides that, using fixed-height cells has another
advantage: we can route power and ground along the top and bottom of each row of cells. This solves
a lot of headaches for distributing power to complex circuit designs.

If that analogy didn't make sense to you, maybe this way of thinking about it will. Think of
standard cells like Lego bricks: they all have the same height, and they connect in predictable
ways. **They're the same logic gates you know and love, just a bit more... standard**.

TODO: Add image of a standard cell (or a design using standard cells) next to a picture of a brick.
Maybe use the "corporate needs you to find the difference" meme format?

When using standard cells, we don't have to think about physical design problems, such as how
transistor M1 connects to to transistor M3 with wires that are a few nanometers wide. Instead we
get the luxury of abstraction: we can just think about how the logic gates connect. This seriously
reduces the amount of work involved in implementing a design, and has saved this engineer many
headaches.

There are a lot of other problems that standard cells solve for us, but we won't go into that here.
That's a topic for another post.

### Ok, but what does a Library have to do with it?

**A standard cell library is a collection of standard cells that are made to work well together**.
They'll all have the same height, and you'll have cells for all the major logic components: AND,
OR, XOR, NAND, et cetera. You'll also have things like inverters, buffers, and flip-flops. Think of
a standard cell library like a toolbox full of devices for working with digital logic.

Seems pretty straightforward, right? But consider how powerful this idea is! With a standard cell
library, I can take *any digital logic design* and turn it into a real physical design. We even
have automated tools to do this. You hand the tool a standard cell library and some HDL, and the
tool "synthesizes" your design using the cells in the standard cell library.

There's a little more to it than just replacing all the &'s in your HDL, of course.
