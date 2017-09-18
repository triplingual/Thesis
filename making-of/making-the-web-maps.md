# Making the Web Maps

## Mapbox

_needs further elaboration_
In a [past project](Mary Berg's Diary) I used Mapbox.js, a plugin for or an overlay to (depending on who you read, I don't really know, myself) Leaflet. Leaflet is a mapping API, but provides no baselayers. Mapbox is a tile service among other things.

In this project I decided to try to use MapboxGL JS, in the interest of forward compatibility, but it's not been so easy. Getting a basic satellite view of the Senegambian region up was not terribly hard. Following (Mapbox's example)[], I substituted at will and everything was OK. :ta-da: But then the next steps proved a little more tricky. Again following Mapbox examples, I tried to load the GeoJSON I had made for one view of the region's boundaries (really just a merging of the data from Humanitarian Data Exchange for the Sénégal nation-state boundary and the same for The Gambia. Nothin doin.

First of all, any thorough debugging will start at a fairly low level. Can I use WebGL? [CanIUse](http://caniuse.com/#feat=webgl) will say whether your browser supports WebGL, but that doesn't say anything about whether _your_ browser — the one right there in front of you that you're using to read this, perhaps — has it running. To test this out generally, visit [WebGL Report](http://webglreport.com/), where you will get a readout of whether your browser supports WebGL and which version, or (the Mapbox test page for WebGL)[http://mapbox.github.io/mapbox-gl-supported/] which will do similar. In Chrome for Mac, you may need to (enable hardware acceleration)[chrome://settings/?search=accel]. Right now I am looking around for something that will tell me what I'm doing wrong, and will probably fall back to a Mapbox example. 