[TOC]

# 1. Introduction

Academic communities and public sectors have been using mobility and traffic data to address social issues  (i.e GDP prediction, environmental impacts, crime, etc). One method to measure mobility and traffic state is  by analyzing a series of  pictures and footage from traffic cameras installed at fixed locations.



# 2. Data and Data Processing

### 2.1. Primary Data Sources

#### 2.1.1 Image Dataset captured from traffic cameras


#### 2.1.2. Vehicle Count For Traffic Cameras

The Surrey vehicle count dataset consists of 245,499 records extracted from traffic footages using Computer Vision methods (details introduced in Section 3), taken at 1-hour interval from December 01, 2020 to December 31, 2020 for 341 camera stations across Surrey, BC, Canada. Variables of interests include counts for count for car, bicycle, bus, person, motorcycle, truck.

// TODO: Make a graph for different vehicle type
// Distribution




The vehicle counts dataset provided by our sponsor, the Cedar Academy Society consists of the amount of vehicles and people detected by the original computer vision algorithm in each camera station. Each type of count has its own section, its associated file image, and its coordinates in latitude and longitude. Those counts were: 


 
### 2.2. Other Data Sources


#### 2.2.1. Business
A list of all registered business data within the city of Surrey was provided. Among the dataset, we excluded 5% of the entries that did not contain a buisness name as they were registered as a home occupation.The important entries that were taken into consideration were the name of the business, its category, its status (whether if it is still active or terminated) and the coordinate locations. Unfortuantely, it did not contain any data on earnings as our initial intention was to investigate on how the trends in traffic pattern affect public business revenues given the COVID-19 pandemic. Nevertheless, we used the nearby traffic cameras within close proximity of businesses as well as the intensity of the traffic activity to guage earning performance. In other words, we operated under the notion that as traffic intensity increases, so does the performance of a business.
 
The status of the business was divided into 10 categories. However, we focused on the businesses who had an active status. Within the dataframe, 80% of the businesses were active and therefore, we disregarded 20% of businesses as they were either in various stages of approval, or were terminated or rejected.

#### 2.2.2. COVID-19 Data
Provincial COVID-19 lab data dating from Janurary 2020 to July 2021 was taking from the BC CDC website. Each entry included the date and the health authority region. Within the date and the health authority region, there are also data entries that highlight the type of tests. Specifically, the tests can be broken into 4 categories:

- The number of new tests which describes the cumulative number of new COVID-19 tests performed within the last 24 hours prior to the update of the ArcGIS dashboard found on the BC Centre for Disease Control
- The number of total tests which describes the cumulative number of COVID-19 tests since testing commenced mid-January
- The positivity rate which is calculated as an average of a 7 day period as a ratio of test specimens that were confirmed positive to the total amount of specimens being tested

- The turn around time which is calculated as the daily average time between a collection of a specimen and the report of a test result.

We used this data source as a confounding variable to determine if the relationship between the business activities and the traffic patterns within the city of Surrey was influenced by the COVID-19 pandemic. As such, we focused on the cases within the Fraser Health Region (as that is where the city is located in) as well as the dates in October and December to see the change in business activity. 

#### 2.2.3. Weather
The hourly climate data taken from the Government of Canada's past weather and climate database was a resource that we briefly implemented in our analysis of determining potential undercounting biases. The nearest station we selected in order to analyze weather patterns was Pitt Meadows as it contains hourly data in which, with slight aggregation of the two minute data provided by our sponsor, will hopefully provide adquate analysis to determine potential biases in traffic camera images detected by the original algorithm. 

The rationale for implementing weather data was that we infered that weather conditions, especially rain, could affect the traffic cameras and as a result, may distort the images and thus, rendering biases when passed into the original detection algorithm.



#### 2.2.4. Day and Night time
The detection algorithm is severely vulnerable to vehicle's **over-exposed headlight issue** during night time. To help reduce the detection biases due to this reason, we applied several image-preprocessing methods(i.e. gamma adjustment, histogram equalization, etc.) with reasonable parameters to the filtered night-time image dataset to see the overall performance. 

The night-time image dataset was generated by filtering out nightime images within a specific time period. Our initial step was to find the dusk and sunrise times for each day in the month of December using the `astral` package as we wanted to compare the difference in the biases between dusk, the darkest part of the evening, and sunrise, where the camera sees initial light. Next, we took the average dusk and sunrise times of all 31 days of December. From there, we filtered out the nighttime image datset between our averaged dusk and sunrise time obtained from the previous step.

#### 2.2.5. Bike Lanes
A dataset of all bike routes within the city of Surrey current to 2020 was provided. The bike routes contained 7 different type of lanes and current status of the bike routes were placed in 3 categories: 
- `In Service` for currently operating routes, 
- `Proposed` for future or planned bike routes for development
- `NA` for bike routes that do not fall into either the `In Service` nor the `Proposed` bike routes.

