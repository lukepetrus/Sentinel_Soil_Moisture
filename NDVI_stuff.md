# PlanetDataNDVIStuff


## Introduction

I am going to try to get some NDVI from the planet data

#### Note: eval is set to false for github .md because there are no meaningful visuals.

## Load the Data

``` r
library(tidyverse)
library(terra)
library(purrr)
```

``` r
# Loads in some planet data
PlanetData1 <- rast("./data/PlanetData/part1/PSScene/20240620_182805_39_2473_3B_AnalyticMS_SR_clip.tif")
PlanetData2 <- rast("./data/PlanetData/part1/PSScene/20240620_182807_25_2473_3B_AnalyticMS_SR_clip.tif") |> 
  project("epsg:4326")

# Scale it back down
PlanetData2 <- PlanetData2 * 0.0001

# Loads in the fields
ReinkeWheelLine <- vect("./data/FieldData/onx-markups-03132026.kml")
LittleReinkePivot <- vect("./data/FieldData/onx-markups-03132026(1).kml")
NReinkeWheelLine <- vect("./data/FieldData/onx-markups-03132026(2).kml")
WilsonPivot <- vect("./data/FieldData/onx-markups-03132026(3).kml")
NorthReinkePivot <- vect("./data/FieldData/onx-markups-03132026(4).kml")
SouthReinkePivot <- vect("./data/FieldData/onx-markups-03132026(5).kml")
UpperPivot <- vect("./data/FieldData/onx-markups-03132026(6).kml")
ClarePivot <- vect("./data/FieldData/onx-markups-03132026(7).kml")
LittleValleyPivot <- vect("./data/FieldData/onx-markups-03132026(8).kml")
SnubPivot <- vect("./data/FieldData/onx-markups-03132026(9).kml")
TurkeyBarnPivot <- vect("./data/FieldData/onx-markups-03132026(10).kml")
BigValleyPivot <- vect("./data/FieldData/onx-markups-03132026(11).kml")
NewValleyPivot <- vect("./data/FieldData/onx-markups-03132026(12).kml")
HousePivot <- vect("./data/FieldData/onx-markups-03132026(13).kml")

# Combines the fields into one vector
combined <- rbind(ReinkeWheelLine, LittleReinkePivot, NReinkeWheelLine, WilsonPivot, NorthReinkePivot, SouthReinkePivot, UpperPivot, ClarePivot, LittleValleyPivot, SnubPivot, TurkeyBarnPivot, BigValleyPivot, NewValleyPivot, HousePivot)

# Plots the fields with their names
plot(combined)
text(combined, 'Name', cex = 0.5 )
```

That’s all well and good, but let’s see if I can auto-load all the files
I want.

``` r
# List the files in the chosen directory that contain the regex I am looking for
fileList <- list.files(path = "./data/PlanetData/part1/PSScene/", pattern = "SR", full.names = T)

#That is a list of files. Now to generate rasters from them
rastList <- map(fileList, rast)

# Now let's check one
plot(rastList[[1]])
```

Great success (*Borat voice*)

``` r
# Make a raster with 48 rows
rastTibble <- tibble(.rows = 48)

# populate the rows with files and extract dates
nestedFiles <- rastTibble |> 
  mutate(fileName = fileList,
         date = str_extract(fileName, "[:digit:]{8}")) |> 
  group_by(date) |> 
  nest()

# selects january, then selects the fileName column
nestedFiles$data[[1]]$fileName

# Turn them all into rasters
nestedRasters <- map(nestedFiles$data, ~map(.x$fileName, rast))

# Mosaic the rasters
mosaicRasters <- map(nestedRasters, ~mosaic(sprc(.x))) |> 
  map(~project(.x, "epsg:4326")) |> 
  setNames(nestedFiles$date)

# Test
plotRGB(mosaicRasters[[1]], r = 3, b = 1, g = 2, stretch = "lin")
```

Now, I am actually returning from the future (*Clip the Fields* section)
to do up the fields in the same way I did the rasters. Let’s have some
fun.

``` r
# list field files
fieldFileList <- list.files(path = "./data/FieldData/", full.names = T)

# Turn them into spatVectors
vectList <- map(fieldFileList, vect)

# Extract field names
fieldNames <- map_chr(vectList, ~.x[]$Name)

# Assigns the fields their names
vectList <- setNames(vectList, fieldNames)
```

## Fix the Data

