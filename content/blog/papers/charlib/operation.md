---
title: 'CharLib | Part 2: How it Works'
date: 2025-01-07T14:16:52-05:00
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
math: true
ShowToc: true
---

Now that we understand standard cell characterization, we can actually look at the paper. As it
turns out, understanding the background is the hardest part here.

<!--more-->

> *Feeling a bit lost?* This article is the second part in a series on the paper "Charlib: An Open
Source Standard Cell Library Characterizer". If you're looking for the other parts, or the paper
itself, [click here](..).

## CharLib: An Open Source Standard Cell Library Characterizer

This paper, published in the proceedings of the 2024 Midwest Symposium on Circiuts and Systems,
describes a new standard cell characterizer written in the Python programming language. It's open
source and designed to be simple to use.

The innovation here isn't in the details of the characterization process. That's mostly the same as
what other open source characterizers have been doing for years. Instead, CharLib introduces a new
way of handling standard cell information with the goal of making characterization easier and more
consistent.

### Section I: A shift in perspective

#### The Current Method: Tool-Specific Scripts

Existing characterizers follow a typical paradigm. You more or less tell the tool where each
individual cell is, then load the tool with information about that cell, then tell it to run a
specific procedure. This is a pretty manual process, with the tool only handling the actual
simulation automatically.

