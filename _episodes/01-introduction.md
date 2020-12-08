---
title: "Introduction"
teaching: 0
exercises: 0
questions:
- "What is pileup? Why should we care about it?"
objectives:
- "Understand the concepts of bunch crossing, pileup, and pileup modeling."
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

## What is pileup?

- bunch crossing
- bx rate during run 1 and run 2
- number of simultaneous interactions - example calculation
- number of pileup
- difference w.r.t. underlying event
- comparison of pileup in run1 vs. run2, larger effect during run 2

## Why should we care about pileup?

- how pileup shows up in detector: extra junk, tracks, jets, you name it
- affects everything from trigger rates to reconstruction, e.g. tracking and jet clustering
- we try to make pileup resistant algorithms but usually there is some residual pileup dependence
- in-time-pileup vs. out-of-time pileup

## Measuring pileup in data

- basic idea: measure luminosity, calculate cross section, infere pileup

## Modeling pileup in simulation

- we need to simulate pileup if we want simulations to match data
- basic idea idea: how do we simulate it? by mixing pileup events with the hard processes with some randomness
- exact pileup distribution can depend on what data samples you select; so we need to reweight the simulation (or maybe we simulated already before the data was collected!); these are called pileup corrections

## Outline of the exercise

- learning goal: how to check pileup distributions of data and simulated samples, and correct them
- practical example: HTT analysis

{% include links.md %}

