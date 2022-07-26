# Petrochemical Risk Finder

![Alt Text](imageFiles/Web_Demo.gif)

## Objective
The Petrochemical Risk Finder is a web application developed to give users a sense of their risk of petrochemical contamination. 

## Description
This tool allows a user to enter any address in the contiguous United States and get an index 1 - 10 based on their risk of petrochemical contamination. It also displays a map showing the 10 nearest petrochemical tanks to the user's location. Hovering over each petrochemical tank shows a rating of the tank's likelihood to break, the distance of the tank to the inputted location, the type of petrochemical tank, and the tank's diameter.

## Datasets
This project relies on information taken from three datasets. The first dataset is the Above Ground Storage Tank dataset, or AST dataset. This dataset was created by Duke researcher Celine Robinson and includes the location, type, and diameter of each petrochemical tank in the United States. The second dataset used is the FEMA National Risk Index dataset, or NRI dataset. This dataset contains a risk rating for each county in the United States for a number of natural disasters. The natural disasters we looked at were earthquakes, strong winds, hurricanes, tornados, costal flooding, and riverine flooding. The final dataset used is the FEMA National Flood Hazard Layer dataset, or NFHL dataset. This dataset contains the location of each floodplain in the United States.

## Process

### Geocoding
Geocoding for this project uses both Open Street Maps and the Google Maps API. Google Maps API charges for requests at a certain number, so we use it as sparingly as possible. When the user inputs an address, we check to see if Open Street Maps can return coordinates on the address. If it can't, we see if the Google Maps API can return coordinates on the address. If neither geocoding services recognize the address, we print 'Address not Recognized.'

### Finding Ten Nearest Tanks
Finding the ten nearest tanks from the inputted address involves using a method taken from the University of Helsinki relying on machine learning via the BallTree method in the sklearn library. This method refrences the coordinates of the inputted address with the coordinates of all tanks around the United States. It then returns a dataframe of the ten nearest tanks to the inputted address. To get the distance from the inputted address and the tanks, we use the .haversine method in the haversine library.

### Calculating the Location Risk Index
To calculate the petrochemical risk index, we give each petrochemical tank in the United States an individual tank risk score based on the likelihood that it would leak. Petrochemical tank leaks are usually caused by natural disasters, so to calculate the likelihood a tank would leak we pulled in millions of lines of natural hazard data for counties across the United States provided from government organizations like FEMA. Whenever a user inputs their address, we calculate the risk index based on how close that address is to the nearest petrochemical tanks and what is the risk index of those nearby tanks.

### Displaying the Map
To display a map that shows the inputted address and the ten nearest tanks we use the library folium. It allows us to add custom markers to the map with tooltips to give more context.

### Search Bar and Button
We used the ipywidgets library for the web application search bar and button. This allows us to execute code when the button return key is pressed.