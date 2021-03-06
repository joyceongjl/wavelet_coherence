---
output: 
  html_document:
    keep_md: true
---



## Introduction

This is a tutorial for analysis of wavelet coherence on biological time series data to investigate spatial (intraspecific) and community (interspecific) synchrony. First load these libraries:


```r
library(wsyn, warn.conflicts = F, quietly = T)#for wavelet analyses
## Warning: package 'wsyn' was built under R version 3.5.3
library(reshape2)#to use melt function
library(ggplot2)#to plot times series
library(dplyr, warn.conflicts = F, quietly = T)
library(corrplot)#to plot correlation matrix
## corrplot 0.84 loaded
```

## Generate random biological time series

4 random timeseries called sp1loc1, sp2loc1, sp1loc2, sp2loc2. 
sp1loc1 and sp2loc1 should have synchrony at 4 years, 
sp2loc1 and sp2loc2 should have synchrony at 8 years and is phase shifted.
Time series need to be box-cox transformed, note that in this example I use clev=1 because this is generated time series. With real data I normally use clev=5. Note that I am using previously generated data using the code below so the following results will be consistent


```r
times<-1:50
# sp1loc1<-sin(2*pi*times/4)+rnorm(50, mean=0, sd=0.5)+1.1
# sp2loc1<-sp1loc1 + sin(2*pi*times/8)+rnorm(50, mean=0, sd=0.5)
# sp1loc2<-sin(2*pi*times/3)+ rnorm(50, mean=0, sd=0.9)
# sp2loc2<-sp2loc1 + 3*sin(2*pi*times/8+4)+rnorm(50, mean=0, sd=0.5)+3.1
# 
# dat<-rbind(sp1loc1, sp2loc1, sp1loc2, sp2loc2)
# clndat<-cleandat(dat, times,1)$cdat
clndat<-as.matrix(read.csv("D:/snappers/clndat_example.csv", header=T, row.names=1))
clndat[,1:7]#see what the data looks like
##                 V1         V2         V3          V4         V5         V6
## sp1loc1  0.7688961  0.4375289 -1.9304311  0.42936368  0.9291764 -0.1104507
## sp2loc1  1.3386441  1.1223963 -0.5465356 -0.01220649  0.6870344 -1.5129432
## sp1loc2  1.5871646 -1.5308863 -0.6562685 -0.05386383 -0.4046720  0.4279434
## sp2loc2 -1.6017430 -0.6534549 -0.6721743  2.98325209  3.6846584  0.3380327
##                V7
## sp1loc1 -1.158409
## sp2loc1 -2.120506
## sp1loc2 -0.576478
## sp2loc2 -2.559620
```

## Plot of generated biological time series


```r
plot(times, clndat[1,]/3+1, type="l", xlab="time", ylab="time series index", ylim=c(0,6))
for (i in 2:dim(clndat)[1]){
  lines(times, clndat[i,]/3+i)
}
```

<img src="README_figs/Readme-unnamed-chunk-3-1.png" width="672" />

## Generate random environmental time series

2 random timeseries called ev1 and ev2, ev1 with cycles at 4 years and ev2 with cycles at 8 years.
Time series need to be box-cox transformed, note that in this example I use clev=1 because this is generated time series. With real data I normally use clev=5. 


```r
# ev1<-sin(2*pi*times/4)+rnorm(50, mean=0, sd=0.5)+1.1
# ev2<-sin(2*pi*times/8)+rnorm(50, mean=0, sd=0.5)+1.1
# ev12<-rbind(ev1,ev2)
# clnev<-cleandat(ev12, times,1)$cdat
clnev<-as.matrix(read.csv("D:/snappers/clnev_example.csv", header=T, row.names=1))
```

## Plot of generated environmental time series


```r
plot(times, clnev[1,], type="l", xlab="time", ylab="env var index", ylim=c(-4,4))
lines(times, clnev[2,], col="blue")
```

<img src="README_figs/Readme-unnamed-chunk-5-1.png" width="672" />

## Plot of generated species time series in ggplot


