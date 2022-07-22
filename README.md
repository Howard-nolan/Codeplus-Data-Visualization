# Visualization in Python for Duke Data Scientists

![Example](/images/10_all_us_distances.gif)
![Example](/images/13_webapp_2.gif)

### Authors: Alyssa Ting, Susan Feng, Joey Nolan
#### About the Authors:
**Alyssa:** a rising second-year Duke student from São Paulo, Brazil. She is interested in studying both Computer Science and Statistical Sciences and hopes to work in Data Science in the future.

**Susan:**

**Joey:**

## Project Introduction 
This project was a part of the 2022 Duke University Code+ Program, a 10-week summer program designed for Duke University undergraduate students. The primary goal of this specific project, called Visualizations in Python, was to use state-of-the-art tools to develop interactive visualizations and workflows that would allow Duke researchers to analyze and visualize their large datasets in new and enhanced ways. The visualizations developed and documented for this project were created in JupyterLab notebooks hosted in a custom singularity. This singularity was then deployed on Duke’s OnDemand website, a virtual SLURM-based job scheduler using machines provided by the Duke Compute Cluster, in order to allow for a heavier use of computational resources.

### Our Researcher
The researcher we worked with is Celine Robinson, pursuing a Ph.D in Civil and Environmental Engineering concentrating in Systems, Risk, and Decision. The research we focused on while working with her pertains specifically to the risk of above-ground petrochemical storage tanks spilling, and the potential effects of these spills on the communities nearby. She is specifically interested in understanding these impacts on communities dense with children or eldery people, a focus which guided our work. 

### About the Data
This project works primarily with two datasets: the Above-Ground Storage Tanks (AST) dataset and the InfoUSA dataset, both provided to us by our researcher. In addition, we pulled government-owned natural hazard data to supplement our work. 

![Flowchart](/images/flowchart.png)
<img src="/images/flowchart.png" alt="flowchart" width="400"/>

#### AST Dataset
This dataset was collected by Celine, and contains nearly 100 thousand observations on petrochemical storage tanks across the United States. The data includes information such as the type of storage tank, diameter, and precise latitude and longitude coordinates for each storage tank. The full dataset is not available to the public, but we’ve provided a random sampling of it below: 

![Example](/images/01_synthetic_ast.PNG)

#### InfoUSA Dataset
This dataset was purchased by Duke University, and contains household-level census data separated by household, totalling 200 million observations across around 38 thousand different files. For each household it has data on, this dataset contains information on the number of children, age code, estimated income, latitude and longitude coordinates, and more. The work we have done focuses primarily on the number of children per household and the head of household’s age code, as well as each household’s latitude and longitude coordinates. This dataset is private, but we’ve provided a synthetic version of the data below:

[INSERT SYNTHETIC DATA]

#### National Risk Index Data, available [here](https://hazards.fema.gov/nri/data-resources).
This dataset was made publicly available by the Federal Emergency Management Agency (FEMA), and contains extensive information on natural hazards risks for each county across the country. The columns we were specifically interested in were the ones with Risk Index Score values for each county. The Risk Index Score is on a scale from 0 to 100, and was calculated by FEMA and indicates the relative risk of that county for a specific natural hazard. The natural hazards deemed relevant to our project by our researcher were tornadoes, hurricanes, strong winds, coastal floods, riverine floods, and earthquakes.

![Example](/images/03_nri_data.PNG)

#### Floodplain Data, available [here](https://www.fema.gov/flood-maps/national-flood-hazard-layer).
This dataset was made publicly available by FEMA, and contains information, including geometries, of over one million floodplains across the US. We specifically used the geometries provided to identify which tanks were on floodplains.

![Example](/images/04_floodplain_data.PNG)

### Tools
We used a variety of Python libraries and packages to clean, process, manipulate and visualization datasets with thousands to millions of observations.

