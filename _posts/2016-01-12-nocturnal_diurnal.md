---
title: Limit migration to nocturnal or diurnal (FLightR)
---
Sometimes we have strong biological assumption on migration, e.g. we know that our animal is a nocturnal migrant. There could be two approaches to work with such a tag:

1. Rerun model with probabilities of over day migration set to 0 (recommended);
2. Use no movement model and just get midday positions by multiplying consequent dawn and dusk positions.

## Scenario 1 - limit day migration to 0
### load data

```{r}
library(FLightR)
# extract initial data from the results object.
load("Result.RData")

all.in<-Result
all.in<-all.in[-which(names(all.in)=="Results")]
```
### restrict probabilities
It is very important to realize here that in ```all.in$Indices$Matrix.Index.Table``` all the movement indices should be related to movements before the current time. For example the first line in the Matrix.Index.Table will show likelihood for the movements between starting point and first twilight.

To properly set 0 for some migrations we should use another index - Main.Index

```{r}
all.in$Indices$Matrix.Index.Table$Decision[all.in$Indices$Main.Index$proposal.index=='Dusk']<-0
```
### rerun the particle filter

```{r}
nParticles=1e6
Threads= detectCores()-1
a= Sys.time()
Result<-run.particle.filter(all.in, save.Res=F, cpus=min(Threads,6),
                            nParticles=nParticles, known.last=TRUE,
                            precision.sd=25, save.memory=T, k=NA,
                            parallel=T,  plot=T, prefix="pf",
                            extend.prefix=T, cluster.type="SOCK",
                            a=45, b=1500, L=90, adaptive.resampling=0.99, check.outliers=F)
b= Sys.time()
b-a
save(Result, file="Result.nocturnal.migration.RData")
```

## Scenario 2 - no movement model
We can just multiply probabilities of neighboring dusks and dawns. This will definitely be less precise than scenario 1, as we will not use the rest of the points to improve the positions.

```{r}
Days<-all.in$Indices$Main.Index$Biol.Prev[all.in$Indices$Main.Index$proposal.index=='Dusk']
if (Days[1]==1) Days<-Days[-1] # we want to exclude day if we started from the evening
```
### plot
my.golden.colors <- colorRampPalette(c("white","#FF7100"))
par(mfrow=c(3,3), ask=T)
for (t in 1:length(Days)) {
  # ok now I want to see how stable my estimates are.
  Distribution<-apply(all.in$Spatial$Phys.Mat[,(Days[t]-1):Days[t]],1,  FUN=prod)
  Max<-which.max(Distribution)
  image.plot(as.image(Distribution,
             x=all.in$Spatial$Grid[,1:2], nrow=60, ncol=60),
             col=my.golden.colors(64), main=paste(mean(all.in$Indices$Matrix.Index.Table$time[(Days[t]-1):Days[t]])))
  abline(h=all.in$Spatial$Grid[ Max,2])
  abline(v=all.in$Spatial$Grid[ Max,1])
  library(maps)
  map('world', add=T)
  map('state', add=T)
}
```

## get daily Modes
```{r}
no_movement_model<-data.frame()

for (t in 1:length(Days)) {
  # ok now I want to see how stable my estimates are.
  Distribution<-apply(all.in$Spatial$Phys.Mat[,(Days[t]-1):Days[t]],1,  FUN=prod)
  Max<-which.max(Distribution)
 
  no_movement_model<-rbind(no_movement_model, data.frame(Lon=all.in$Spatial$Grid[ Max,1], Lat=all.in$Spatial$Grid[ Max,2], Time=mean(all.in$Indices$Matrix.Index.Table$time[(Days[t]-1):Days[t]])))
  
}
```
### plot daily modes by latitude
```{r}
par(mfrow=c(1,2))
plot(Lon~Time, data=no_movement_model, pch="+")
plot(Lat~Time, data=no_movement_model, pch="+")
```
of course Scenario 2 results are much worse than Scenario 1.

This is part of [FLightR Wiki](https://github.com/eldarrak/FLightR/wiki/FLightR)