```r
spcdt<-as.data.frame(t(clndat))
spcdt$time<-times
spdf<-melt(spcdt, id.vars="time", value.name = "cleandat")
spdf<-spdf %>% mutate(sp=substr(variable, 1,3), loc=substr(variable, 4,7))
spdf$sp<-as.factor(spdf$sp)
spdf$loc<-as.factor(spdf$loc)
spdf$time<-as.numeric(spdf$time)
str(spdf)#check
## 'data.frame':	200 obs. of  5 variables:
##  $ time    : num  1 2 3 4 5 6 7 8 9 10 ...
##  $ variable: Factor w/ 4 levels "sp1loc1","sp2loc1",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ cleandat: num  0.769 0.438 -1.93 0.429 0.929 ...
##  $ sp      : Factor w/ 2 levels "sp1","sp2": 1 1 1 1 1 1 1 1 1 1 ...
##  $ loc     : Factor w/ 2 levels "loc1","loc2": 1 1 1 1 1 1 1 1 1 1 ...
spp1<-ggplot(spdf, aes(x=time, y=cleandat, color=sp, linetype=loc))
spp1 + geom_line() + theme_bw() + labs(x="Time", y="Transformed index") + 
  theme(legend.box = "horizontal", legend.position = c(0.85, 0.15)) +
  scale_y_continuous(limits = c(-5,4)) +
  scale_color_discrete(name="Species", labels=c("Sp1", "Sp2")) +
  scale_linetype_discrete(name="Location", labels=c("Loc1", "Loc2"))
```

<img src="README_figs/Readme-unnamed-chunk-6-1.png" width="672" />

## Plots by location 1 (synchronous at 4 year timescales) and species 2 (synchronous at 8 year timescales)


```r
loc1df<-filter(spdf, loc=="loc1")
loc1p1<-ggplot(loc1df, aes(x=time, y=cleandat, color=sp))
loc1p1 + geom_line() + theme_bw() + labs(x="Time", y="Transformed index") + 
  theme(legend.box = "horizontal", legend.position = c(0.85, 0.15)) +
  scale_y_continuous(limits = c(-4,3)) +
  scale_color_discrete(name="Species", labels=c("sp1loc1", "sp2loc1"))
```

<img src="README_figs/Readme-unnamed-chunk-7-1.png" width="672" />

```r

sp2df<-filter(spdf, sp=="sp2")
sp2p1<-ggplot(sp2df, aes(x=time, y=cleandat, linetype=loc))
sp2p1 + geom_line() + theme_bw() + labs(x="Time", y="Transformed index") + 
  theme(legend.box = "horizontal", legend.position = c(0.85, 0.15)) +
  scale_y_continuous(limits = c(-5,4)) +
  scale_linetype_discrete(name="Location", labels=c("sp2loc1", "sp2loc2"))
```

<img src="README_figs/Readme-unnamed-chunk-7-2.png" width="672" />

## Plot of wavelet mean fields and wavelet phasor mean fields 

This step is more for exploratory analyses, look at wsyn vignette for more information on wavelet analyses. Note that scale.max.input should be 1/3 of the full time series, in this example it is 50/3. This is just to see if overall, there is synchrony among all biological time series.


```r
spwmf<-wmf(clndat, times=1:50, scale.min=2, scale.max.input = 16)
plotmag(spwmf)
```

<img src="README_figs/Readme-unnamed-chunk-8-1.png" width="672" />

```r

spwpmf<-wpmf(clndat, times=1:50, scale.min=2, scale.max.input = 16, sigmethod="quick", nrand=1000)
plotmag(spwpmf)
```

<img src="README_figs/Readme-unnamed-chunk-8-2.png" width="672" />

The mean field plots show the amount of synchrony between the 4 biological time series, if both wmf and wpmf plots show similar areas with high power (red areas), those are likely to be real synchrony.This can also help to determine what timescales to split up the data for coherence analyses. Because these are generated datasets, I will conduct the coherence analyses on timescales <=5 (short) and timescales 6-10 (long). 

## Coherence analyses

Note that the visualization of a 'correlation matrix' using the package corrplot is designed for correlation values from -1 to 1, however, coherence values are only from 0 to 1. Hence, the negative values (in blue) can be ignored. I also only use 1000 surrogates to be faster here, for real data I try to use 10,000 surrogates.


```r
spcoh2.5<-synmat(clndat, times=1:50, method="coh", scale.max.input = 16, tsrange=c(2,5)) # coherence value
rownames(spcoh2.5)<-c("sp1loc1", "sp2loc1", "sp1loc2", "sp2loc2")
colnames(spcoh2.5)<-c("sp1loc1", "sp2loc1", "sp1loc2", "sp2loc2")
spcohsig2.5<-synmat(clndat, times=1:50, method="coh.sig.fast", scale.max.input = 16, tsrange=c(2,5), nsurrogs=1000) #significance of coherence based on 1000 surrogates
spcoh2.5pvals<-1-spcohsig2.5 #because the default values for synmat gives 1-p-values
colbwr<-colorRampPalette(c("blue", "white", "red"))#to specify colour palette
corrplot(spcoh2.5, method="number", type="lower", tl.pos="ld", tl.srt=0, tl.offset=0.7, col=colbwr(10), is.corr=TRUE, diag=F, tl.col="black", p.mat=spcoh2.5pvals, 
         sig.level=0.05, insig="blank", mar=c(0.5,0.5,1.5,0.5))
mtext("Coherence at timescale 2-5 years (p<0.05)", side=3, line=0)
```

