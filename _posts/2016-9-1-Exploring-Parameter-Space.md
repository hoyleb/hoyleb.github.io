---
layout: post
title: Efficiently exploring high dimensional parameter space with Hybrid
---

Often I have to explore some high dimensional parameter space, looking for the best position with respect to some likelihood function. This can be either minimising a chi-squared function or maximising a likelihood function. 

In this blog I'd like to document one routine I've tweaked, which is useful if you don't know a-priori where in your (potentially vast) parameter space a good fitting solution sits. Once you know a good starting point, then you might run a full [MCMC](http://dan.iel.fm/emcee/current/) chain in order to characterise the likelihood surface, so that you can quantify errors and degeneracies on your parameter values.

The routine is called  "Hybrid", and is based on this [research paper](http://arxiv.org/pdf/astro-ph/0602338v2.pdf) and [my buddy's PhD thesis](https://github.com/hoyleb/hoyleb.github.io/blob/master/images/JK_PhD_Thesis_v3.pdf.bz2).

The basic idea is that Hybrid is the combination of two routines. These are [Simulated Annealing](http://docs.scipy.org/doc/scipy-0.15.1/reference/generated/scipy.optimize.anneal.html) and [Particle Swarm Optimisation](http://pythonhosted.org/pyswarm/). Simulated Annealing is a fancy way of saying "standard MCMC with the probability of accepting a bad step inversely proportional to the number of steps taken". This acceptance value is captured in a temperature, which cools with run time. Particle Swarm Optimisation works by having lots of ant-like workers travelling around parameter space, and taking larger or smaller steps depending on what their neighbour ants are doing, and depending on which one of the swarm of ants is finding the current best position.

I've made a couple of tweaks to this routine, so that the initial velocities of each of the ants is different at the beginning. This sort of mimics "scout" ants, and "worker" ants. I've also allowed the temperature of the system to be flexible if it begins to be stuck in a poor position.

First let's install what we need to get hybrid working, then we can go through a worked example.

1) We need to install some dependencies:

```%>pip install pyDOE jobib emcee```

```jobib``` allows easy parallelisation across many cores
```emcee``` will be our defacto [and awesome] comparison routine
```pyDOE``` gives us a latin-hypercube which is a great way to populate initial walkers in a 
high volume parameter space.

2) Then download hybrid.py from here and add to your ```$PYTHONPATH```


Now let me show you what it can do, by means of a worked example. We'll use the same example as [here, by the emcee team](http://dan.iel.fm/emcee/current/user/line/). Be sure to check that excellent tutorial out first, and then come back here.

Right, let's modify the likelihood function ```lnprior``` a bit, so that we can "count" how many calls to the likelihood function we have had.
```
count = 0 
def lnlike(theta, x, y, yerr):
    #minor addition to count how many calls to likelihood function.
    global count
    count +=1
    m, b, lnf = theta
    model = m * x + b
    inv_sigma2 = 1.0/(yerr**2 + model**2*np.exp(2*lnf))
    return -0.5*(np.sum((y-model)**2*inv_sigma2 - np.log(inv_sigma2)))
```

Next, let's re-run their optimise step and see how many calls to the likelihood function it takes:

```
import scipy.optimize as op
nll = lambda *args: -lnlike(*args)
count = 0
result = op.minimize(nll, [m_true, b_true, np.log(f_true)], args=(x, y, yerr))
m_ml, b_ml, lnf_ml = result["x"]
print ("number of calls optimise {:}".format(count))
```

Now let's pretend we don't know the best starting positions. Let's choose a few random starting position from within the prior space. To do this, let's construct the parameter limits as an array, which we can get from the function ```lnprior```, namely

```
def lnprior(theta):
    count +=1
    m, b, lnf = theta
    if -5.0 < m < 0.5 and 0.0 < b < 10.0 and -10.0 < lnf < 1.0:
        return 0.0
    return -np.inf
```
So that the extracted prior range looks like this:
```
#paramsMinMax = [[min(m), max(m)], [min(b), max(b)], [min(lnf), max(lnf)]]
paramsMinMax = np.array([[-5.0 ,0.5], [0.0, 10.0], [-10.0, 1.0]])
```

Now we can set up the hybrid code, and use a helper function to get a random set of starting positions. We'll also set the number of walkers that we want to use to 6 (=3 params * 2)

```
from hybrid import Hybrid
hy = Hybrid(lnprior, paramsMinMax, num_walkers=6)
print ("Inital starting positions", hy.starting_positions)
```
Under the hood we use the Latin-Hyper cube routine as a neat way to maximially explore the prior space, with the number of walkers we want to put down.


Rather than giving the final emcee call a good idea about the best initial starting positions, we are going to try and figure them out using ```hybrid```.

![_config.yml]({{ site.baseurl }}/images/config.png)


 ****