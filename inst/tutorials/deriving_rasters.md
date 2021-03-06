## Deriving rasters from other rasters

It is not uncommon to need to derive one set of rasters from another.  [Raster math](https://rspatial.org/raster/spatial/8-rastermanip.html#) is pretty easy, but sometimes the steps required can be tricky.  This trickiness includes computing a cumulative series, like cumulative SST.

Here we'll make a cumulative series of CHLOR_A values from our monthly 2018 dataset form the Gulf of Maine. There are a number of ways to do this, but we describe here a very simple approach. First we load packages from our library, then we load our database and filter it to just monthly CHLOR_A in 2018.

```
library(ohwobpg)
library(raster)
library(dplyr, warn.conflicts = FALSE)

path <- system.file("gom", package = "ohwobpg")
db <- read_database(path) %>%
  dplyr::filter(per == 'MO' &
                param == 'chlor_a')
```

So now we have our 12 records.  Let's read them into a raster stack, `chl`.
```
chl <- raster::stack(as_filename(db, path))
```

Next we iterate through layers 2 through 12, computing the cumulative sum as we go. Afterwards, we'll give the layers more meaningful names.

```
for (i in seq(from = 2, to = raster::nlayers(chl))){
  chl[[i]] <- chl[[i]] + chl[[i-1]]
}
names(chl) <- format(db$date, "%b")
```

Now we can do a simple plot for all 12 months.  Note how missing data data propagates forward in time. We'll use the [rasterVis](https://CRAN.R-project.org/package=rasterVis) package to draw, but if it isn't installed we'll just do a simple base-R plot that is similar.

```
installed <- rownames(installed.packages())
if ("rasterVis" %in% installed){
  rasterVis::levelplot(log10(chl))
} else {
  breaks <- seq(from = -1, to = 2.5, by = 0.5)
  # adapted from the [viridisLite]() package
  # pal <- viridisLite::magma(length(breaks) - 1)
  pal <- c("#000004FF", "#1D1147FF", "#51127CFF", "#822681FF", "#B63679FF", 
           "#E65164FF", "#FB8861FF", "#FEC287FF", "#FCFDBFFF")
  plot(log10(chl), col = pal, breaks = breaks)
}
```
![Derived raster images](deriving_rasters.png)