<img src="README_figs/Readme-unnamed-chunk-9-1.png" width="672" />

```r

spcoh6.10<-synmat(clndat, times=1:50, method="coh", scale.max.input = 16, tsrange=c(6,10))
rownames(spcoh6.10)<-c("sp1loc1", "sp2loc1", "sp1loc2", "sp2loc2")
colnames(spcoh6.10)<-c("sp1loc1", "sp2loc1", "sp1loc2", "sp2loc2")
spcohsig6.10<-synmat(clndat, times=1:50, method="coh.sig.fast", scale.max.input = 16, tsrange=c(6,10), nsurrogs=1000) 
spcoh6.10pvals<-1-spcohsig6.10 
corrplot(spcoh6.10, method="number", type="lower", tl.pos="ld", tl.srt=0, tl.offset=0.7, col=colbwr(10), is.corr=TRUE, diag=F, tl.col="black", p.mat=spcoh6.10pvals, 
         sig.level=0.05, insig="blank", mar=c(0.5,0.5,1.5,0.5))
mtext("Coherence at timescale 6-10 years (p<0.05)", side=3, line=0)
```

<img src="README_figs/Readme-unnamed-chunk-9-2.png" width="672" />

At timescales of 2-5 years, sp1loc1 and sp2loc1 are coherent (as expected because the data was made to be coherent at 4 year timescales), also sp2loc1 and sp2loc2 were coherent at these short timescales.
At timescales of 6-10 years, sp2loc1 and sp2loc2 were coherent, as expected because they were made to be synchronous at 8 year timescales.

## Phase analyses

The next step is to figure out if the coherences were synchronous (in-phase), anti-synchronous (anti-phase) or phase-lagged. The code below shows the phase analyses first for sp1loc1 and sp2loc1, then for sp2loc1 and sp2loc2 at timescale bands of 2-5 years and 6-10 years. 


```r
loc1coh<-coh(clndat[1,], clndat[2,], times=1:50, norm="powall", sigmethod="fast", nrand=10000, scale.max.input = 10)
loc1coh<-bandtest(loc1coh, c(2,5))
loc1coh<-bandtest(loc1coh, c(6,10))
plotmag(loc1coh)
```

<img src="README_figs/Readme-unnamed-chunk-10-1.png" width="672" />

```r
plotphase(loc1coh)
```

<img src="README_figs/Readme-unnamed-chunk-10-2.png" width="672" />

```
## NULL
```

sp1loc1 and sp2loc1 are coherent from timescales of around 3-5 years, where the dotted red line is above the two black lines in the first plot. From the second plot of phase and timescales, we can see that at the timescales between 3-5 years (where coherence was significant), the phases were close to 0, ie., they were synchronous. In conclusion, sp1loc1 and sp2loc1 were synchronous (in-phase) at timescales 3-5 years. Note: Recall plot of sp1loc1 and sp2loc1 previously to see synchronous phases at 3-5 years. 


```r
sp2coh<-coh(clndat[2,], clndat[4,], times=1:50, norm="powall", sigmethod="fast", nrand=10000, scale.max.input = 10)
sp2coh<-bandtest(sp2coh, c(2,5))
sp2coh<-bandtest(sp2coh, c(6,10))
plotmag(sp2coh)
```

<img src="README_figs/Readme-unnamed-chunk-11-1.png" width="672" />

```r
plotphase(sp2coh)
```

<img src="README_figs/Readme-unnamed-chunk-11-2.png" width="672" />

```
## NULL
```

sp2loc1 and sp2loc2 are significantly coherent at around timescales of 3-5 years, and marginally significant around 8-10 years. At timescales of 2-5 years, both timeseries were in-phase/synchronous, whereas at timescales of 8-10 years, sp2loc1 is leading sp2loc2 by half a cycle (average phase of pi/2), can also see this on the plots of the timeseries of sp2loc1 and sp2loc2.

## Drivers of coherence

We now add in the environmental variables to see if any of them are likely to be driving the coherence patterns visible in the species timeseries. In this step, we will perform the coherence matrix with the two timescale bands of 2-5 and 6-10 years.