#### Data manipulation and processing: Pandas, GeoPandas, NumPy, and Dask
**Pandas:** an open-source Python library that provides an easy and intuitive way to read, process, and write data. We use Pandas to read in our data from ```.csv```, ```.txt``` or ```.parquet``` files as a dataframe, then manipulate this dataframe through a variety of easy-to-use tools provided by the library. 

**GeoPandas:** an open-source Python library specifically focused on working with geospatial data. It provides much of the same functionality for manipulating and processing data as the Pandas libraries, but allows users to work with geospatial data (like points and geometries). We use GeoPandas to manipulate our spatial data in order to create visualizations.

**NumPy:** an open-source Python library used for scientific computing. It allows for fast and easy manipulation of data through the use of arrays. We use NumPy to process and manipulate a lot of our data before we visualize it through GeoPandas or the Cuxfilter libraries.

**Dask:** an open-source Python library for parallel computing. It allows users to efficiently perform processing and computing on large-scale data. We use it when working with dataframes with tens of millions of observations. Specifically, Dask allowed us to efficiently convert large pandas dataframes to the GeoDataFrames necessary for the code we run to process it later.

#### Visualizations: HoloViews and GeoViews
**HoloViews:** an open-source Python library specializing in data analysis and visualization. It incorporates visualization tools like bokeh and matplotlib to allow users to work seamlessly with the data and its visualization. We use it in order to process and visualize our large amounts of data, as it provides an immediate, automatic visualization rendered by a variety of supported plotting libraries, including Bokeh or Matplotlib.

**GeoViews:** an open-source Python library built on the HoloViews library allowing users to easily visualize multidimensional and geographical data. We use it to plot and visualize geographical data, such as to map tanks and households across the US.

#### Visualizations using Graphical-Processing Units (GPUs): Cuxfilter
**Cuxfilter:** an open-source Python library, part of the RAPIDS suite of open-source software libraries built to work with data science on GPUs. This specific library seamlessly connects different visualization libraries such as bokeh and datashader to a GPU dataframe. We use this library to plot amounts of data orders of magnitudes larger than that we plot on HoloViews and GeoViews, all within seconds.

## Data Merging and Wrangling Workflow Overview
Make a flowchart :)))))) at the end!

