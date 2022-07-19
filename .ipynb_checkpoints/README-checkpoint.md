# Visualization in Python for Duke Data Scientists

# Authors: Alyssa Ting, Susan Feng, Joey Nolan

## Project Introduction 
This project was a part of the 2022 Duke University Code+ Program, a 10-week summer program designed for Duke University undergraduate students. The primary goal of this specific project, called Visualizations in Python, was to use state-of-the-art tools to develop interactive visualizations and workflows that would allow Duke researchers to analyze and visualize their large datasets in new and enhanced ways. The visualizations developed and documented for this project were created in JupyterLab notebooks hosted in a custom singularity. This singularity was then deployed on Duke’s OnDemand website, a virtual SLURM-based job scheduler using machines provided by the Duke Compute Cluster, in order to allow for a heavier use of computational resources.

### Our Researcher
The researcher we worked with is Celine Robinson, pursuing a Ph.D in Civil and Environmental Engineering concentrating in Systems, Risk, and Decision. The research we focused on while working with her pertains specifically to the risk of above-ground petrochemical storage tanks spilling, and the potential effects of these spills on the communities nearby.  

[ADD THAT SHE IS SPECIFICALLY INTERESTED IN CHILDREN AND ELDERLY]

### About the Data
This project works primarily with two datasets: the Above-Ground Storage Tanks (AST) dataset and the InfoUSA dataset. 

#### AST Dataset
This dataset was collected by Celine, and contains nearly 100 thousand observations on petrochemical storage tanks across the United States. The data includes information such as the type of storage tank, diameter, and precise latitude and longitude coordinates for each storage tank. The full dataset is not available to the public, but we’ve provided a random sampling of it below: 

[INSERT RANDOMLY SELECTED DATA]

#### InfoUSA Dataset
This dataset was purchased by Duke University, and contains household-level census data separated by household, totalling 200 million observations across around 38 thousand different files. For each household it has data on, this dataset contains information on the number of children, age code, estimated income, latitude and longitude coordinates, and more. The work we have done focuses primarily on the number of children per household and the head of household’s age code, as well as each households latitude and longitude coordinates. This dataset is private, but we’ve provided a synthetic version of the data below:

[INSERT SYNTHETIC DATA]

#### National Risk Index Data, available at https://hazards.fema.gov/nri/data-resources
This dataset was made publicly available by the Federal Emergency Management Agency (FEMA), and contains extensive information on natural hazards risks for each county across the country. The columns we were specifically interested in were the ones with Risk Index Score values for each county. The Risk Index Score is on a scale from 0 to 100, and was calculated by FEMA and indicates the relative risk of that county for a specific natural hazard. The natural hazards deemed relevant to our project by our researcher were tornadoes, hurricanes, strong winds, coastal floods, riverine floods, and earthquakes.

[INSERT SCREENSHOT OF DATA]

#### Floodplain Data, available at https://www.fema.gov/flood-maps/national-flood-hazard-layer
This dataset was made publicly available by FEMA, and contains information, including geometries, of over one million floodplains across the US. We specifically used the geometries provided to identify which tanks were on floodplains.

[INSERT SCREENSHOT OF DATA]

### Tools
We used a variety of Python libraries and packages to clean, process, manipulate and visualization datasets with thousands to millions of observations.

#### Data manipulation and processing: Pandas, GeoPandas, NumPy, and Dask
**Pandas**: an open-source Python library that provides an easy and intuitive way to read, process, and write data. We use Pandas to read in our data from ```.csv```, ```.txt``` or ```.parquet``` files as a dataframe, then manipulate this dataframe through a variety of easy-to-use tools provided by the library. 

GeoPandas: an open-source Python library specifically focused on working with geospatial data. It provides much of the same functionality for manipulating and processing data as the Pandas libraries, but allows users to work with geospatial data (like points and geometries). We use GeoPandas to manipulate our spatial data in order to create visualizations.

NumPy: an open-source Python library used for scientific computing. It allows for fast and easy manipulation of data through the use of arrays. We use NumPy to process and manipulate a lot of our data before we visualize it through GeoPandas or the Cuxfilter libraries.