```r
clnall<-rbind(clndat,clnev)
allcoh2.5<-synmat(clnall, times=1:50, method="coh", scale.max.input = 16, tsrange=c(2,5))
rownames(allcoh2.5)<-c("sp1loc1", "sp2loc1", "sp1loc2", "sp2loc2", "ev1", "ev2")
colnames(allcoh2.5)<-c("sp1loc1", "sp2loc1", "sp1loc2", "sp2loc2", "ev1", "ev2")
allcohsig2.5<-synmat(clnall, times=1:50, method="coh.sig.fast", scale.max.input = 16, tsrange=c(2,5), nsurrogs=1000) #significance of coherence based on 1000 surrogates
allcoh2.5pvals<-1-allcohsig2.5 #because the default values for synmat gives 1-p-values
colbwr<-colorRampPalette(c("blue", "white", "red"))#to specify colour palette
corrplot(allcoh2.5, method="number", type="lower", tl.pos="ld", tl.srt=0, tl.offset=0.7, col=colbwr(10), is.corr=TRUE, diag=F, tl.col="black", p.mat=allcoh2.5pvals, 
         sig.level=0.05, insig="blank", mar=c(0.5,0.5,1.5,0.5))
mtext("Biol & env vars coherence at timescales 2-5 years (p<0.05)", side=3, line=0)
```

<img src="README_figs/Readme-unnamed-chunk-12-1.png" width="672" />

```r

allcoh6.10<-synmat(clnall, times=1:50, method="coh", scale.max.input = 16, tsrange=c(6,10))
rownames(allcoh6.10)<-c("sp1loc1", "sp2loc1", "sp1loc2", "sp2loc2", "ev1", "ev2")
colnames(allcoh6.10)<-c("sp1loc1", "sp2loc1", "sp1loc2", "sp2loc2", "ev1", "ev2")
allcohsig6.10<-synmat(clnall, times=1:50, method="coh.sig.fast", scale.max.input = 16, tsrange=c(6,10), nsurrogs=1000) 
allcoh6.10pvals<-1-allcohsig6.10 
corrplot(allcoh6.10, method="number", type="lower", tl.pos="ld", tl.srt=0, tl.offset=0.7, col=colbwr(10), is.corr=TRUE, diag=F, tl.col="black", p.mat=allcoh6.10pvals, 
         sig.level=0.05, insig="blank", mar=c(0.5,0.5,1.5,0.5))
mtext("Coherence at timescale 6-10 years (p<0.05)", side=3, line=0)
```

<img src="README_figs/Readme-unnamed-chunk-12-2.png" width="672" />

Ev2 is marginally coherent with sp2loc1 and sp2loc2 at 2-5 year timescales.
Ev2 is highly coherent with sp2loc1 and sp2loc2 at 6-10 year timescales, which makes sense because all of them were made to have a period of 8 years. 

## Phase analysis of environmental and biological variable

Since sp2loc1 is leading sp2loc2, lets do a phase analysis of sp2loc1 and the environmental variable ev2. 


```r
sp2loc1ev2coh<-coh(clnall[2,], clnall[6,], times=1:50, norm="powall", sigmethod="fast", nrand=10000, scale.max.input = 10)
sp2loc1ev2coh<-bandtest(sp2loc1ev2coh, c(2,5))
sp2loc1ev2coh<-bandtest(sp2loc1ev2coh, c(6,10))
plotmag(sp2loc1ev2coh)
```

<img src="README_figs/Readme-unnamed-chunk-13-1.png" width="672" />

```r
plotphase(sp2loc1ev2coh)
```

<img src="README_figs/Readme-unnamed-chunk-13-2.png" width="672" />

```
## NULL
```

P-values for both timescale bands are significant (p<0.05), and the first plot shows that for timescales 2-5 years, sp2loc1 and ev2 are coherent between 3-4 year timescales. For timescales 6-10 years, they are coherent at 7-9 year timescales. The second plot of phases shows that sp2loc1 is leading ev2 by more than half a cycle (which does not make sense biologically since we would expect environmental variables to be leading biological signals). At timescales of 7-9 years, both are synchronous/in-phase since the phase is around zero. 

# Plot of sp2loc1 and ev2


```r
plot(times, clnall[2,], type="l", xlab="time", ylab="Index", ylim=c(-4,4))
lines(times, clnall[6,], col="blue")
```

<img src="README_figs/Readme-unnamed-chunk-14-1.png" width="672" />

At long timescales (7-9 years), both sp2loc1 (black line) and ev2 (blue line) are synchronous. 
