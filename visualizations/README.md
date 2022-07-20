# Visualization in Python for Duke Data Scientists

# Authors: Alyssa Ting, Susan Feng, Joey Nolan

## Visualizations

### Stacked Bar Graph of Tank Types per State (HoloViews): using Pandas ```.groupby()```, ```.size()``` and ```.pivot_table()```
This visualization uses the matplotlib backend in Holoviews to display information from the AST dataset. It outputs a stacked bar graph illustrating the exact breakdown of each tank type per state, as well as the number of tanks in each state. 

To create it, we used various different functions to manipulate the AST data as a Pandas dataframe. First, the ```.groupby()``` function groups the data by state as well as tank type, then the ```.size()``` function returns the number of each group– in this case, the number of each type of tank in each state. Using these functions consecutively returns a new dataframe listing the state, the tank type, and the number of that type of tank in that state. This dataframe is then converted to a pivot table using the ```.pivot_table()``` function, where the columns are tank types and the rows are states. The horizontal bar chart function in matplotlib, with parameter ‘stacked’ set to True, outputs the stacked bar graph visualization. 

### Number of Children Per County (GeoViews): using pandas ```.groupby()``` and ```.sum()```
This visualization uses the GeoViews library to display a map of the US, broken down at the county-level with each county colored by its total number of children, as processed from the InfoUSA dataset. It also uses the AST dataset to plot points overlay points for each storage tank on top of this map.

To create it, we used pre-processed InfoUSA data that has the number of children in each zip code. Using the pandas groupby function as above, we group this data by county and use the sum function to sum the number of children for all zip codes in a county, thus creating a dataframe with each county and the total number of children for that county. We then merged this dataframe with another dataframe we pulled from the US government’s cartographic files database, which provides geometries for each county. Combining these two dataframes allows us to plot this data spatially, mapping all counties to form a map of the US and coloring it by the number of children in that county. We then converted the latitude and longitude coordinates of each tank to Point geometries, and plotted those in a similar fashion, then used GeoViews’ ```*``` operator to overlay the two visualizations. 

### Number of Households Near Tanks per County (GeoViews): using GeoPandas ```.sjoin()```
This map of the US was created using GeoViews, and displays each county colored by the number of households in that county that are within five miles from a tank (a boundary provided to us by our researcher). 

To find the number of counties within five miles of a tank for each county, we used the GeoPandas’ ```.sjoin()``` function. This function, as detailed in the Data Merging and Wrangling Workflow Overview, can be used to find the intersection between two spatial dataframes. Using a pre-processed dataset including distances from each household to the tank nearest to it, we filtered to only keep the households within five miles of a tank. Then, we looped over a dataframe with geometries for each county, provided by the US government’s cartographic files database, to identify which tanks were in each county, and included the length of that dataframe using the len function as a new column in the counties GeoDataFrame. The final output was a GeoDataFrame with each county, the number of households within five miles of a tank in that county, and that county’s geometry. This is the dataframe used in GeoViews to plot the visualization below. 

### Charleston and Harris County Case Studies (Cuxfilter): interactive visualizations using GPUs
These two visualizations were created using the Cuxfilter library, which allows users to plot a large amount of data to create customizable and interactive dashboards using GPUs. Each visualization displays points for all households and tanks in that county, then allows the user to customize which points they would like to see: depending on distance from the nearest tank, as well as whether or not the household has elderly people or children. 
This visualization was created with pre-processed InfoUSA and AST data as detailed in the  Data Merging and Wrangling Workflow Overview. The dataframe includes coordinates for each point in the EPSG 3857 projection, whether or not the household has children or elderly people in it, and the distance from that household to the nearest tank. We then use the features of the Cuxfilter library to specify which tools and variables to allow the users to interact with. 

### All US Households Colored by Distance to Nearest Tank (Cuxfilter): interactive visualizations using GPUs
This visualization is very similar to the ones described above, but it instead plots all US households with children, coloring the points by their distances to nearest tanks. It was created using the pre-processed data as well, in addition to the Cuxfilter library and GPUs, but this time with a number of observations orders of magnitude larger than used for the case studies. 

### All US Households with Natural Hazard Sliders (Cuxfilter): interactive visualizations using GPUs
This visualization is similar to the ones described above, but in addition to allowing the user to select to view households within a certain distance from tanks, the user can also choose to view households within a certain national risk index for each relevant natural hazard (tornadoes, hurricanes, strong winds, coastal floods, riverine floods, and earthquakes). It is created with the Cuxfilter library. 

### Address Lookup Web App (Folium): interactive Web App for real-time searching and display of select points
This interactive Web App uses the data processing workflows explained in detail above to create a visualization displaying the ten nearest tanks to any address the user inputs into a search bar.

It uses geocoding through the OpenStreetMap and GoogleMaps Application Programming Interfaces (APIs) to convert the address inputted by the user into latitude and longitude coordinates, then finds the ten storage tanks nearest to that address and calculates the distance to the nearest one. In addition, it displays a “Risk Index”, an indicator that uses information of that address’ proximity to a tank and the physical vulnerability of that tank to inform the user with an estimate of the risk of experiencing a tank spill at that specific address.

## User Instructions
### Step 1: Setup environment for project in the Duke Compute Cluster
Create a JupyterLab Singularity instance in an OnDemand session with or without GPUs and the amount of time, CPU cores (max is 40 cores), and RAM (max is 208 GB), depending on which visualizations you want to run. Only visualizations that require GPUs (such as our Worldwide Reddit Post Activity visualization) will need gpu-scavenger or gpu-common as the partition. Make sure to ask for GPUs (max of 2) if you want to run a GPU visualization.

### Step 2: Clone Gitlab repository
Copy the following command into your terminal once you are inside the directory you want this project folder to be in:
```
git clone git@gitlab.oit.duke.edu:at341/codeplus-celine-dcc-package.git
```
### Step 3: Open JupyterLab Notebooks + Run Code
Once the repository has been cloned, you can open the JupyterLab notebooks and begin running the code. For any visualizations that need GPUs to run, make sure under the Kernel tab that the rapids kernel is selected (should be selected by , otherwise, select the Python 3 kernel. 

If you encounter any issues with a Python library not being available/up-to-date/compatible, you can go to your terminal and run the command pip install “name of library” and that should install the library.