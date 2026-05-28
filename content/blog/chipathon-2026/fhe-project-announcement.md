---
title: "I'm Participating in a Chipathon!"
date: 2026-05-26T11:03:50-05:00
tags:
    - open source
    - cryptography
    - FOSS
    - EDA
---

This summer, I'll be taking part in the [IEEE SSCS "PICO" Chipathon](https://sscs.ieee.org/technical-committees/tc-ose/sscs-pico-design-contest/)!

<!--more-->

I'm super excited to get to be a part of an event where all sorts of open-source EDA developers
come together and try to build real, working electronics hardware with fully transparent processes.
I'll be working with a scrappy team of passionate people from all over the globe on a really cool
project: building a hardware accelerator for post-quantum cryptographic systems.

## The Pitch

After commiting to participate in the chipathon design competition this year, I decided pretty
quickly that I wanted to do something involving [post-quantum cryptography](https://en.wikipedia.org/wiki/Post-quantum_cryptography)
(PQC).

### Why PQC?

Well, two reasons:
- I read a few books by Neal Stephenson (namely *Cryptonomicon* and *Anathem*) a few years ago.
Both of those stories have fascinated me and inspired me to learn more about how math changes the
world.
- I took Dr. Paul Fili's course on cryptography here at Oklahoma State. This was my first
introduction to a lot of complex math topics like number theory, abstract algebra, and topology,
and I really enjoyed it.

I posted a call to action in the Chipathon Discord channels: if people were interested in doing a
PQC project, they should reach out to me. It didn't take long for the DMs to start pouring in, and
before I knew it I had a crack team of technologists ready to get started.

## The Team

In no particular order, the team consists of the following individuals.

| Name              | Brief Bio |
| ----------------- | --------- |
| Me!               | You know who I am. Or if you don't you can learn more on the [About page](/about). |
| Rahul Tiwari      | Standard cell layout design engineer at Synopsys Hyderabad. |
| Emon Sarkar       | Grad student (like me!) studying ECE at the University of Waterloo. |
| Bhanuday Bhardwaj | Researcher in neuromorphic computing at the Center for Development of Advanced Computing (CDAC) in India. |
| Karan Mali        | Freelance hardware designer with experience in avionics comms who also did a stint at CDAC. |
| N Nishchit        | Undergrad (with some serious embedded programming experience) studying ECE at BMS College of Engineering. |

You can follow our progress on GitHub under the [Team Rocket Org](https://github.com/team-rocket-chipathon).
We'll also be updating [this GitHub issue](https://github.com/sscs-ose/sscs-chipathon-2026/issues/15)
weekly with project details and progress.

## The Project

Once we had the team together, we decided as a group that we wanted to build hardware that would be
useful for accelerating modern (or even future) cryptographic systems. There's been a lot of work
on the [Learning With Errors](https://en.wikipedia.org/wiki/Learning_with_errors) problem lately,
and we ended up deciding to focus our efforts on the [TFHE cryptosystem](https://tfhe.github.io/tfhe/).

Our goal is to build an [AXI](https://en.wikipedia.org/wiki/Advanced_eXtensible_Interface)-compatible
coprocessor to speed up TFHE operations. We intend to build a highly modular design which can be
easily plugged into existing open cores or parted out into useful IP blocks.

I'm incredibly excited to see where this goes. Hopefully as I learn more about TFHE I'll post some
of what I discover here. Stay tuned!