Dask: an open-source Python library for parallel computing. It allows users to efficiently perform processing and computing on large-scale data. We use it when working with dataframes with tens of millions of observations. Specifically, Dask allowed us to efficiently convert large pandas dataframes to the GeoDataFrames necessary for the code we run to process it later.

#### Visualizations: HoloViews and GeoViews
HoloViews: an open-source Python library specializing in data analysis and visualization. It incorporates visualization tools like bokeh and matplotlib to allow users to work seamlessly with the data and its visualization. We use it in order to process and visualize our large amounts of data, as it provides an immediate, automatic visualization rendered by a variety of supported plotting libraries, including Bokeh or Matplotlib.

GeoViews: an open-source Python library built on the HoloViews library allowing users to easily visualize multidimensional and geographical data. We use it to plot and visualize geographical data, such as to map tanks and households across the US.

#### Visualizations using Graphical-Processing Units (GPUs): Cuxfilter
Cuxfilter: an open-source Python library, part of the RAPIDS suite of open-source software libraries built to work with data science on GPUs. This specific library seamlessly connects different visualization libraries such as bokeh and datashader to a GPU dataframe. We use this library to plot amounts of data orders of magnitudes larger than that we plot on HoloViews and GeoViews, all within seconds.

## Data Merging and Wrangling Workflow Overview
Make a flowchart :)))))) at the end!

The source data we were provided were both significantly large, and required a lot of cleaning, processing and manipulation in order to create meaningful visualizations. We have numerous notebooks dedicated specifically to the processing and manipulations we did to each dataset for our visualizations, which you can look into for more detail. In general, however, we worked with file merging, data processing, transformation between different coordinate systems, nearest neighbor analysis, and spatial joins.  

### File Merging: merging thousands of .txt files and writing the merged file as a .parquet file
The InfoUSA dataset has household-level census data for 15 years, 2006 to 2020. Our work uses only the 2020 data. The 2020 data alone contains 38,248 .txt files, each file representing the census data per each household in one zipcode. However, in order to analyze and visualize this data meaningfully, we used the pyarrow engine and the .parquet file format in order to combine and store all this data in one, merged .parquet file.  

Since each file had around a few thousand observations each and 54 columns, merging thousands of these files and storing them into a single CSV file quickly became tough on both processing power and memory usage– although we were able to merge all 38,248 files and write it as a .csv file, we weren’t able to read in that .csv file as it took up too much memory. 

To get around this issue, we chose to use the pyarrow engine when reading in each .txt file, merge it with the other files, and then save the combined files as a .parquet file. This engine allows for multithreading when reading in files, which allows for a better, more efficient use of the CPU cores, thus resulting in a faster runtime. We also chose to save the complete combined file as a Parquet file, which is optimized for storing and reading large sets of data. As it is column-based as opposed to row-based, like a CSV file, a Parquet file only focuses on the data relevant to the user’s query. In a one-hundred column table, for example, if the user is only using a few of these columns, Parquet files enables the user to fetch only the required columns and values and load those into memory. Parquet files can also be read in and exported using pandas, which is an added bonus when working with pandas dataframes.

However, even using Pyarrow and Parquet, we were unable to combine all 38,248 files in one go– running into memory issues even when using 208GB of RAM, we decided to then part our data into four subsets, combine each of those subsets, filter for only the columns we will use, then merge them to each other for our final, combined file. This file was stored as a Parquet file, and with more than 190 million rows and 10 columns, it takes less than 30 seconds to read in (using 8 CPUs and 32GB RAM). 

### Data processing: filtering, merging and aggregating data
We were provided multiple sources of data, each of which include extensive amounts of information. In each of these datasets, we are only use the few number of columns relevant to our work, and so in order to make our work more readable and memory-efficient, we have multiple processing notebooks in which we filter through our data for select columns, and then save the narrowed-down versions of those datasets. 

