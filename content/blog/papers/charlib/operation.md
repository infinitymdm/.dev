---
title: 'Part 2: How it Works'
date: 2024-08-07T14:16:52-05:00
draft: true
math: true
---

Now that we understand standard cell characterization, we can actually look at the paper. As it
turns out, understanding the background is the hardest part here.

<!--more-->

## CharLib: An Open Source Standard Cell Library Characterizer

This paper, published in the proceedings of the 2024 Midwest Symposium on Circiuts and Systems,
describes a new standard cell characterizer written in the Python programming language. It's open
source and designed to be simple to use.

The innovation here isn't in the details of the characterization process. That's mostly the same as
what other open source characterizers have been doing for years. Instead, CharLib introduces a new
way of handling standard cell information with the goal of making characterization easier and more
consistent.

### Section I: A shift in perspective

Existing characterizers follow a typical paradigm. You more or less tell the tool where each
individual cell is, then load the tool with information about that cell, then tell it to run a
specific procedure. This is a pretty manual process, with the tool only handling the actual
simulation automatically.

For example, if you wanted to characterize a cell library using Cadence tools, you would write a
script using Cadence Liberate's domain-specific commands. There are commands for importing cell
data, defining paths through cells, defining test conditions, and much more. Learning all those
commands takes time, and characterization has to be kicked off manually.

> TODO: concrete example of Cadence characterization script. 

CharLib, instead, tries to automate the entire process. Cell information is treated like metadata,
which can be stored with cell netlists or in a centralized configuration file for the whole cell
library. Instead of configuring the tool every time you run characterization, you describe your
cell library once, then you can use that configuration any time you want to run characterization.
It's a shift from prescriptive programming to descriptive.

One of the big advantages of this is that you can store cell information relavant to
characterization alongside your cells. This makes everything simple and bite-sized: you don't have
to have one massive script that handles all the cells in the library. (Maybe someday you'll even be
able to store characterization metadata in the cell netlist, using a special comment format or
something. That could be pretty cool.)

CharLib also tries to minimize the amount of work you have to do by letting you set library-wide
defaults that cascade down to all cells. There is some information that's different for every cell,
of course. Those items are required to be documented on each cell. But everything else - even test
conditions like slew rates and capacitive loads - can be easily set once for the whole library. You
can still override library defaults by specifying settings on a per-cell basis, of course.

> TODO: concrete example of charlib configuration file

That's not to say that CharLib perfectly automates everything, of course. You still have to write
a config file for your cell library, and that means you'll still need all the same information that
would have gone into a script for a different tool. The difference is that instead of learning
commands specific to the tool, you're documenting your cell library in a way that CharLib knows how
to read. Since this is simple, descriptive information, there's no reason that other tools can't
use this information as well. Everyone benefits from documentation, but only a single tool can use
a script.

### Section II: Nuts and Bolts

When it comes to characterization itself, CharLib doesn't do anything new. It uses tried-and-true
methods to characterize cells, leaning on the work of previous open-source characterizers such as
[libretto](https://github.com/snishizawa/libretto) and
[lctime](https://codeberg.org/librecell/lctime). But just for fun, let's take a closer look at how
it works.

The characterization process for any given cell happens in four steps. Each cell in the library has
to go through this process.
1. Identify Test Arcs
2. Measure Input Capacitance
3. Measure Delays
4. Collect and Present Results

Let's go through what each of those means.

#### Test Arc Identification

For starters, what's a test arc? Simply put, a test arc is a path through a cell (from one input to
one output) where all other cell inputs are set to nonmasking conditions and we can observe the
state transition we want. Even for a simple explanation, that's pretty jargon-dense, so let's look
at an example.

Let's say you want to characterize an OR cell. You start with the path from input A to the output,
Y. But what do you do with input B? If you leave it disconnected, your simulation won't represent
real-world usage well. But you can't just set it to a random value, because if you set it to logic
1, the OR cell will always output 1. You have to set input B to zero in order to observe how the OR
cell affects a signal on input A. So your test arc is the path from A to Y, with input B held
stable at logic 0.

> TODO: image of OR gate being tested, with masking and nonmasking conditions shown

Which test arcs are valid depends on the logic that the cell implements. This points to a very
simple method for identifying valid test configurations: comparing rows of the truth table for the
cell's logical function.

> TODO: insert image of truth table with rows compared to identify test arcs

CharLib's algorithm for identifying valid test arcs first builds a truth table based on the cell's
function (which is part of the cell metadata), then steps through the rows of that table to
identify pairs of rows where changing a single input produces a change in the cell output. This
works for arbitrary combinational logic functions. Sequential cells are a little more complex
because of their internal state, but the same general principle applies. Here's a flowchart showing
the full process.

![Test arc identification flowchart](../test_arc_flowchart.svg  "This is about as simple as it gets. This stuff is just plain difficult.")

Once we've figured out *how* to test our cell, we take a brief aside to measure cell capacitance.

#### Measuring Input Capacitance

To measure capacitance, we model (aka pretend) that the cell is a capacitor connected to ground.
