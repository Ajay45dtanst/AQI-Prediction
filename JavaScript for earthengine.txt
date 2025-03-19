 // Define time range
var startDate = '2022-01-01';
var endDate = '2024-10-21';

// Define 10 major cities with bounding box (buffered regions)
var cities = {
  "Delhi": ee.Geometry.Point([77.1025, 28.7041]).buffer(5000), // 5km buffer
  "Mumbai": ee.Geometry.Point([72.8777, 19.0760]).buffer(5000),
  "Kolkata": ee.Geometry.Point([88.3639, 22.5726]).buffer(5000),
  "Chennai": ee.Geometry.Point([80.2707, 13.0827]).buffer(5000),
  "Bengaluru": ee.Geometry.Point([77.5946, 12.9716]).buffer(5000),
  "Hyderabad": ee.Geometry.Point([78.4867, 17.3850]).buffer(5000),
  "Ahmedabad": ee.Geometry.Point([72.5714, 23.0225]).buffer(5000),
  "Pune": ee.Geometry.Point([73.8567, 18.5204]).buffer(5000),
  "Jaipur": ee.Geometry.Point([75.7873, 26.9124]).buffer(5000),
  "Indore": ee.Geometry.Point([80.9462, 26.8467]).buffer(5000)
};

// Load Sentinel-5P NO₂ dataset
var NO2_dataset = ee.ImageCollection("COPERNICUS/S5P/NRTI/L3_NO2")
    .filterDate(startDate, endDate)
    .select('NO2_column_number_density')
    .median(); // Aggregate over time

// Load Sentinel-5P CO dataset
var CO_dataset = ee.ImageCollection("COPERNICUS/S5P/NRTI/L3_CO")
    .filterDate(startDate, endDate)
    .select('CO_column_number_density')
    .median();

// Load Sentinel-5P O₃ dataset
var O3_dataset = ee.ImageCollection("COPERNICUS/S5P/NRTI/L3_O3")
    .filterDate(startDate, endDate)
    .select('O3_column_number_density')
    .median();

// Load MODIS Aerosol Optical Depth (proxy for PM2.5)
var PM25_dataset = ee.ImageCollection("MODIS/006/MCD19A2_GRANULES")
    .filterDate(startDate, endDate)
    .select('Optical_Depth_047')
    .median();

// Function to export satellite images
function exportImage(cityName, geometry) {
  var image = ee.Image.cat([
    NO2_dataset.rename('NO2'),
    CO_dataset.rename('CO'),
    O3_dataset.rename('O3'),
    PM25_dataset.rename('PM2_5')
  ]).clip(geometry); // Clip to city bounding box

  // Export as GeoTIFF
  Export.image.toDrive({
    image: image,
    description: 'Satellite_AQI_' + cityName,
    folder: 'GEE_AQI_Images',
    fileNamePrefix: cityName + '_AQI',
    scale: 1000,  // Resolution (adjust as needed)
    region: geometry,
    fileFormat: 'GeoTIFF'
  });
}

// Loop through cities and export images
Object.keys(cities).forEach(function(city) {
  exportImage(city, cities[city]);
});
