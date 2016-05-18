---
title: Cannot allocate vector of size XX Gb (FLightR)
---

Current implementation of FLightR (0.3.6) uses a lot of RAM for the particle filter. If you have got this message it means you have somehow decrease it (or move to a better workstation). There are two main solutions, decrease spatial extent 
or amount of threads particle filter uses:

  
## Decrease spatial extent

a) just change xlim, ylim.

b) if your bird cannot occur over water, you can just exclude these points, something like

```{r}
Land<-create.land.mask(All.Points.Focus, distance=25000) # distance from grid point to the closest body of land

Grid<-cbind(All.Points.Focus, Land=Land)
Grid<-Grid[Land==1,]

plot(Grid, type="n")
map('state',add=TRUE, lwd=1,  col=grey(0.5))
map('world',add=TRUE, lwd=1.5,  col=grey(0.8))
abline(v=start[1])
abline(h=start[2])
points(Grid[Grid[,1:2], pch=".", col="red", cex=2) 
```

_Note: that the same function can be used to create behavioural masks._

c) Draw a polygon based on your prior knowledge and exclude points that are not in the polygon (function `over()` works fine for this).

## Decrease amount of threads
R does not have shared RAM, so each of the threads will use separate chink of RAM. Before running particle filter (`run.particle.filter()`) we select how many threads we want to use with a function:

```{r}
Threads= detectCores()-1
```

Change it to 

```{r}
Threads=3
```

Or less, depending on amount of RAM you have.

This is part of [FLightR Wiki](https://github.com/eldarrak/FLightR/wiki/FLightR)