To understand this process in more detail, view the data manipulations README [here](https://gitlab.oit.duke.edu/at341/codeplus-celine-dcc-package/-/tree/master/processing/README.md).

The source data we were provided were both significantly large, and required a lot of cleaning, processing and manipulation in order to create meaningful visualizations. We have numerous notebooks dedicated specifically to the processing and manipulations we did to each dataset for our visualizations, which you can look into for more detail. In general, however, we worked with file merging, data processing, transformation between different coordinate systems, nearest neighbor analysis, and spatial joins.  

### File Merging: merging thousands of ```.txt``` files and writing the merged file as a ```.parquet``` file

The InfoUSA dataset has household-level census data for 15 years, 2006 to 2020. Our work uses only the 2020 data. The 2020 data alone contains 38,248 ```.txt``` files, each file representing the census data per each household in one zip code. However, in order to analyze and visualize this data meaningfully, we used the ```pyarrow``` engine and the ```.parquet``` file format in order to combine and store all this data in one, merged ```.parquet``` file. With more than 190 million rows and 10 columns, it takes less than 30 seconds to read in (using 8 CPUs and 32GB RAM). 

### Data processing: filtering, merging and aggregating data
We were provided multiple sources of data, each of which include extensive amounts of information. In each of these datasets, we are only use the few number of columns relevant to our work, and so in order to make our work more readable and memory-efficient, we have multiple processing notebooks in which we filter through our data for select columns, and then save the narrowed-down versions of those datasets. In addition, in order to visualize our data in meaningful ways, we merged data from various different sources in order to gain insight on the nuances of the data. And finally,  we performed numerous different aggregations of the data in order to get a more generalized view of certain aspects.

### Nearest Neighbor Analysis: calculating the shortest distance between two points
We used a version of nearest neighbor analysis to investigate the distances between households and the storage tank nearest to them. Our approach takes code written and published by the University of Helsinki, which finds and calculates the distances between buildings and the nearest transport stop. It uses the ```BallTree``` method from scikit-learn, an open-source machine learning Python library, which uses machine learning to identify the tank closest to 53 million households in less than an hour. We then use the coordinates of each household and tank to calculate the distance between the two. 

### Transformations between coordinate systems: converting from EPSG 4326 to EPSG 3857
As we are working with a variety of spatial data, largely information on the latitude and longitude of certain points like households or tanks, it was necessary for us to always be mindful of the coordinate system or projection our data was in, as well us the system our visualizations library needed it to be in. Our data was mostly in EPSG 4326, which we then transformed to EPSG 3857 so that it could be displayed using the cuxfilter library.

### Spatial joins: using ```.sjoin()``` to find the intersections between geometries
We took advantage of the GeoPandas library’s ```.sjoin()``` function to produce a few of our visualizations. This function performs a spatial join of two GeoDataFrames, and by setting the ‘predicate’ parameter to ‘intersects’, outputs a new GeoDataFrame which only includes the observations with geometries that are the intersections of the two original GeoDataFrames.

## Visualizations

To understand this process in more detail, view the visualizations README [here](https://gitlab.oit.duke.edu/at341/codeplus-celine-dcc-package/-/tree/master/visualizations/README.md), as well as each Jupyter Notebook.

### Stacked Bar Graph of Tank Types per State (HoloViews): using Pandas ```.groupby()```, ```.size()``` and ```.pivot_table()```
This visualization uses the matplotlib backend in Holoviews to display information from the AST dataset. It outputs a stacked bar graph illustrating the exact breakdown of each tank type per state, as well as the number of tanks in each state. 

![Example](/images/05_stacked_bar.PNG)

### Number of Children Per County (GeoViews): using pandas ```.groupby()``` and ```.sum()```
This visualization uses the GeoViews library to display a map of the US, broken down at the county-level with each county colored by its total number of children, as processed from the InfoUSA dataset. It also uses the AST dataset to plot points overlay points for each storage tank on top of this map.

![Example](/images/06_children_county.gif)

### Number of Households Near Tanks per County (GeoViews): using GeoPandas ```.sjoin()```
This map of the US was created using GeoViews, and displays each county colored by the number of households in that county that are within five miles from a tank (a boundary provided to us by our researcher). 

![Example](/images/07_hh_count_county.gif)

### Charleston and Harris County Case Studies (Cuxfilter): interactive visualizations using GPUs
These two visualizations were created using the Cuxfilter library, which allows users to plot a large amount of data to create customizable and interactive dashboards using GPUs. Each visualization displays points for all households and tanks in that county, then allows the user to customize which points they would like to see: depending on distance from the nearest tank, as well as whether or not the household has elderly people or children. 

![Example](/images/08_charleston_case_study.gif)
![Example](/images/09_harris_case_study.gif)

### All US Households Colored by Distance to Nearest Tank (Cuxfilter): interactive visualizations using GPUs
This visualization is very similar to the ones described above, but it instead plots all US households with children, coloring the points by their distances to nearest tanks. It was created using the pre-processed data as well, in addition to the Cuxfilter library and GPUs, but this time with a number of observations orders of magnitude larger than used for the case studies. 

![Example](/images/10_all_us_distances.gif)

### All US Households with Natural Hazard Sliders (Cuxfilter): interactive visualizations using GPUs
This visualization is similar to the ones described above, but in addition to allowing the user to select to view households within a certain distance from tanks, the user can also choose to view households within a certain national risk index for each relevant natural hazard (tornadoes, hurricanes, strong winds, coastal floods, riverine floods, and earthquakes).

![Example](/images/11_coastal_flooding.gif)

### Address Lookup Web App (Folium): interactive Web App for real-time searching and display of select points
This interactive Web App uses the data processing workflows explained in detail above to create a visualization displaying the ten nearest tanks to any address the user inputs into a search bar.

![Example](/images/12_webapp.gif)

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