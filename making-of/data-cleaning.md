# Going from Data That There Is to Data That I Want

_Dear Reader: What I note here is what worked for me. I may have missed something that I did, since I'm not a DVR. Conversely, something I write may be replicable on my machine, with my data, under the light of a full moon, but might not work for you, dear reader._

### Assumptions

* You have GDAL installed. You may get something from looking at [my previous writeup from when I first used GDAL](https://github.com/triplingual/berg-diary/blob/master/README.md).
* You are comfortable with running command-lines / shells (I recommend [Programming Historian](https://programminghistorian.org/lessons/intro-to-bash) [[Español](https://programminghistorian.org/es/lecciones/introduccion-a-bash)] for this and in general)

## Turn a Shapefile Into GeoJSON

You should first make sure you know what projection systems you need the GeoJSON coordinates to fit, just to be sure your points aren't materially off. One way to start if you have a shapefile is to look at the `.prj` file using `head` or similar on a *nix/BSD box. Alternately, you can run `ogrinfo -al -so path/to/shapefile.shp` (h/t [Gothos](http://gothos.info/2009/04/transform-projections-with-gdal-ogr/)). That tells you the projection system of the shapefile. If you're using Mapbox tiles like I am, [it seems to use WGS84](http://openstreetmapdata.com/info/projections).

Since I am looking historically at the Senegambian area — grudgingly using the colonizers' term — I started by merging the shapefiles for The Gambia and Senegal, obtained at the [Humanitarian Data Exchange](https://data.humdata.org/dataset/senegal-administrative-boundaries) and using [merge instructions from GDAL](http://www.gdal.org/drv_shapefile.html). Rather than removing the Senegal—The Gambia border, the merge kept the border from each shapefile,  doubling the boundary. It's an egregious anachronism, but one I haven't erased yet.

Info for using ogr2ogr (from GDAL) to [convert shapefiles to GeoJSON](http://vallandingham.me/shapefile_to_geojson.html). Ditto [converting KML to GeoJSON](https://gis.stackexchange.com/questions/92885/ogr2ogr-converting-kml-to-geojson). (You start to see the pattern here: `ogr2ogr -f {format} {output-file} {input-file}`)

## Merging GeoJSON files

Info for dissolving/merging GeoJSON polygons: As nearly always, [a StackExchange (in this case a GIS StackExchange) Q & A](https://gis.stackexchange.com/questions/118223/merge-geojson-polygons-with-wgs84-coordinate) and Data Viz for All. [Mapshaper command reference](https://github.com/mbloch/mapshaper/wiki/Command-Reference#-dissolve)

`-filter-islands` threshold that was most useful for me was 200 when I was removing little islands from Borneo (I wanted the coastline complex, but didn't need tiny islands), but I got there from 4,10–100 by 10s, then jumped to 200. It's possible that what I really want is, say, 138, but I don't need hyper-accuracy since I'm talking about 170 years ago before there was an Indonesia.

`-filter` was useful when dealing with the Antilles. Béart makes claims in his map about what the mancala family game was called there. To get the Antilles as a group, I decided to use Mapzen Metro Extracts rather than other services that would require me to know all the countries in advance; Metro Extracts allows for delimiting a bounding box. Once the extract was ready, I downloaded the ZIP, unzipped it, and uploaded it to Mapshaper. After poring through Mapzen's GeoJSON for the Antilles (had to do this in 2 parts), I noticed that every island had a `"place": "island"` key/value pair in the properties. Therefore, in the Mapshaper console emulator, I used `-filter 'place=="island"'` and it only retained the things marked as islands. (NB: It trimmed out some things that I perhaps would rather have kept, like Carriacou, an island of Grenada, that weren't properly included or marked in the Mapzen data. Some of this was OK, as I'm specifically trying to gesture, not make clinically precise assertions about where and how mancala games were played. Some of this wasn't, like excluding Montserrat. Since I'm speaking in part about the French, I want to not exclude other places tainted by their coloniality. But one or two places I can get from Mapzen Spelunker.) 

Subsequently, I used the -filter-islands to cull out the tiny islands that just end up making the file big.

## Excluding parts of a GeoJSON outline

Removing country borders from map of Africa when I have no attested mancala family play in that country (acknowledging troublesome equation of modern country with historic peoples):
In mapshaper.org, after uploading all unattested country GeoJSON files, with Africa continental outline layer active:
$ -erase {country-file.geojson} remove-slivers # not entirely clear whether this does what I think it ought to and what I read the doco as saying.
$ -erase libya.geojson remove-slivers
$ -erase mozambique.geojson remove-slivers
$ -erase zambia.geojson remove-slivers
$ -erase swaziland.geojson remove-slivers
$ -erase eritrea.geojson remove-slivers
$ -erase djibouti.geojson remove-slivers
$ -filter-slivers min-area=700000000

## Character Encodings

This [GIS StackExchange question about converting character encodings](https://gis.stackexchange.com/questions/15912/how-to-encode-shapefiles-from-latin1-to-utf-8) can be useful when going from a specific character set to another, including UTF-8. I had to do this with some Polish GIS files for the Mary Berg project, as they were in ISO 8859-2. This time I didn't seem to have such difficulty with encodings, but I did get a warning during one merge process and ran it again setting `-lco ENCODING=UTF-8`

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