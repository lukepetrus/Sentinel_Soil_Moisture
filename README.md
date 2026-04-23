# AMB West Project 2026
## Project Overview
The goal of this project was to contribute meaningfully to a project being conducted by AMB West Ranches, the Montana State University Agricultural Extension, and
the Montana State Hydrologist's Office to maximize early season groundwater recharge while minimizing nitrate leaching. The client requested we collate data and
deliver them relevant data layers that could be used for later analysis. They also encouraged us to perform any analysis we felt we could on the data we collected
that we felt could contribute to the project. I decided to explore how Sentinel-1 data could be used to read soil moisture across the surface and provide insight 
into how long the soil takes to become unsaturated, as well as generate an animated plot using NDVI.
## SSM_exploration
### .qmd
This is the file where I do my work. It does not render well in Github, though, so I have rendered it as a github-friendly .md file as well.
### .md
This is the rendered version of the above .qmd file with all the nice formatting and it reads much better than the .qmd but will not be useful to clone for
modification. It is also worth noting that I have set all the code chunks to `eval: false` because they are largely data analysis and the outputs are not
particularly interesting or important beyond confirming that the code works. Moreover, the final deliverable--the .gif plot--won't render in github anyways, so
I've skipped it.
## NDVI_stuff
### .qmd
This is the file where I do my work. Like the last one, it does not render well in github, so I've provided a .md for readability.
### .md
The rendered version. Once again, it is worth noting that I have set all code chunks to `eval: false` because most of them are data analysis with no meaningful
output and the one with meaningful output generates an animated .gif that won't render in github anyways.
## Sentinel_Soil_Moisture.Rproj
This is the .Rproj file that is useful for keeping working directories organized for this project.
## data
### FieldData
This folder contains .kml files for each field on the ranch.
### Other Data
I have not included the raster data here because they are very large files and I do not want to burden my github with gigabytes of raster data. I have also opted
not to include the output data for the same reason. All outputs are available to the client and to future projects through Montana State University's Sharepoint.