The main use of this dataset was to utilize it in our later development of our R Shiny app for traffic analysis. Since the initial datasets provided from our sponsor were from the month of December, we decided to only include serviceable bike routes. Therefore, we excluded 11% of the dataset as they were either listed as `Proposed` or `NA`.

#### 2.2.6. Boundaries 
A`GEOJSON` datafile that consists of the administrative boundary lines of the city of Surrey along with its neighbourhoods of Guildford, Whalley, City Centre, Cloverdale, Fleetwood, Newton,South Surrey, and Whalley was provided.

https://dsi.ubc.ca/sites/default/files/dssg-finalreport-surrey-ev.pdf <- take their datasource part as an example*




# Computer Vision Module
## Overview
## YOLO
## Undercount Issue
Current pipeline uses `detect_common_object()` function from cvlib. The version used to generated the dataset so far in the server uses **YOLOv3** model pretrained on COCO dataset capable of detecting 80 common objects in context. 
// Limitation of shapes/objects/resolution

*Problem:*
- High occulusion & low resolution: when vehicles line up at the intersection, they overlaps heavily with each other. Current pipeline fails to detect the semantic features when they interweave, thus is missing count at far distance when there is a long line-up. On the other hand, it is able to detect the vehicles correctly at far distance. the when the line is moving (the cars are more apart).

*Result:*
- **Saturation**: the maximum vehicle count at the example intersection is almost *capped at around 15* where one can easily find snapshot with 20+ cars under visual check.
- **Random noise**: when the road is busy, the vehicle counts shows obvious fluctations as line-up resulting from red light could appear out-of-sync from the snapshot frequency.

//TODO: Insert image examples

First, low **detected count** does not necessarily mean low traffic. In the below examples, the entire queue incoming is hardly captured.

Second, when switching between free-of-flow (**F**) and queue (**Q**) traffic state, the undercounting issue occurs inconsistently. The top image should count more cars than the bottom one while the detected count captures the opposite.

## Nighttime Detection 





# Statistical Analysis

## Camera Comparisons

The initial objective of the project is to identify a correlation between camera installations and traffic states. The original proposal states that:

*Cameras are installed near locations with heavy traffic. This leads to a biased dataset and exaggerates nearby mobility levels due to preferential sampling.*


However, it has been later found out to **not** be the problem in our scenario, due to the facts that: 
* We are mostly interested in point of interest along the traffic.
* We **do not** interpolate traffic density off the lanes or areas far from the sampling locations. 

Our new objective is not totally deviated from the original goal, yet it shifts towards comparisons across cameras. We are interested in pairs of cameras with **high spatial correlations** and conduct **comparisons** of their vehicle counts to see if they share similar traffic states concurrently. AM(7-10) and PM(16-19) rush hours are targeted due to intense traffic activities over the periods. 

The pair of spatially correlated cameras are identified as either:

- Two cameras installed on the **same intersection**
- Two cameras which are **spatial nearest neighbors**

**Note**: The nearest neighbor is not always symmetric, and the next-selected camera has to be the nearest neighbor to the prev-selected camera, i.e. if camera a's nearest neighbor is camera b, yet camera b's nearest neighbor is camera c, then you should select a then b to form a valid pair, but not the other way round.

### Data Prep
First of all, we filtered out all the intersections/roads with **at least two** cameras installed. Among all 31 of them, we have selected a sample size of **10** POIs to conduct the two types of tests stated above.

