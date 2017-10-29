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

By the time I got to Sumatra and Sulawesi, I was tired of the manual work I needed to put into assembling the pieces and then `dissolve`-ing them. Turns out, with a non-free but hopefully not burdensome option, I was able to merge GeoJSON files fairly easily, albeit with many steps and some prerequisites. If you don't have the prerequisites, I still think this is worth considering, but it may involve more learning and its accompanying frustration.

### Prerequisites

(a/k/a things I don't explain or don't explain much in The Process)

* Patience
* Experience on the command line / shell / Terminal
	* ssh
	* scp
	* setting permissions on files
* Ability to read technical documentation
* Moderate understanding of what online corporations do with your data


### The Process

* Get [an Amazon Web Services (AWS) account](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html); you can use your regular Amazon login if you have one, but it's worth considering whether managing your work (including expenses) would benefit from a distinct AWS account, weighed against the overhead of managing multiple accounts.
* [Create a non-root admin user account in AWS](http://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html), downloading the credentials document when it's available.
* Create a developer-type user account in AWS (very similar to creating the non-root admin user above, but with a different group and permissions. Download the credentials document when it's made available to you.
* Get a [Bitnami Stacksmith account](https://bitnami.com/account/sign_up)
* Create the smallest, least powered [AWS EC2 server for Node.js](https://aws.bitnami.com/launch/nodejs) you can through Bitnami
	* (You'll need to wait 2 hours after creating the AWS admin user in order to create this server.)
	* You'll need two of the items from the downloaded credentials file for the admin user you created above.
	* I don't know yet whether the storage for this box is persistent or not. My recommendation is to assume that anything you've stored on the box will disappear when you shut it down, which you should whenever you're not using it because the money starts to add up if you leave it running. That's fine if you can swing the coin and you need the box available all the time. I don't. Right now, I just want to have a Node.js box that I can merge GeoJSON files on from time to time.
* ssh to the AWS EC2 box using the PEM file and connection command line provided by Bitnami
* In another shell, scp (similarly using the PEM file and connection command line Bitnami provides) the GeoJSON files you want to merge to the EC2 box
* Install [Mapbox's geojson-merge module for Node.js](https://github.com/mapbox/geojson-merge) on the EC2 box (might need to use `sudo`, I did)
* As instructed in the Mapbox GitHub README, execute `geojson-merge infile0.geojson infile1.geojson > outfile.geojson` or `geojson-merge folder/filepattern-*.geojson > outfile.geojson`
* Download the merged file
* Upload merged file to [Mapshaper online](http://mapshaper.org)
* Run `-dissolve` in Mapshaper
* Optionally, run `-simplify {NN}%` or `-filter-islands min-vertices={NNN}` or other console commands in Mapshaper online as suitable. (Note: You wouldn't hurt yourself if you exported the Mapshaper file state before running anything you're not 100% sure of. That way, if the modification you run fails, you can restart without having to go back to the wholly unmodified file. Just a thought.)
* When you're satisfied with your GeoJSON and don't have any more to merge right now, shut down the AWS EC2 instance from either the AWS console or the Bitnami one. I'm finding the Bitnami one a touch easier because I don't have to wade through Amazon's UI, which is built to serve a host of other purposes than Bitnami's is.

## Further Mapshaper Details

The `-filter-islands` threshold for the `min-vertices` parameter that was most useful for me was 200 when I was removing little islands from Borneo (I wanted the coastline complex, but didn't need tiny islands), but I got there from 4,10–100 by 10s, then jumped to 200. It's possible that what I really want is, say, 138, but I don't need hyper-accuracy since I'm talking about 170 years ago before there was an Indonesia. When I pared down Sumatra and Sulawesi, I used `min-vertices=100` and it was fine, but I also used `-simplify 50%` on Sumatra with no adverse consequences (yet).

`-filter` was useful when dealing with the Antilles. Béart makes claims in his map about what the mancala family game was called there. To get the Antilles as a group, I decided to use Mapzen Metro Extracts rather than other services that would require me to know all the countries in advance; Metro Extracts allows for delimiting a bounding box. Once the extract was ready, I downloaded the GeoJSON ZIP under the heading "Datasets grouped into individual layers by OpenStreetMap tags", unzipped it, and uploaded the one with "landusages" in its title to Mapshaper. I also opened the GeoJSON in BBEdit (though nearly any text editor should be fine).

For the Antilles, I noticed that every island had a `"place": "island"` key/value pair in the properties for each object in the GeoJSON file. Therefore, in the Mapshaper console emulator, I used `-filter 'place=="island"'` and it only retained the things marked as islands. (NB: It trimmed out some things that I perhaps would rather have kept, like Carriacou, an island of Grenada, that weren't properly included or marked in the Mapzen data. Some of this was OK, as I'm specifically trying to gesture, not make clinically precise assertions about where and how mancala games were played. Some of this wasn't, like excluding Montserrat. Since I'm speaking in part about the French, I want to not exclude other places tainted by their coloniality. But one or two places I can get from Mapzen Spelunker.) For Kédang, I used a filter of `-filter 'type=="island"'` and then `-filter 'name!=null'` and then `-filter-slivers min-area=700000000` to get rid of the smaller islands. I really just want the main island for map legibility. In retrospect, I probably could have used `-filter-islands` instead.

Just the same I did subsequently use `-filter-islands -min-vertices={}` (where I played with the number of vertices) to cull out the tiny islands that just end up making the file big.

The Dominican Republic file got `simplify`d to 75%, which may have reduced the number of points in the GeoJSON file by 25%, but the filesize only got reduced by 10%.

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