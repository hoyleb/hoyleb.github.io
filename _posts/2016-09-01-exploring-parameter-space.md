------
layout: post
title: Efficiently exploring high dimensional parameter space with Hybrid
---

Often I have to explore some high dimensional parameter space, looking for the best position with respect to some likelihood function. This can be either minimising a chi-squared function or maximising a likelihood function. 

In this blog I'd like to document one routine I've tweaked, which is useful if you don't know a-priori where in your (potentially vast) parameter space a good fitting solution sits. Once you know a good starting point, then you might run a full MCMC chain in order to characterise the likelihood surface, so that you can quantify errors and degeneracies on your parameter values.

The routine is called  "Hybrid", and is based on this research paper http://arxiv.org/pdf/astro-ph/0602338v2.pdf and [my buddies' PhD thesis](https://github.com/hoyleb/hoyleb.github.io/blob/master/images/JK_PhD_Thesis_v3.pdf.bz2) 

The basic idea is that Hybrid is the combination (or hybrid) of two routines. These are Simulated Annealing and Particle Swarm Optimisation. Simulated Annealing is a fancy way of saying "standard MCMC with the probability of accepting a bad step inversely proportional to the number of steps taken". This acceptance value is captured in a temperature, which cools with run time. Particle Swarm Optimisation works by having lots of ant-like workers travelling around through parameter space, and taking larger or smaller steps depending on what their neighbours are doing, and depending on which one of the swarm of ants is finding the current position.

I've made a couple of tweaks to this routine, so that the initial velocities of each of the ants is different at the beginning. This sort of mimics "scout" ants, and "worker" ants. I've also allowed the temperature of the system to be flexible if it begins to be stuck in a poor position.

Enough talk, let me show you what it can do, by means of a worked example.

First we need to install some dependencies:
```%>pip install pyDOE jobib emcee```

```jobib``` allows easy parrelisation across many cores
```emcee``` will be our defacto [and awesome] comparison routine
```pyDOE``` gives us a latin-hypercube which is a great way to populate initial walkers in a high volume parameter space.

Then add hybrid.py to you path

![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.
