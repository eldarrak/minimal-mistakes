---
layout: post
title: Setting priors on movement parameters (FLightR)
---
As any other Bayesian model FLightR uses priors on migration parameters. General rule for priors is that they should be selected wide enough to give data more freedom in affecting posteriors.
If for some reason you have decided to change migration priors note, that priors for the movement at time between twilight `i-1` and twilight `i` are written at the line `i` of the Index.tab object.

Prior distribution of migratory distances is assumed to follow a truncated normal distribution with parameters `M.mean`, `M.sd`, `a` and `b`. Note that these are distances between to subsequent twilights but not speed. 
M.mean and M.sd can be specified for every transition between twilights. 

```{r, eval=FALSE}
Index.tab$M.mean<- 300 # distance mu
Index.tab$M.sd<- 500 # distance sd
```
But truncation points are assumed to be fixed for all the transitions, so they are specified in the particle filter run:

```{r, eval=F}
Result<-run.particle.filter(all.in, ..., a=45, b=1500)
```
`a` is minimum distance flown. As far as our grid has 50 km between points 45 just means any movement. 
`b`  is maximum distance allowed to fly between twilights. It might depend on your species, but generally should not be set too short.

Migration probability is another prior parameter. It can be specified for every transition as

```{r, eval=FALSE}
Index.tab$Decision<-0.1 # prob of migration
```
There are also parameters for direction and concentration parameters of vonMises distribution. They might be changed from default even distribution, but no experiments was done with this so far. 

```{r, eval=FALSE}
Index.tab$Direction<- 0 # direction 0 - North
Index.tab$Kappa<-0 # distr concentration 0 means even
```

Posterior values for movement parameters may be found here:

```{r, eval=FALSE}
Result$Results$Movement.results
```
This is part of [FLightR Wiki](https://github.com/eldarrak/FLightR/wiki/FLightR)