``` r
# Now we're going to mosaic some rasters
janRast <- mosaic(rastList[[1]], rastList[[2]]) |> 
  project("epsg:4326")

febRast <- mosaic(rastList[[3]], rastList[[4]], rastList[[5]], rastList[[6]], rastList[[7]]) |> 
  project("epsg:4326")

marchRast <- mosaic(rastList[[8]], rastList[[9]], rastList[[10]]) |> 
  project("epsg:4326")

aprilRast <- mosaic(rastList[[11]], rastList[[12]], rastList[[13]]) |> 
  project("epsg:4326")

mayRast <- mosaic(rastList[[14]], rastList[[15]], rastList[[16]]) |> 
  project("epsg:4326")

juneRast <- mosaic(rastList[[17]], rastList[[18]], rastList[[19]], rastList[[20]], rastList[[21]]) |> 
  project("epsg:4326")

julyRast <- mosaic(rastList[[22]], rastList[[23]]) |> 
  project("epsg:4326")

augRast <- mosaic(rastList[[24]], rastList[[25]], rastList[[26]], rastList[[27]]) |> 
  project("epsg:4326")

septRast <- mosaic(rastList[[28]], rastList[[29]], rastList[[30]], rastList[[31]], rastList[[32]]) |> 
  project("epsg:4326")

octRast <- mosaic(rastList[[33]], rastList[[34]], rastList[[35]], rastList[[36]], rastList[[37]]) |> 
  project("epsg:4326")

novRast <- mosaic(rastList[[38]], rastList[[39]], rastList[[40]], rastList[[41]], rastList[[42]], rastList[[43]], rastList[[44]], rastList[[45]]) |> 
  project("epsg:4326")

decRast <- mosaic(rastList[[46]], rastList[[47]], rastList[[48]]) |> 
  project("epsg:4326")

# And a check
plotRGB(juneRast, r = 3, b = 1, g = 2, stretch = "lin")
```

Gorgeous. It works. I now have the rasters.

Okay, that takes care of getting some data in. Now lets plot them in
over each other.

``` r
plotRGB(PlanetData2, r = 3, b = 1, g = 2, stretch = "lin")
polys(combined, col = "tomato", lwd = 1, alpha = 0.5)
```

Now, let’s do an NDVI.

## NDVI Stuff

``` r
# Raster math
NDVI <- (PlanetData2$nir - PlanetData2$red) / (PlanetData2$nir + PlanetData2$red)
plot(NDVI, col = map.pal("grass", 100))

# Cool, it works. Now let's define it as a function

NDVIfunc <- function(red, nir) {
  (nir - red) / (nir + red)
}

# And a test
plot(NDVIfunc(PlanetData2$red, PlanetData2$nir))
```

Sweet Baby Jesus, we’re cooking with vegetation indices.

Goddamnit, though. This function needs more functionality if I want to
use map() again.

``` r
# Improving the function to only take one input and separate the bands within the function
NDVIfunc <- function(raster) {
  red <- raster$red
  nir <- raster$nir
  
  (nir - red) / (nir + red)
}

# Test
#plot(NDVIfunc(juneRast))
```

Mwahahahaha! *It is alive!*

``` r
# Turn my mosiac'd rasters into a list (don't need anymore)
#MRastList <- list(janRast, febRast, marchRast, aprilRast, mayRast, juneRast, julyRast, augRast, septRast, octRast, novRast, decRast)

#This list above is all well and good, but I am going to change the below workflow to use the mosaicRasters list i generated in the Automation2 chunk

# Run the function on each one
NDVIlist <- map(mosaicRasters, NDVIfunc) |> 
  map(\(x) {
    names(x) <- "NDVI"
    x
  })

# Test
plot(NDVIlist[[6]])

#The CRS is no longer weird and this is now irrelevant

# I discovered that the CRS is weird
#crs(NDVIlist[[1]])

# Reproject
#NDVIlist <- map(NDVIlist, ~project(.x, y = "epsg:4326"))

# Test
#crs(NDVIlist[[1]])
```

Now, let’s clip a field out and hit it with some G\*.

## Clip the Fields

This is the old version of the clip.

