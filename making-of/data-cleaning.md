# Going from Data That There Is to Data That I Want

_Dear Reader: What I note here is what worked for me. I may have missed something that I did, since I'm not a DVR. Conversely, something I write may be replicable on my machine, with my data, under the light of a full moon, but might not work for you, dear reader._

### Assumptions

* You have GDAL installed. You may get something from looking at [my previous writeup from when I first used GDAL](https://github.com/triplingual/berg-diary/blob/master/README.md).
* You are comfortable with running command-lines / shells (I recommend [Programming Historian](https://programminghistorian.org/lessons/intro-to-bash) [[Español](https://programminghistorian.org/es/lecciones/introduccion-a-bash)] for this and in general)

## Turn a Shapefile Into GeoJSON

You should first make sure you know what projection systems you need the GeoJSON coordinates to fit, just to be sure your points aren't materially off. One way to start if you have a shapefile is to look at the `.prj` file using `head` or similar on a *nix/BSD box. Alternately, you can run `ogrinfo -al -so path/to/shapefile.shp` (h/t [Gothos](http://gothos.info/2009/04/transform-projections-with-gdal-ogr/)). That tells you the projection system of the shapefile. If you're using OpenStreetMap like I am, [it seems to use WGS84](http://openstreetmapdata.com/info/projections).

Since I am looking historically at the Senegambian area — grudgingly using the colonizers' term — I started by merging the shapefiles for The Gambia and Senegal, obtained at the [Humanitarian Data Exchange](https://data.humdata.org/dataset/senegal-administrative-boundaries) and using [merge instructions from GDAL](http://www.gdal.org/drv_shapefile.html). Rather than removing the Senegal—The Gambia border, the merge kept the border from each shapefile,  doubling the boundary. It's an egregious anachronism, but one I haven't erased yet.

Info for using ogr2ogr (from GDAL) to [convert shapefiles to GeoJSON](http://vallandingham.me/shapefile_to_geojson.html)

This [GIS StackExchange question about converting character encodings](https://gis.stackexchange.com/questions/15912/how-to-encode-shapefiles-from-latin1-to-utf-8) can be useful when going from a specific character set to another, including UTF-8. I had to do this with some Polish GIS files for the Mary Berg project, as they were in ISO 8859-2.

## Make GeoJSON more Readable

Regexes to use in BBEdit for reducing line count of a geoJSON file that's gone through a [print improver](http://jsonprettyprint.com/) (sometimes called a pretty printer or JSON beautifier, which, ugh). My first file ballooned to 40K lines. "Easy to read" doesn't mean "easy to scroll".

Convert
<pre>\[\n\s*-1([0-9])</pre>
To
<pre>[ -1\1</pre>

Convert
<pre>,\n\s*1([0-9])</pre>
To
<pre>, 1\1</pre>

Convert
<pre>\n\s*],\n(\s*)\[ -1([0-9])</pre>
To
<pre> ],\n\1[ -1\2</pre>

When using these regexes on multiple shapefiles, I found the first thing that started to vary was the spacing.