For example, if you wanted to characterize a cell library using Cadence tools, you would write a
script using Cadence Liberate's domain-specific commands. There are commands for importing cell
data, defining paths through cells, defining test conditions, and much more. Learning all those
commands takes time, and characterization has to be kicked off manually. Take a look at this
excerpt modified from some [example liberate scripts](https://gitee.com/chaujohnthan/liberate/blob/master/char.tcl):

```tcl
### Define temperature and default voltage ###
set_operating_condition -voltage 1.5 -temp 125

## Load template information for each cell ##
source ${rundir}/TEMPLATE/template_example.tcl

set_var extsim_cmd "spectre2"

## Load Spice models and subckts ##
set spicefiles $rundir/MODELS/include_SS.sp
foreach cell $cells {
    lappend spicefiles ${rundir}/NETLIST/${cell}.sp
}
read_spice $spicefiles

## Characterize the library for NLDM (default)
char_library -thread 1 -extsim spectre2 -cells {INVX1}

## Save characterization database for post-processing ##
write_ldb ${rundir}/LDB/example.ldb

## Generate a .lib with ccs, ecsm ###
write_library ${rundir}/LIBRARY/example_nldm.lib
```

While the code itself is pretty well commented, I totally get it if your eyes glazed over the
moment you saw that monospaced font. Basically the above code does the following:
1. Set up a circuit simulator
2. Load a library of standard cells (by calling a different script)
3. Characterize the standard cell library

That's a lot of code for such a small task. And that's not even half the code you have to write to
run characterization; note the call to another script, `template_example.tcl`, where all the cell
slew rates and loads are set up. You can find those other scripts in the same repository linked up
above. Credit where credit is due though; Cadence's tools do the job, and do it very well. The
syntax is not exactly concise, but it's not too hard to tell what it's doing, either.

All that said, there are a few things we wanted to improve on in designing CharLib's interface:
- You should be able to run characterization from a single command-line call, without explicitly
setting up the underlying tools every time (i.e. you shouldn't have to write scripts specifically
for the tool).
- You shouldn't have to rewrite "boilerplate" code every time you want to run characterization.
It's obvious that the tool is going to have to do things like load cells every time you
characterize, so that should be integrated into the process somehow.
- You should be able to describe a cell library once and for all time, then reuse that description
(even for other tools).

With these goals in mind, it's easy to see why CharLib takes a different approach.

#### The CharLib Method: Tool-Agnostic Description

CharLib instead operates by reading a description of the cell library. Cell information is treated
like metadata which can be stored with cell netlists or in a centralized configuration file for the
whole cell library. Instead of configuring the tool every time you run characterization, you
describe your cell library once. Then you can use that configuration any time you want to
characterize your library. It's a shift from prescriptive programming to descriptive.

One of the big advantages of this is that you can store cell information relavant to
characterization alongside your cells. This makes everything simple and bite-sized: you don't have
to have one massive script (or a script that calls other scripts that calls other scripts... you
get the picture) that handles all the cells in the library. (Eventually CharLib may even let you
store cell configuration right in the cell netlist using a special comment format! But I'm getting
ahead of myself - watch the [CharLib Github](https://github.com/stineje/CharLib) if you're
interested in new features or releases.)

CharLib also tries to minimize the amount of work you have to do by letting you set library-wide
defaults that cascade down to all cells. There is some information that's different for every cell,
of course. Those items are required to be documented on each cell. But everything else - even test
conditions like slew rates and capacitive loads - can be easily set once for the whole library. You
can still override library defaults by specifying settings on a per-cell basis, of course.

For comparison, here's a complete example of a CharLib configuration file for library with a single
inverter:

```yml
settings:
    lib_name: gf180mcu_osu_sc
    cell_defaults:
        models:
            - gf180_temp/models/sm141064.ngspice typical
            - gf180_temp/models/design.ngspice
        slews:  [0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1]
        loads:  [0.005, 0.01, 0.025, 0.05, 0.1]

cells:
    gf180mcu_osu_sc_gp12t3v3__inv_1:
        netlist:    gf180_temp/cells/gf180mcu_osu_sc_gp12t3v3__inv_1.spice
        inputs:     [A]
        outputs:    ['Y']
        functions:  [Y=~A]
```

This might also look like a lot of code for a small task, but consider that there are no other
scripts hiding in the background: all the details are right here. Technically this code doesn't
even *do* anything on its own - it just describes the cell library and simulation details. But if
you pull up a terminal and run `charlib run /path/to/this/config/file`, CharLib will ingest this
configuration file and automatically characterize the cell.

That's not to say that CharLib perfectly automates everything, of course. You still have to write
a config file for your cell library, and that means you'll still need all the same information that
would have gone into a script for any other tool. The difference is that instead of learning
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
2. Measure Input Capacitances
3. Measure Delays
4. Collect and Present Results

Let's go through what each of those means.

#### Test Arc Identification

For starters, what's a test arc? Simply put, a test arc is a path through a cell, from one input
pin to one output. A good test arc will tests how changes to that one input affect the output,
without letting other pins affect the test. That means a test arc has to capture
1. Which input we're testing
2. Which output we're testing
3. A signal to feed into the input that will give us something to observe on the output
4. A setup for all other inputs so that observation comes through unimpeded

Item 4 here is called the "nonmasking conditions" for the test arc, because it's the setup that
will not hide, or "mask", the input signal from causing some observable change on the output.

That's pretty jargon-dense, so let's look at an example.

> Let's say you want to characterize an OR cell. You start with the path from input A to the
output, Y. But what do you do with input B? If you leave it disconnected, your simulation won't
represent real-world usage well. But you can't just set it to a random value, because if you set it
to logic 1, the OR cell will always output 1. You have to set input B to zero in order to observe
how the OR cell affects a signal on input A. So your test arc is the path from A to Y, with B=0 as
the nonmasking condition.

<!-- TODO: image of OR gate being tested, with masking and nonmasking conditions shown -->

Which test arcs are valid depends on the logic that the cell implements. This points to a very
simple method for identifying valid test configurations: comparing rows of the truth table for the
cell's logical function.

CharLib's algorithm for identifying valid test arcs first builds a truth table based on the cell's
function (which is part of the cell metadata), then steps through the rows of that table to
identify pairs of rows where changing a single input produces a change in the cell output. This
works for arbitrary combinational logic functions. Sequential cells are a little more complex
because of their internal state, but the same general principle applies. Here's a flowchart showing
the full process.

![Test arc identification flowchart](../test_arc_flowchart.svg  "This is about as simple as it gets. This stuff is just complicated.")

Once we've figured out *how* to test our cell, we take a brief aside to measure cell capacitance.

#### Measuring Input Capacitance

To measure capacitance, we model (a.k.a. pretend) that the cell is a capacitor connected to ground.
Then we do some funny business.

I'm not going to try to explain Laplace transforms or Fourier analysis here; [other folks have done
a really good job explaining those topics](https://www.youtube.com/watch?v=spUNpyF58BY). The gist
of the approach here is this: if you generate an alternating current waveform with a particular
frequency, then pass it through a capacitor, it will come out the other end slightly altered. With
a little calculus, we can find the capacitance from the altered signal and our input current:

$$
    C = i_0 \frac{d}{d{s}} \left( \frac{1}{v(s)} \right)
$$

Don't worry too much about the math; suffice it to say that circuit simulators are really good at
measuring voltage and current, and they do most of the work for us here. Under the hood, CharLib
is just plugging in numbers and letting those tools give us the results.

#### Delay Characterization

Ok, at this stage we've figured out *how* to test our circuits (test arcs), and we've made a few
initial measurements using the standard cell spice files, so we know they work. Now it's time to do
the real work: measuring delays.

The goal here is actually pretty simple: if we switch one of the circuit inputs from low to high
(or vice versa) we want to measure how long it takes for that change to start showing up on the
cell output (\\(t_{prop}\\)), as well as how long the output takes to change state
(\\(t_{trans}\\)). However, as discussed in [part 1](../background/#changing-conditions), we have
to take lots of measurements with varying \\(t_{slew}\\) and \\(c_{load}\\) in order to get a good
model.

##### Division of Labor

At this stage, we have a list of standard cells, each of which has a list of test arcs, a list of
slew rates, and a list of capacitive load values. For each cell, we want to test all combinations
of these three groups. If you know anything about [combinatorics](https://en.wikipedia.org/wiki/Combinatorics),
you'll recognize that this quickly balloons into a **lot** of simulations. There isn't a clear way
to avoid this, unfortunately (but perhaps this points towards an area for future research!).
However, we can take advantage of the resources available to us: [parallel processing](https://en.wikipedia.org/wiki/Parallel_computing)!

Instead of running each cell one at a time, we can kick off multiple cells at the same time as
distinct processes. Software on the computer called a "scheduler" decides what order to execute
these processes in. While all the test arcs, slews, and loads for each cell are still run
back-to-back (a.k.a "sequentially"), multiple cells can now be characterized at the same time. This
works even better on modern multi-core processors, which are really multiple processors glued
together. Each processor core can run multiple programs while other cores are doing their own
thing. For us, that means more cells can be characterized at once.

> Right now, characterization just gets divided up into a single process per cell. Ideally, it
should be divvied up into multiple processes per cell as well. Every combination of test arc, slew
rate, and load could be its own process. That might lead to big efficiency improvements and faster
characterization! Keep an eye out for this feature in future versions of CharLib.

Now that we've talked through how we're going to break down the processing, let's get into the
details that actually matter: circuit simulation.

##### Simulation

Let's say I'm a characterization program kicked off by CharLib. I've been handed a [SPICE](https://en.wikipedia.org/wiki/SPICE)
file that represents a combinational cell's behavioral model, a test arc describing what value
each input should be set to and which outputs to watch, a capacitive load to place on that output,
and a slew rate to use for my input signal. Now what?

![Delay simulation inputs before wiring up to harness](../delay_wiring_0.svg "Cool beans, now I've got a bunch of stuff. What do I do with it?")

Well, the first step is to wire up the circuit. CharLib does this by creating a "`Harness`" that
handles the wiring and can work with any slew rate or load values. If you're particularly
technically inclined, you can [take a look at the code here](https://github.com/stineje/CharLib/blob/main/charlib/characterizer/Harness.py).
The `Harness` unpacks the test arc, wiring the inputs and outputs up to voltage sources and load
capacitors respectively. Nonmasking conditions are also applied at this stage by hooking up logic
1's to high voltage (\\(V_{DD}\\)) and logic 0's to ground. Any cell outputs that aren't being tested
get wired to a pull-down resistor.

> Note: There are some cases where logic 0 doesn't mean 0 volts. For simplicity, we're not going to
worry about that sort of thing. It's pretty rare anyways, at least for digital logic circuits, but
it does happen sometimes.

![Delay simulation circuit connected with wiring harness](../delay_wiring_1.svg "My circuit is all wired up now! What's next?")

Now we have a complete circuit. We feed a test signal into \\(v_{in}\\) that slews from 0 to 1 (or
vice versa) over \\(t_{slew}\\) nanoseconds, then watch the voltage \\(v_{out}\\) across the load
to see when it switches. We'll take two measurements here. \\(t_{prop}\\) measures how long it
takes the signal to propagate through the cell, and \\(t_{trans}\\) measures how long it takes the
output to change state.

![Delay plots showing t_prop and t_trans measurements](../delay_plot.svg "Remember this chart from part 1? Bet you didn't think it was important last time.")

We'll repeat this process for every capacitive load we're given, with a rising and a falling input
signal. In the charts below, these values are the "Fanout"s shown in the legend. You can see how
changing the capacitance makes a huge difference in the delay. These are actual simulation charts
generated by running CharLib.

![Simulated delay chart for a GF180 buffer](../gf180_buf_rise_0.1.svg)
![Simulated delay chart for a GF180 buffer](../gf180_buf_fall_0.1.svg "These are actual simulation results. Did I mention CharLib can produce these automatically?")

> Wondering what those dashed grey lines are? Those are what we call "logic thresholds". In digital
logic, a 1 isn't really 1 volt, and a zero isn't really 0 volts. Instead, we say that a voltage
above maybe 80% of our overall circuit voltage is a 1, and below 20% or so is a zero. Anything in
between is "indeterminate"; there's no telling whether the circuit will treat it as a 1 or a 0. We
usually want to avoid that sort of thing.

Once we've taken all the load measurements for this slew rate, we switch to a new slew rate and
repeat. By the end of all the simulations, we have a whole bevy of timing measurements that we can
stick in a table indexed by slew rate and load capacitance. These "lookup tables" are the results
of characterization. Mission accomplished!

While the above example only covered combinational cell delay characterization, I'm not going to go
into detail on sequential cell delay characterization here. It has a whole bunch of quirks and
extra nonmasking conditions, but it's generally the same as combinational cell characterization.
Just more complicated and with a few extra measurements. If you want to learn more about that, you
ought to [read the paper](https://ieeexplore.ieee.org/document/10658687).

Anyways, now that we have a better handle on how delay characterization works under the hood, we
can talk about how we deal with the results of all those simulations.

##### Liberty Files

Once we've run all the simulations for all the cells, slew rates, and loads we have, how do we
store and communicate those results? Lucky for us, smart people have been thinking about this for
several decades. They came up with a file format called a liberty file which is like a specialized
form of [JSON](https://en.wikipedia.org/wiki/JSON). It's standardized and pretty well-defined,
though I'll admit it's pretty confusing when you first look at it. The standard is about a million
pages long and not a fun read, but just in case you want to look at it, [here you go](https://people.eecs.berkeley.edu/~alanmi/publications/other/liberty13_03.pdf).

The important thing here is that there's a standard way to write all the lookup tables we've
collected out to a file. Other tools can then use that file to synthesize designs from the
characterized PDK or whatever. CharLib uses [liberty-parser](https://pypi.org/project/liberty-parser/)
to build liberty files.

### Conclusion

I hope that learning how CharLib works hasn't been too overwhelming. Taken altogether, this is a
very complex piece of software. But when you look closer, it's not doing anything that's really all
that complicated. The processes that it uses under the hood are all pretty simple - it's just a
culmination of a bunch of little good ideas.

Keep in mind also that characterization is just a tiny part of the digital design process. As
complex as CharLib seems, it's only one link in a much larger software supply chain.

As always, if there are parts of this you find confusing, reread & reach out! [Send me an email](mailto:marcus@infinitymdm.dev)
with your questions. I'll do my best to clarify things for you, and update this article with more
detail when needed.