In addition, in order to visualize our data in meaningful ways, we merged data from various different sources in order to gain insight on the nuances of the data. For example, we were interested in investigating the exposure of petrochemical tanks to a variety of natural hazards, such as earthquakes and hurricanes. To do so, we classified each tank by the county in which it was found, and merged this dataframe with the National Risk Index data, which provided risk index values for each natural hazard for each county in the United States. Like this, we were able to associate a risk index for various natural disasters to each tank. We then used our Nearest Neighbor Analysis code (to be explained in further detail below) to find the tank nearest to each household, and associate the risk of each tank spilling to a number of households. Like this, we were able to visualize which households and areas in the United States were more vulnerable to tank spillages according to natural hazards. 

Finally, we performed numerous different aggregations of the data in order to get a more generalized view of certain aspects. An example of this is our aggregated ‘ChildrenHHCount’ column in the data, taking it from the number of children per household and summarizing it to find the number of children in each county. Doing so, we were able to provide our researcher a way to identify areas in the US more dense in children to focus her analysis. 

### Nearest Neighbor Analysis: calculating the shortest distance between two points
We used a version of nearest neighbor analysis to investigate the distances between households and the storage tank nearest to them. To do this analysis, we first developed a brute-force algorithm that would take each household, loop over each tank in the AST dataset, calculate the distance between those two points, and keep the shortest distance. This nested for loop method, when working with tens of millions of households and nearly a hundred thousand tanks, quickly proved both inefficient and unfeasible. Therefore, after research, we found code written and published by the University of Helsinki, which finds and calculates the distances between buildings and the nearest transport stop. 

This code uses scikit-learn, an open-source machine learning Python library that provides simple, yet efficient tools for predictive data analysis. From this library, the code published by the University of Helsinki specifically uses the BallTree method in the library’s neighbors module. This unsupervised learning method finds a predefined number of training samples closest in distance to the new point, and uses this to intelligently pinpoint each point nearest to the point of interest. Using this method, we were able to identify the tank closest to 53 million households in less than an hour. 

However, to ensure the accuracy of the method published by the University of Helsinki, we used both their algorithm and our brute-force algorithm on a small subset of households and tanks. Comparing the results of the two, we found that the University of Helsinki’s algorithm was finding the same closest tanks to each household as our brute-force method, but the distance calculated was off by hundreds of meters in both the positive and negative directions. Using this information, taking a closer look at the University of Helsinki’s code, we identified that their method to calculate distance based on radians was slightly inaccurate as opposed to the one we used in our brute-force algorithm.

Thus, we only used the University of Helsinki’s code to efficiently and accurately identify the latitude and longitude coordinates of the tank nearest to each household, then used these coordinates and the coordinates of the household it is the nearest to in order to calculate the distance between the two points.

Even using this more efficient version of our code, we were only able to find the distances between each household and the tank nearest to it for around 53 million households, as we filtered through all the households for only those with children in them. Running our adapted algorithm on around 190 million points proved time and memory inefficient. Our researcher, Celine, noted that in the future we can use spatial joins to identify households within five miles of a tank, and then perform the distance calculations on this much smaller subset of the data.

### Transformations between coordinate systems: converting from EPSG 4326 to EPSG 3857
As we are working with a variety of spatial data, largely information on the latitude and longitude of certain points like households or tanks, it was necessary for us to always be mindful of the coordinate system or projection our data was in, as well us the system our visualizations library needed it to be in. 

EPSG 4326 refers to a coordinate system within a database of coordinate system information published by the European Petroleum Survey Group (EPSG). It is, specifically, latitude and longitude coordinates on the WGS84 reference ellipsoid, and is the coordinate system our InfoUSA and AST data are in. 

The cuxfilter library, though, uses a world map background provided by OpenStreetMaps, which uses EPSG 3857 (a Spherical Mercator projection coordinate system). Therefore, every time we used the cuxfilter library to plot our points, we used the pyproj interface, which allows us to use the PROJ coordinate transformation software to transform our EPSG 4326 coordinates to EPSG 3857. 

