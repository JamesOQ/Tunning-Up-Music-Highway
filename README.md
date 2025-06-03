# Tuning Up Music Highway
This is a repository for the Tuning Up Music Highway project for the Erdos Institute Summer 2025 Data Science Bootcamp.

Files Included:

datasets:
  1. Serious Injuries and Fatalities Data for I-40 Tennessee.csv
     - This is our main dataset which contains all fatal and serious injury causing crashes which happened on I-40 Tennessee between 2023 and 2025.
     
  2. Overall crash data for Madison and Henderson counties.csv
     - Secondary dataset which contains the geospatial coordinates and severity of crash for each crash that happened in either Madison or Henderson Counties between 2023 and 2025.
  
  3. Serious Injuries and Fatalities Data for I-40 Tennessee *with weather*.csv
     - An update to our main dataset which includes added weather condition data queried by data, time, and location from Visual Crossing.
  
  4. Serious Injuries and Fatalities Data for I-40 Tennessee *with weather and wetness*.csv
     - Another update to our main dataset which grades the wetness of the road at the time of crash based on the cumulative rainfall of the 6 previous hours. The grading is as follows:
       - more than 1 inch of cumulative rainfall is labeled as 'very wet'
       - between 0 and 1 inches of cumulative rainfall is 'damp'
       - otherwise 'dry'.

Code:
  1. Weather data API query.ipynb
     - Program that queries historical weather data from the Visual Crossing weather API based on location, date, and time.
  2. Road Wetness API query.ipynb
     - Program that queries cumulative precipitation amount from the Visual Crossing weather API based on the 6 hours before the crash. 
  3. Geospatial Automated Tagger.ipynb
     - Program which takes our excel sheet of manually tagged geospatial features from Google Maps and, for each feature and datapoint in our dataset, tags the category of that feature which corresponds to the longitude of the crash location of the datapoint.
