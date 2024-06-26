# Urban-Flood-Mapping
// In December 2023 Chennai was hit by the Severe Cyclonic Storm Michaung. 
// Close to 500mm of rainfall was received over a period of a day in Chennai as per the IMD stations of Nungambakkam and Meenambakkam. 
// The Northern parts of the city were worse affected, receiving more than 600mm.
// https://en.wikipedia.org/wiki/Cyclone_Michaung

// Select images by predefined dates
var beforeStart = '2023-11-21'
var beforeEnd = '2023-12-03'
var afterStart = '2018-12-03'
var afterEnd = '2023-12-12'

var admin2 = ee.FeatureCollection("FAO/GAUL_SIMPLIFIED_500m/2015/level2");
var Chennai = admin2.filter(ee.Filter.eq('ADM2_NAME', 'Chennai'))
var geometry = Chennai.geometry()
Map.addLayer(geometry, {color: 'grey'}, 'Chennai District')

var collection= ee.ImageCollection('COPERNICUS/S1_GRD')
  .filter(ee.Filter.eq('instrumentMode','IW'))
  .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VH'))
  .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))
  .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING')) 
  .filter(ee.Filter.eq('resolution_meters',10))
  .filterBounds(geometry)
  .select(['VV', 'VH'])


var beforeCollection = collection.filterDate(beforeStart, beforeEnd)
var afterCollection = collection.filterDate(afterStart,afterEnd)

var before = beforeCollection.mosaic().clip(geometry);
var after = afterCollection.mosaic().clip(geometry);

var addRatioBand = function(image) {
  var ratioBand = image.select('VV').divide(image.select('VH')).rename('VV/VH')
  return image.addBands(ratioBand)
}

var beforeRgb = addRatioBand(before)
var afterRgb = addRatioBand(after)

var visParams = {min:[-25, -25, 0] ,max:[0, 0, 2]}
Map.centerObject(geometry, 10)
Map.addLayer(beforeRgb, visParams, 'Before Floods');
Map.addLayer(afterRgb, visParams, 'After Floods'); 