![](https://i.imgur.com/5Dc6SOf.png)


We would then group the filtered intersections by `street_address` and `time` so as to retrieve all the pairs, and use `slice()` to separate the pair and create a new categorical column to classify `Pair 1` and `Pair 2` data. 

The data for the nearest-neighbor cameras were retrieved through the `st_nn()` method from the `nn_geo` package with `k=1`, and were  merged to the dataset and classified as `Pair 3`.


### Paired t-test and Two-sample t-test
The comparisons between cameras were conducted through a paired t-test for same-intersection cameras and a two-sample t-test for nearest-neighbor cameras. To simplify the problem, we temporarily excluded the spatial covariates, and performed an overall test for all sampled locations.

To further investigate any correlation between the difference in camera pairs and the temporal factor, we filtered out the data for AM and PM rush hours accordingly and ran the tests again.   

### Chi-squared test for temporal covariate (TODO)

### Results and Analysis (TODO)

#### Temporal effect - AM vs PM
Time also plays a significant role in explaining the differences in car counts between the camera pairs.  

Overall, based on the December 2020's hourly data, there has been found a **greater** difference during **PM** rush hours compared to AM. 

However, this may vary across different locations and requires further experiments with more data. 


## Correlation of Traffic Activity and COVID-19 Pandemic Activity

In addition to investigating high spatial correlations and comparisons of vehicle counts, we were also interested in investigating whether the COVID-19 pandemic had a
confounding effect on the relationship between the economic recovery of small businesses and nearby traffic activity. Furthermore, we utilized the different stages in B.C's reopening plan to measure changes in traffic activity.

As such, we utilized the traffic datasets from October and December to measure any changes. It is important to note that the government of B.C introduced rigid  restrictions in the middle of November which lasted until January of 2021. We initially predicted that the traffic activity will be lower in December compared to previous months. Our null hypthesis assumes that there is no relationship between the effect of COVID-19 cases and traffic count while our alternative hypothesis proposes the contrary. 

### Initial visualization
To visually confirm the changes in traffic activity from October to December, we constructed our heatmaps in which it plotted the average car counts of each traffic station of the entire month. We chose to plot the car counts as it has enough sample sizes for us to categorize the different intensity levels of the traffic activity. Our initial heatmaps revealed the decrease in traffic activity in December compared to October. 

The results of the initial heatmap plots may have alluded that the decrease in traffic activity may also result in the decrease of business activity and that the restrictions as well as the increased number of COVID-19 cases may play a contributing factor in this observation.

### Specific Camera Analysis

Next, we focused our attention of the heatmap activity near 152St. Since 152St contains a diverse amount of businesses from different industries, we infered that we are able to construct a general assumption on the economic situation of Surrey. We also chose the camera near 152St and 104Ave as that camera is near the Guildford Mall- a high profile location for economic activity. Finally, we plotted the traffic camera activity for 152St and 104Ave as well as the positivity rate for COVID-19 cases for both October and December respectively

### Results
As seen in the image below, the p-value is lower than 0.05 and thus, supporting our null hypothesis

[insert image]





# Discussion 

## Camera POV
// TODO: Camera POV Issue




# Unbiased Mobility Web App

## UI Features 
In general the app uses the two maps for its basemap. The Positron and the Dark Matter basemaps found [here](https://carto.com/blog/getting-to-know-positron-and-dark-matter/). The Positron map is used in the Camera Map menu to assist users to street names and routes while the DarkMatter map is used to assist in heatmap visualization in order to predict traffic patterns.

### Business Overlays

There are 6 business overlays included in this application. All of them are clustered into groups. Those are:    
-  Stores
-  Foods and Restaurants
-  Liquor Stores
-  Health and Medicine
-  Businesses and Finance
-  Services.  

Services are a vague term to encorporate businesses that are not categorized into the other 5 categories.  This does not include home based businesses.  As the map is zoomed in, the clusters become more dispersed and each individual business icon is shown more in detail. In order to view the name of the business, the cursor must be hovered ontop of the icon.

### Nearby Cameras of Businesses

As one may notice, there is another option in the business overlays' drop-down called `Nearby Cams`. This toggle control will allow the user to locate nearby cameras upon selecting on a business marker and to view the traffic counts captured by them in the explorer panel. This is aimed to give user a brief understanding of the nearby traffic states of a business point of interest.

The range within which you wish to find nearby cameras can also be customized through the `red gear button` on the top left corner, and the radius is adjustable in **meters** through a slider input. Notice that the current maximum number of nearby cameras is set to default **3** (considering the basic usage), so the highlighted cameras will not exceed 3 however you adjust the radius. 

### Heatmap

An option to switch to the heatmap view is provided on the sidebar on the left labelled `Heatmap`. Similar to the basemap in the `Camera Map`, the heatmap has boundaries of all the neighbourhoods of Surrey. However unlike the boundaries found in `Camera Map`, where the user can filter out individual neighbourhoods, the boundaries for `Heatmap`is only fixed to the entire city of Surrey. As of now, the heatmap only displays car count. 


### Bicyle Routes
An overlay of bicycle routes is included in this application along with the overlay panels. Proposed bicycle routes are not included in this overlay. To view the bicycle routes, ensure that the label "Bike Routes " is checked. The bicycle routes are coloured in 7 types. This is shown in the legend listed in the top left corner. It is important to note that the legend is partially hidden from the menu and is undraggable. In order to fully view the legend, the panel (with  the 3 bars) must be collapsed.

### Neighbourhood Filters 
A selection of neighbourhoods is available for selection. The following neighbourhoods are:
-  City Centre for `CITY CENTRE` 
-  Cloverdale for `CLOVERDALE`
-  Fleetwood for `FLEETWOOD`
-  Newton for `NEWTON`
-  South Surrey for `SOUTH SURREY`
-  Whalley for `WHALLEY`

The option panel is on the left hand side and it is titled "Select a Neighbourhood". The default selected is `SURREY`, where it displays all the neighbourhoods within the city of Surrey. When a neighbourhood is selected, the map will shift its view to the selected neighbourhood.




## Acknowledgement