``` r
# Clip out each field
BigValleyNDVI <- crop(NDVI, BigValleyPivot, mask = T)
ClareNDVI <- crop(NDVI, ClarePivot, mask = T)
HouseNDVI <- crop(NDVI, HousePivot, mask = T)
LittleReinkeNDVI <- crop(NDVI, LittleReinkePivot, mask = T)
LittleValleyNDVI <- crop(NDVI, LittleValleyPivot, mask = T)
NewValleyNDVI <- crop(NDVI, NewValleyPivot, mask = T)
NorthReinkeNDVI <- crop(NDVI, NorthReinkePivot, mask = T)
NReinkeWheelNDVI <- crop(NDVI, NReinkeWheelLine, mask = T)
ReinkeWheelNDVI <- crop(NDVI, ReinkeWheelLine, mask = T)
SnubNDVI <- crop(NDVI, SnubPivot, mask = T)
SouthReinkeNDVI <- crop(NDVI, SouthReinkePivot, mask = T)
TurkeyBarnNDVI <- crop(NDVI, TurkeyBarnPivot, mask = T)
UpperNDVI <- crop(NDVI, UpperPivot, mask = T)
WilsonNDVI <- crop(NDVI, WilsonPivot, mask = T)

# make an NDVI list for June
NDVIFieldlist <- list(BigValleyNDVI, ClareNDVI, HouseNDVI, LittleValleyNDVI, LittleReinkeNDVI, NewValleyNDVI, NorthReinkeNDVI, NReinkeWheelNDVI, ReinkeWheelNDVI, SnubNDVI, TurkeyBarnNDVI, UpperNDVI, WilsonNDVI)

plot(BigValleyNDVI)
```

And now for the new and improved version.

``` r
# Create a crop function to use in another map function
cropFunc <- function(ndvi) {
  map(vectList, ~crop(ndvi, .x, mask = T))
}

# 168 rasters in 1 line
cropByMonth <- map(NDVIlist, cropFunc)

# Test
plot(cropByMonth[[2]][[3]], main = names(cropByMonth[[2]])[[3]])
```

And now local Moran.

## Spatial Autocorrelation

``` r
# Run local Moran
LocMoran <- autocor(BigValleyNDVI, method = "moran", global = F)

# Test
plot(GStar)
```

Cool, now lets see about automating it

``` r
# map, baby, map
moranList <- map(cropByMonth, ~map(.x, ~autocor(.x, method = "moran", global = F)))

# test
plot(moranList[[2]][[7]])
```

## Visualization

Now I am going to try some stuff for visualizing all these fields. It
sounds like to plot it with geom_raster I need to convert each one to a
data frame that contains x, y, and the value at that location. Let’s
try.

``` r
library(rayshader)

# Turn months into a data frames
dfJanBigVal <- as.data.frame(cropByMonth[[2]][[3]], xy = T)
dfJulyBigVal <- as.data.frame(cropByMonth[[7]][[3]], xy = T)


# Plot it
graph1 <- ggplot(dfJulyBigVal) +
  geom_raster(aes(x = x, y = y, fill = nir)) +
  scale_fill_gradient(low = "tomato", high = "springgreen", limits = c(0, 1))

# Render it in 3d
plot_gg(graph1, height_aes = "fill")
```

Okay, so we’ve proven that we can render it in 3D, but can we animate a
visual that contains all 7 maps?

``` r
library(gganimate)
library(ggh4x)

# A function that turns each raster into its own dataframe with values that are inhereted from the list index
rasterBlaster <- function(monthList, date) {
  imap(monthList, \(raster, field) {
       as.data.frame(raster, xy = T) |> 
         mutate(
           date = date,
           field = field
         )
       })
}

# Runs rasterBlaster(), binds the data frames within the months together by row, then binds the combined dataframes for each month together by row to make a master dataframe with a row for every pixel in every field for every month
dfFull <- imap(cropByMonth, rasterBlaster) |> 
  map(~list_rbind(.x)) |> 
  list_rbind()

# Turn the date into a datetime object
dfFull <- dfFull |> 
    mutate(
    date = ymd(date),
    field = factor(field)
  )

# Set the layout design for the facet_manual()
design <- matrix(c(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, NA, 13, 14, NA), 
                 4, 4, byrow = T)

# Create the ggplot base for the animation
animPlot <- ggplot(dfFull, aes(x = x, y = y, fill = NDVI)) +
  geom_tile() +
  scale_fill_gradient(low = "wheat1", high = "darkgreen", 
                      limits = c(0, 1)) +
  facet_manual(~field, design = design, scales = "free") +
  labs(
    title = "AMB West NDVI",
    subtitle = "Date: {frame_time}",
    x = "",
    y = "",
    caption = "From Planet 3m data at a roughly 1 month revisit, 01/31/24-12/05/24
By Luke Petrus"
    ) +
  theme_bw() +
  theme(axis.text = element_blank(),
        plot.title = element_text(face = "bold", hjust = 0.5, color = "black"),
        plot.subtitle = element_text(hjust = 0.5, color = "black"),
        plot.background = element_rect(fill = "white"),
        legend.background = element_rect(fill = "white"),
        legend.text = element_text(color = "black"),
        legend.title = element_text(color = "black"),
        plot.caption = element_text(hjust = 0)) +
  transition_time(date)

# Render the animation  
animate(animPlot)

# Save the animation
anim_save("fieldNDVI.gif")
```

