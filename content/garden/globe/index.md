---
title: "Creating a travel map of your last holiday"
date: 2022-11-28
lastmod: 2022-11-28
draft: false
garden_tags: ["blender", "QGIS"]
summary: "Using Blender and QGIS to create a travel map."
status: "seeding"
---


Did you ever need a map to illustrate your magnificent trip of the last holidays? Of course, you can find plenty of maps on the internet but to find just that one you have in mind? Either, they are too small or too large, too colorful, too crowded, .... It *should* be rather easy to create your own map with exactly that look and feel that you wish with those two great open-source tools: [QGIS](https://www.qgis.org/) and [Blender](https://www.blender.org).

I would like to have some kind of inset map with a small locator globe to show where on earth my vacation spot was situated with a bigger flat map in the background. My first attempt to create the locator globe was rather successful. Another (supposedly simpler) technique however resulted in a pile of problems and work-arounds.

[here about example]


# Creating a small locator globe with QGIS and Blender

First, you need some geographical data for creating the maps. There are lots of possibilities but, perhaps, the easiest is to download some data-files for your region. A very good starting point for global data are the free vector and raster map data from [Natural Earth](https://www.naturalearthdata.com/). For a locator globe, you don't need very detailled data; a scale of 1:110m is sufficient (1 to 110 million or 1 cm = 1,100 km). Also, you don't need any cultural information (e.g. country borders, etc.). I start with some basic and simple globe but will improve upon it later on. So, you only need the following files 

- Coastline: ne_110m_coastline.zip [link](/http//www.naturalearthdata.com/download/110m/physical/ne_110m_coastline.zip)
- Graticules at 15° interval: ne_110m_graticules_15.zip [link](http//www.naturalearthdata.com/download/110m/physical/ne_110m_graticules_15.zip)

You don't have to unzip the files. Qgis can work right away with the zip files. Start QGIS and drag the zip-files to the Layers panel (see figure 1). Note that only the shp-file is added to the Layers panel. The map will look something like in figure 1.

{{< figure src="_qgis-screenshot.svg" caption="Figure 1: QGIS with 2 layers added (coastline and graticules)." width="80%" >}}

QGIS is a layered application such as GIMP or Inkscape. This means that because the graticule layer is below the coastline, the coastline will not be interrupted with the graticule lines. The layer style is a bit changed from the defaults. The coastline has a color of Black and a size of 0.5 mm. You can change this in the Properties panel at the right handside.

You need to export this image to put it on a locator globe in Blender. Choose the menu Project > Import/Export > Export Map to Image. Choose Calculate from Layer (graticules) to define the size of the image.

Now, start Blender and create a new project. Replace the default cube with a UV Sphere. Switch in the Properties panel to the Material Properties tab and add a new material. Click on the yellow button next to Base Color to add an Image Texture. Assign the exported image from QGIS with the Open button.

{{< figure src="blender-screenshot.svg" caption="Figure 2: Blender with textured sphere." width="80%" >}}

## Improvements

- In Blender, you could smooth the sphere by applying a Subdivide Modifier.
- There is also a slight problem with the UV Unwrapping (but you probably won't see it on this small scale). Because the sphere is closed at the top and bottom; you will notice two rows of triangles in the UV map. This is not optimal and will result in distortion. But, at the polar regions, you probably won't notice it. There are a few work-arounds; see for example [YouTube tutorial](https://www.youtube.com/watch?v=72hgMoyTbXE&t=116s).
- In QGIS, you could use the Land and Ocean data set to create a more colorful map. Therefore, you need the data-files: ne_110m_land.zip and ne_110m_ocean.zip. They replace the rather dull coastline data. Of course, you need to change the styles of both layers at bit. 

# Creating a small locator globe with QGIS ONLY!

Isn't it possible to create the locator globe map within QGIS alone? Yes, but it turns out to have some bumps in the road. Also, you will miss the advantage of being able to rotate the globe (in Blender) and to create for example some day-and-night animation.

A quick search on the internet ("how to create a globe in QGIS") will produce several solutions; e.g [cartographyclass.com](https://cartographyclass.com/a-world-locator-map-in-5-minutes/). They all have in common that you need to create a specific User Coordinate Reference System (CRS). Of course, I followed along and most of the time; it worked -for their coordinates!. Changing the coordinates to see the globe from a different angle, however, resulted most of the time in distortions. For example, straight lines for the parallel and meridian lines, disappearing countries, and so on.

Moreover, it turns out that QGIS nowadays (version 3.22) already has such a CRS: "The world from  space" (ESRI:102038). So, after you have added the data-files (coastline & graticule), you need to change the project CRS (which is WGS 84: EPSG:4326 for the Natural Earth data files). Use the menu Project > Properties: choose the 4th tab at the left (Project Coordinate Reference System (CRS) and filter for "The world ...". After selecting this CRS, you will notice that there are some straight lines near the North pole which shouldn't be there.

In the textbox (lower left) of the Project Coordinate Reference System dialog, You can find the definition of this CRS. At the bottom, you will find the associated command that all the tutorials used in one variation or another.

'+proj=ortho +lat_0=42.5333333333 +lon_0=-72.5333333334 +x_0=0 +y_0=0 +ellps=sphere +units=m +no_defs'

With this command, you will create an orthogonal projection, where the earth location (42.53; -72.53) will be placed in the map at coordinate 0,0 (=middle of the map). The projection itself is an ellipse, resembling a sphere. 

The reason why most tutorials gave a satisfying result is the choice of the origin point. For example, the cartographyclass.com tutorial proposes the following command:
''+proj=ortho +lat_0=49.2 +lon_0=-123 +x_0=0 +y_0=0 +a=6371000 +b=6371000 +units=m +no_defs'' 

So, with the earth location (7.9; -3.27 which is Ghana) it seemed to be OK. But, try to change the earth location to for example (21; 6) and artifacts will pop-up.

What exactly is going on, is still an enigma to me. The most comprehensible explanation comes from [Avenza.com](https://www.avenza.com/resources/blog/2019/01/14/orthographic-projections/). With the globe projection, you always can see only half of the flat map. The other half is hidden behind the horizon (azimuth?). So, the QGIS projection command should hide those lines.

In my tried but failed solutions, I have tried to erase the points in the flat map that weren't visible in the globe view. But, how? Which points should I delete. Drawing a simple circle will not do the job; you are looking from a point in outer space with perspective.

1. One solution expands the previous method. Create the globe in Blender. Rotate the globe to your liking. Switch to the UV Editing workspace. Select the globe in Edit mode and select all visible points. The will light up in the UV map. You can add a circle around the globe to make it more precise. Make a screenshot of the UV map and import that UV map in QIS.
2. Move & scale the UV bitmap so that it aligns with the shp-layer. After googling "qgis scale move a bitmap", it turns out that you need a plugin for that (Freehand Georeferencer). 3. Edit the shp-layer an remove the points that are not visible from your perspective. Once again: some bumps. A layer from a zip-file could not be edited but there seems to be not much info about it. Ultimately, I succeeded to export the layer (right mouse button). This exported layer could be edited with the Vertex tool. Deleting some points, however will reconnect the remaing points, sometimes in a straight line (cause of the artifacts?). You have also to do this with the graticule layer.
3. Perhaps, the detour with Blender isn't necessary. May be, I can create a circle within QGIS itself. After one more Google search (try "QGIS add circle"), I need a memory layer (nowadays called Scratch layer in QGIS) and the "Digitizing toolbar". But, this toolbar is greyed out (inactive). You have to change the projection to a flat projection; e.g. WGS84. But then, you relize that you don't know where to draw the circle on the flat map. So, that's a dead end, once again.
4. Ultimately, you think you have found a solution at [StackExchange](https://gis.stackexchange.com/questions/78346/ortho-projection-produces-artifacts). You can use a plugin called [Clip to Hemisphere](https://github.com/jdugge/ClipToHemisphere). But to no use. The generated error (ne_110m_land — ne_110m_land.shp has an invalid geopmetry) however will bring you to the explanation. 

  After importing the data-files (see above), you need two additional steps:

- Create the custom CRS with the menu Settings > Custom Projections. In the Name field, you can enter the name of your CRS e.g. myGlobe. In the Format > Project String, you need to enter the commands to create a so-called azimuth orthographic projection. In fact, there exists already one : "The world from  space" (ESRI:102038)
- Switch the project to the just created CRS with menu Project > Properties.


# Creating a large map with a detailled inset

Figure 3 shows a flat map of the world with all of the countries and one specific is colored. This country is then enlarged in the inset. For this kind of map, you need a better resolution; e.g. the Large Scale data 1:10m (1 cm = 100 km). Because you need the countries, you also need the Cultural data sets.  

ne-10m-admin_0_countries
add a rule-besed layer styling --> "SOV_A3" = 'BEL' + color and TZA

add a group global & tanzania

add countries, rivers and lakes center line to group tanzania; eventueel label regions????

ga naar project > New Print layout. Maak eerst global

see https://www.youtube.com/watch?v=2c4axNtuDwI&t=2s


!!!!! KAARTEN GRABBEN VIA MONITOR 1080P



, 15, 20,  Our globe will show the physical appearance of earth. For our first map (globe) we need 
graticules & coastline (top)
change color of both and size
project > export > calculate from layer (graticules)

Blender
add mesh > UV sphere
(quads)
add material > change base color to image texture
switch to UV editing and notice that there are triangles. For this globe, it don't matter.
(see you tube to fix)


 

And then, I don't know.

maps in project > QGS > Landkaarten > zanzibar








