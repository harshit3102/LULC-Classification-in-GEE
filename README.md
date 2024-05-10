# LULC-Classification-in-GEE
var imagecollection = ee.ImageCollection("COPERNICUS/S2_HARMONIZED");
var image = imagecollection.filterBounds(geometry)
                        .filterDate('2023-12-15','2024-01-01')
                        .sort('CLOUDY_PIXEL_PERCENTAGE', false)
                        .mosaic()
                        .clip(geometry);
Map.addLayer(image,imageVisParam);

var bands = ['B1','B2','B3','B4','B5','B6','B7','B8','B9','B10','B11','B12'];


// Merge feature collections with class property
var sample = (water).merge(buildingarea).merge(densevegetation).merge(lightvegetation).merge(Barrenland).merge(range_land);
// Sample regions with the 'class' property included
var training = image.select(bands).sampleRegions({
  collection: sample,
  properties: ['class'], // Ensure 'class' property is included
  scale: 12
});

// Check the training data to ensure all features have the 'class' property
print(training);

// Train the classifier
var classifier = ee.Classifier.smileRandomForest(100).train({
  features: training,
  classProperty: 'class',
  inputProperties: bands
});

// Classify the image
var classified = image.select(bands).classify(classifier);

// Display the classified image
Map.addLayer(classified, {min: 0, max: 9, palette: ['yellow','green','blue','red','black','white','Orange','brown','Violet']}, 'Classified Image');
// Define your region of interest (ROI)
var geometry = geometry;

// Specify the class you want to calculate area for
var classValue = 1; // Assuming the class you want to calculate area for is represented by value 1

// Filter the classified image to include only pixels classified as the specified class
var classImage = classified.eq(classValue);

// Calculate the area of the specified class within the ROI
var area = classImage.reduceRegion({
  reducer: ee.Reducer.sum(), // Sum the pixel values to get total area
  geometry: geometry,
  scale: 10, // Adjust scale as needed
  maxPixels: 1e9 // Set maximum number of pixels to process
});