### Spatial joins: using .sjoin() to find the intersections between geometries
We took advantage of the GeoPandas library’s sjoin function to produce a few of our visualizations. This function performs a spatial join of two GeoDataFrames, and by setting the ‘predicate’ parameter to ‘intersects’, outputs a new GeoDataFrame which only includes the observations with geometries that are the intersections of the two original GeoDataFrames.

This proved useful when classifying the tanks by county, for example, as we were only provided the geometries of each tank. When conducting a spatial join between this GeoDataFrame and another GeoDataFrame including geometries for each county across the US, we were able to identify which tanks belonged to each county. This was done by looping over each county in the counties GeoDataFrame, performing a spatial join with that geometry and the AST dataset to identify which tanks belonged to that county, and adding a column to the corresponding tank in the AST dataset indicating that county. This crucial added information on each tank allowed us to make the risk visualizations as detailed above. In addition, this logic also allowed us to count the number of households within five miles of a tank in each county, providing Celine with a way to identify which counties to focus her research on.

## Visualizations
### Stacked Bar Graph of Tank Types per State (HoloViews): using Pandas groupby, size and pivot_table
This visualization uses the matplotlib backend in Holoviews to display information from the AST dataset. It outputs a stacked bar graph illustrating the exact breakdown of each tank type per state, as well as the number of tanks in each state. 

To create it, we used various different functions to manipulate the AST data as a Pandas dataframe. First, the .groupby() function groups the data by state as well as tank type, then the .size() function returns the number of each group– in this case, the number of each type of tank in each state. Using these functions consecutively returns a new dataframe listing the state, the tank type, and the number of that type of tank in that state. This dataframe is then converted to a pivot table using the .pivot_table function, where the columns are tank types and the rows are states. The horizontal bar chart function in matplotlib, with parameter ‘stacked’ set to True, outputs the stacked bar graph visualization. 

### Number of Children Per County (GeoViews): using pandas groupby and sum
This visualization uses the GeoViews library to display a map of the US, broken down at the county-level with each county colored by its total number of children, as processed from the InfoUSA dataset. It also uses the AST dataset to plot points overlay points for each storage tank on top of this map.

To create it, we used pre-processed InfoUSA data that has the number of children in each zip code. Using the pandas groupby function as above, we group this data by county and use the sum function to sum the number of children for all zip codes in a county, thus creating a dataframe with each county and the total number of children for that county. We then merged this dataframe with another dataframe we pulled from the US government’s cartographic files database, which provides geometries for each county. Combining these two dataframes allows us to plot this data spatially, mapping all counties to form a map of the US and coloring it by the number of children in that county. We then converted the latitude and longitude coordinates of each tank to Point geometries, and plotted those in a similar fashion, then used GeoViews’ * operator to overlay the two visualizations. 

### Number of Households Near Tanks per County (GeoViews): using GeoPandas sjoin
This map of the US was created using GeoViews, and displays each county colored by the number of households in that county that are within five miles from a tank (a boundary provided to us by our researcher). 

To find the number of counties within five miles of a tank for each county, we used the GeoPandas’ sjoin function. This function, as detailed in the Data Merging and Wrangling Workflow Overview, can be used to find the intersection between two spatial dataframes. Using a pre-processed dataset including distances from each household to the tank nearest to it, we filtered to only keep the households within five miles of a tank. Then, we looped over a dataframe with geometries for each county, provided by the US government’s cartographic files database, to identify which tanks were in each county, and included the length of that dataframe using the len function as a new column in the counties GeoDataFrame. The final output was a GeoDataFrame with each county, the number of households within five miles of a tank in that county, and that county’s geometry. This is the dataframe used in GeoViews to plot the visualization below. 

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

git clone git@gitlab.oit.duke.edu:at341/codeplus-celine-dcc-package.git

### Step 3: Open JupyterLab Notebooks + Run Code
Once the repository has been cloned, you can open the JupyterLab notebooks and begin running the code. For any visualizations that need GPUs to run, make sure under the Kernel tab that the rapids kernel is selected (should be selected by , otherwise, select the Python 3 kernel. 

If you encounter any issues with a Python library not being available/up-to-date/compatible, you can go to your terminal and run the command pip install “name of library” and that should install the library.