And it works!!! I have exactly the animated graphic I was envisioning.
Only took like 15 hours of coding aha. Now, I have to decide if I want
to do the same for local Moran’s I. Fudge it, why not?

``` r
# Change NDVI to Moran's Local I
moranList <- map(moranList, ~map(.x, \(x) {
  names(x) <- "Local Moran's I"
  x
}))

# Create the unified data frame for the Moran's rasters
dfFullMoran <- imap(moranList, rasterBlaster) |> 
  map(~list_rbind(.x)) |> 
  list_rbind()

# Change field to a factor
dfFullMoran <- dfFullMoran |> 
  mutate(field = factor(field),
         date = ymd(date))

# Create the gganim object in the same fashion as the other one, but with a different gradient
dfFullMoran |> 
  filter(date == "2024-10-26") |> 
ggplot(aes(x = x, y = y, fill = `Local Moran's I`)) +
  geom_tile() +
  scale_fill_gradient(low = "darkslateblue", high = "goldenrod1") +
  facet_manual(~field, design = design, scales = "free") +
  labs(
    title = "AMB West NDVI",
    subtitle = "Date: {frame_time}",
    x = "",
    y = ""
    ) +
  theme_bw() +
  theme(axis.text = element_blank(),
        plot.title = element_text(face = "bold", hjust = 0.5, color = "black"),
        plot.subtitle = element_text(hjust = 0.5, color = "black"),
        plot.background = element_rect(fill = "white"),
        legend.background = element_rect(fill = "white"),
        legend.text = element_text(color = "black"),
        legend.title = element_text(color = "black"))# +
#  transition_time(date)
```

Not going to lie, the exploratory data analysis I am doing on Moran’s I
makes me think this animation isn’t really worth making. I don’t think
it will show anything that can’t be more clearly seen on the NDVI
animation.

## Export

Anyways, now I need to export these NDVI rasters. I will write a quick
function to do that for me.

``` r
# Export function
exportFunc <- function(monthList, date) {
  imap(monthList, \(x, field){
    fname <- str_replace_all(paste0("./data/FinalData/NDVI/", 
                                     as.character(date), 
                                     field, ".tif"), "\\s|-", "_")
    writeRaster(x, filename = fname)
  })
}

imap(cropByMonth, exportFunc)
```

Beautiful.

However, Now I’m feeling like maybe I should do an RGB crop too. Let’s
run it

``` r
# Do the crop using cropFunc and mosaicRasters
cropByMonthRGB <- map(mosaicRasters, cropFunc)

# Alter exportFunc to send them to the right spot
exportFunc <- function(monthList, date) {
  imap(monthList, \(x, field){
    fname <- str_replace_all(paste0("./data/FinalData/RGBNir/", 
                                     as.character(date), 
                                     field, ".tif"), "\\s|-", "_")
    writeRaster(x, filename = fname)
  })
}

# Export 'em
imap(cropByMonthRGB, exportFunc)
```

And on that topic, let’s see if there is a way to make my animation but
do it with RGB rasters.

## What if. . . RGB

Okay, a quick Claude query suggested tidyterra which extends ggplot2’s
functionality with geom_spatraster_rgb() (and a bunch of other stuff). I
am wondering if I am going to be able to facet in the same way and
thinking probably not, but let’s give it a try and see what happens.
Otherwise, I will have to compute a hex color for every pixel and do the
long dataframe thing again, which is doable but annoying.

``` r
library(tidyterra)

# A basic plot
ggplot() +
  geom_spatraster_rgb(data = cropByMonthRGB[[6]][[3]], r = 3, b = 1, g = 2, stretch = "lin")
```

On second thought, this might not be worth it. I am not going to be able
to facet by field with tidyterra, and even if I could I am not sure how
gganimate would handle the transitions between values that are
non-numeric. If I did the long data frame method with hex colors that
still doesn’t solve the transition problem for gganimate. I think this
might be the limit of what I can do here.
