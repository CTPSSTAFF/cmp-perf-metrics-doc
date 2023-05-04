# Calculation of CMP Performance Metrics

## _Abstract_

This document describes the process for calculating performance metrics for express and arterial highways for the CTPS Congestion Management Process (CMP). The process for calculating performance metrics for the two classes of roadways are mostly the same, but there are differences, and these are called out in detail.

## _Limitations_

Whenever the metrics produced by these calculations are reported or published (either directly or via an application), [INRIX](https://inrix.com/) must be cited as the source of the input speed and travel-time data used to calculate the metrics.

## _Use Cases_

-   Congestion Management Process
-   Long Range Transportation Plan
-   Transportation Improvement Program
-   Travel Demand Model
-   Corridor studies

## _Dependencies_

-   Access to INRIX speed and travel-time data is required. Since 2019, this has been through the [RITIS](https://ritis.org/) platform, available to CTPS under MassDOT license. Prior to this, CTPS purchased speed and travel time data directly from INRIX.
-   [Google Cloud Storage](https://cloud.google.com/storage/?) - for storing very large data sets, in particular those processed by Google BigQuery
-   [Google BigQuery](https://cloud.google.com/bigquery) - for querying very large datasets (up to petaBytes in size) efficiently
-   [Google Cloud Command Line Interface (gcloud CLI)](https://cloud.google.com/cli) - command-line interface to Google cloud tools, e.g., BigQuery

## _Inputs_

-   INRIX speed and travel time data
-   INRIX TMC network for Massachusetts
-   List of TMCs for Express Highways in the region
-   List of TMCs for Arterial Highways in the MPO area
-   List of dates used for this year's CMP
-   List of AM and PM peak periods for Express Highways
-   List of AM and PM peak periods for Arterial Highways
-   Conflation of INIRX TMC network with Massachusetts Road Inventory

### INRIX Speed and Travel Time Data

Raw speed and travel time data downloaded from RITIS (or purchased directly from INRIX) is in CSV file format, and has the following columns:

|     |     |     |
| --- | --- | --- |
| **Field** | **Contents** | **Datatype** |
| tmc\_code | TMC identifier | text |
| measurement\_tstamp | date and time (see below for format) | text |
| speed | average observed speed for the time period beginning *measurement\_tstamp* | floating point |
| average\_speed | see below | integer |
| reference\_speed | see below | integer |
| travel\_time\_minutes | average travel time through the TMC for the time period beginning *measurement\_tstamp*, in minutes | floating point |
| confidence\_score | always = 30; see below | integer |
| cvalue | see below | floating point |

Notes on specific columns:

**measurement\_tstamp** - date-and-time stamp in the format \[m\]m/\[d\]d/yyyy \[h\]h:mm

Examples:

-   4/19/2023 9:59
-   4/19/2023 10:00
-   11/5/2022 0:01
-   11/10/2022 23:59

**average\_speed** - INRIX-computed average speed

**reference\_speed** - INRIX-computed reference speed

**confidence\_*****score** -* This field has 3 possible values: 30, 20, 10. However, CTPS only uses (and thus only downloads) records with a **confidence*****\_*****score** of 30. Descriptions of the 3 possible values:

**30**: Real-time data. Any segment that has adequate data, at any time of day, will report real time data.

**20**: Historical average.  Between 4 am and 10 pm, any segment without sufficient real time data will show the historical average for that segment during that day/time period (15 minute granularity).

**10**: Reference speed.  From 10 pm to 4 am, any segment without sufficient real time data will show the reference speed for that segment. Any segment that does not have calculated historical averages will show the reference speed 24 hours a day if there is not sufficient real time data.

**cvalue** \- Indicates the probability that the current probe reading represents the actual roadway conditions based on recent and historical trends. CTPS only uses records with a **cvalue** of 75 or higher. **Note:** RITIS does not support filtering of data for extraction and download based on the value of the cvalue field (it does support filtering for extraction and download based on the value of the **confidence\_score** field); consequently, this filtering must be done as part of CTPS's processing.

### INRIX TMC Network for Massachusetts

The shapefile defining the INRIX CMP network for a given year must be obtained directly from INRIX; it is not available on the RITIS platform. (The shapefile for the much more limited NPMRDS CMP network, however, is available from RITIS.) Currently, CTPS's contact for this data is [Mike Massaro](mailto://mmassaro@inrix.com), CTPS's Sales Engineer at INRIX.

As delivered, the INRIX TMC shapefile uses the EPSG:4326 ("lat, lon") Spatial Reference System (SRS). The data is projected to the EPSG:26986 ("Mass State Plane, NAD 83, meters") SRS and stored in CTPS's enterprise Geodatabase ('mpodata'), currently hosted on appsrvr3. The name of the dataset is formed as follows:

mpodata.mpodata.INRIX\_MASSACHUSETTS\_TMC\_*yyyy*\_*n*

-   *yyyy* \- indicates the year of the data
-   *n* \- indicates the release number, within the given year, of the data

INRIX releases the TMC shapefile twice a year: once in the spring and once in the fall. The spring release is release #1; the fall release is release #2.  Hence the value of *n* may only be 1 or 2.

The metadata for the TMC shapefiles can be found in [INRIX metadata (on page 101, in Appendix A)](https://tetcoalition.org/wp-content/uploads/2015/02/I-95_VPP_II_Inteface_Guide-March_2018.pdf)

### Lists of TMCs for Express Highways and Arterial Highways

The set of express highways covered by the CMP extends beyond the MPO boundary to include all of area formerly known as the ‘model area’ of 164 towns. The list of express highway TMCs covered by the CMP is defined by intersecting the full Massachusetts TMC network with the former ‘model area’ geometry. From this, separate lists of TMCs for each route are extracted, so individual routes can easily be queried interactively in RITIS and/or using a vanilla database or the ESRI tools.

These lists are maintained on the lilliput file server in the folder:

\\\\lilliput\\groups\\Traffic\_and\_Design\\11123 CMP 2019 INRIX\\TMC\_lists

This folder contains two subfolders: **expressways** and **arterials.** Each of these, in turn, contains two subfolders: **raw** and **quoted**. 

-   The **raw** folder contains the list of TMCs for each route, *not* delimited by quotation marks, for use in RITIS.
-   The **quoted** folder contains the list of TMCs for each route delimited by quotes, for use when querying a vanilla database or in the ESRI tools.

### List of CMP Dates

The list of dates for which CMP performance metrics will be calculated for this year's CMP is obtained from CTPS's CMP Manager. 
These dates comprise the dates for roughly 60 Tuesdays, Wednesdays, and Thursdays in the spring and fall that do not fall on weeks that are deemed ‘special’, i.e., for which travel behavior may well vary from the norm. One such example week is school vacation weeks.

The dates for the 2019 CMP are:

-   March 5-7, 12-14, 19-21, 26-28
-   April 2-4. 9-11, 23-25, 30
-   May 1-2, 7-9, 14-16, 21-23
-   September 10-12, 17-19, 24-26
-   October 1-3, 8-10, 15-17, 22-24, 29-31
-   November 5-7, 12-14, 19-21

### AM and PM Peak Time Periods for Express Highways and Arterial Highways

The temporal bounds of the AM and PM peak periods for the CMP have not changed for more than 10 years. However, before setting out to begin calculating CMP performance metrics for any given years, check with the CTPS CMP manager to verify the bounds of the temporal periods to use.

As of 2023, the CMP peak periods for express highways are:

-   6:00 a.m. to 10:00 a.m.
-   3:00 p.m. to 7:00 p.m.

As of 2023, the CMP peak periods for arterial highways are:

-   6:30 a.m. to 9:30 a.m.
-   3:30 p.m. to 6:30 p.m.

### INRIX TMC - MassDOT Road Inventory Conflation

This is a very complex process, requiring a non-trivial amount of manual intervention after the application of an automated process. What follows is only a high-level overview of the process.

**TBD**

## _Outputs_

The end products of processing raw INRIX speed and travel-time data for the CMP are two database tables - one for express highways, one for arterial highways. Each table is indexed by TMC identifier; each row contains a unique TMC identifier, its associated geometry, and a list of attributes (“columns”). 

These attributes include:

-   Relatively “static” attributes that in general do not change from year-to-year, such as the identifier for the route of which the TMC is a part, and information harvested from the MassDOT Road Inventory
-   A variety of performance measures calculated for the TMC for the year in question

Processing generates a File Geodatabase Feature class in the working directory in which the analysis was performed. This feature class is then copied to CTPS's enterprise Geodatabase ‘mpodata’ (currently hosted on appsrvr3); this is the ‘official copy' of the results, for internal use. This feature class is also copied to a PostGIS database on CTPS's external MPO webserver, from which it is consumed by public-facing applications that present the data ('CMP performance dashboards').

The list of attributes for the two classes of road (expressway or arterial) vary slightly; each is presented in the following subsections.

### Express Highways

The schema for the output dataset for express highways is as follows:

|     |     |     |
| --- | --- | --- |
| **Column** | **Contents** | **Datatype** |
| tmc | Unique TMC identifier | text |
| rid | “CMP route” ID number (as text) | text |
| route\_num | MassDOT route number | text |
| road\_name | NULL for expressways (vestigial) | text |
| direction | Northbound, Southbound, etc. | text |
| seg\_begin | Description of beginning of TMC | text |
| seg\_end | Description of end of TMC | text |
| from\_meas | Starting M-value of TMC along route | double |
| to\_meas | Ending M-value of TMC along route | double |
| distance | to\_meas – from\_meas | double |
| spd\_limit | Speed limit, harvested from Road Inventory | integer |
| lanes | Number of lanes, harvested from Road Inventory | integer |
| lane\_miles | Lane-miles in TMC | double |
| community | MassGIS-style town name | text |
| ffs | Free-flow speed | double |
| am\_avg\_sp | Average speed in AM peak period | double |
| am\_spd\_ix | AM peak period speed index | double |
| am\_cong\_sp | Congested speed in AM peak period | double |
| am\_cong\_mn | Congested minutes in AM peak period | double |
| am\_avtt\_ix | AM peak period average travel time index | double |
| am\_del\_mi | AM peak period delay per mile | double |
| am\_5ptt\_ix | AM peak period 5% travel-time index | double |
| pm\_avg\_sp | Average speed in PM peak period | double |
| pm\_spd\_ix | PM peak period speed index | double |
| pm\_cong\_sp | Congested speed in PM peak period | double |
| pm\_cong\_mn | Congested minutes in PM peak period | double |
| pm\_avtt\_ix | PM peak period travel-time index | double |
| pm\_del\_mi | PM peak period delay per mile | double |
| pm\_5ptt\_ix | PM peak period 5% travel-time index | double |
| “shape” or “wkb\_geometry” | TMC geometry (spatial data) | geometry |

### Arterial Highways

The schema for the output dataset for arterial highways is as follows:

|     |     |     |
| --- | --- | --- |
| **Column** | **Contents** | **Datatype** |
| tmc | Unique TMC identifier | text |
| rid | “CMP route” ID number (as text) | text |
| road\_name | Road name, e.g., “Beacon St” | text |
| rte\_name | Name of route (may be MassDOT route\_id) | text |
| direction | Northbound, Southbound, etc. | text |
| seg\_begin | Description of beginning of TMC | text |
| seg\_end | Description of end of TMC | text |
| from\_meas | Starting M-value of TMC along route | double |
| to\_meas | Ending M-value of TMC along route | double |
| distance | to\_meas – from\_meas | double |
| spd\_limit | Speed limit, harvested from Road Inventory | integer |
| lanes | Number of lanes, harvested from Road Inventory | integer |
| lane\_miles | Lane-miles in TMC | double |
| community | MassGIS-style town name | text |
| ffs | Free-flow speed | double |
| am\_avg\_sp | Average speed in AM peak period | double |
| am\_spd\_ix | AM peak period speed index | double |
| am\_cong\_sp | Congested speed in AM peak period | double |
| am\_cong\_mn | Congested minutes in AM peak period | double |
| am\_avtt\_ix | AM peak period average travel time index | double |
| am\_del\_mi | AM peak period delay | double |
| am\_5ptt\_ix | AM peak period 5% travel-time index | double |
| am\_lottr | AM peak period level of travel time reliability | double |
| pm\_avg\_sp | Average speed in PM peak period | double |
| pm\_spd\_ix | PM peak period speed index | double |
| pm\_cong\_sp | Congested speed in PM peak period | double |
| pm\_cong\_mn | Congested minutes in PM peak period | double |
| pm\_avtt\_ix | PM peak period travel-time index | double |
| pm\_del\_mi | PM peak period delay | double |
| pm\_5ptt\_ix | PM peak period 5% travel-time index | double |
| pm\_lottr | PM peak period level of travel time reliability | double |
| “shape”  or “wkb\_geometry” | TMC geometry (spatial data) | geometry |

## _Methods_

### Methods - Historical Background

The overall methods for calculating the CMP performance metrics is summarized below in the section entitled **Methods - High-level Overview**. The details of the implementation of these methods are inexorably tied up with the history of CTPS's work with INRIX data for the CMP between 2014 and the present. The following is an summary of the most salient points of this historical context.

#### 2012 CMP

Before the 2012 CMP, CTPS obtained speed and travel-time data for the roadways monitored for the CMP by **direct, manual data collection.** In summary, this method entailed two staff members traveling in a car from one end of each CMP route to the other several times, capturing GPS points periodically and recording speed and elapsed time manually by pencil-and-paper. These data were subsequently transformed into digital form, with the GPS point-traces used to create GIS linework.

Needless to say, this method is labor intensive and readily subject to error. A better way to obtain speed and travel-time data became available when INRIX began to offer such a product commercially. At least at that time, INRIX's speed and travel time data was sold in temporal units of **calendar quarter** and spatial units of **county**. 

The Boston Region MPO area comprises all or part of the following seven counties:

-   Bristol county
-   Essex county
-   Middlesex county
-   Norfolk county
-   Plymouth county
-   Suffolk county
-   Worcester county

In 2014, CTPS obtained funding from the MPO to purchase speed and travel-time data for the four calendar quarters of 2012 for the seven counties listed above. The raw speed and travel-time data was delivered in 28 ZIP archives, one for each (calendar quarter, county) pair.

A shapefile for the TMCs (road segments) in Massachusetts was also shipped with the raw speed and travel-time data. This shapefile was found to be severely defective. In general, the spatial accuracy of the linework was poor, with many line segments 10 or more meters from their ‘ground truth’ location. Further complicating maters, the directionality of the linework was inconsistent: for example, one segment of a north-south route might be drawn north-south, whereas the next might be drawn south-north. No ‘rhyme or reason’ was found to account for this variability.

In short, the shapefile delivered by INRIX was found to be only partially usable. The *approximate* locations of TMCs in it could be used, but review and correction would be needed to establish their their exact location directionality. Consequently, it was necessary to conflate the INRIX TMCs with the MassDOT Road Inventory in order to establish the correct location and directionality of each TMC. This conflation was further required because the **posted speed limit** in effect in each TMC was required in order to calculate many performance metrics, and the **number of travel lanes** was required for other reporting purposes. These two attributes - posted speed limit and number of travel lanes could be harvested from the Road Inventory. 

Although some parts of a conflation process can be performed by automation, inevitably the output of an automated process requires manual review and cleanup. Because of the very poor spatial quality of the 2012 INRIX TMC shapefile, manual review and editing/correction required approximately **1 person-year**: four GIS temps working for approximately 3 calendar months.

Because of the very poor spatial accuracy of the INRIX TMC network, the decision was made to use the Road Inventory geometry as **the** geometry for use in the calculation and reporting of CMP performance metrics. One consequence of this is that the length of each TMC in the INRIX shapefile could not be used as a reliable indication of TMC segment length. The approach taken was to calculate the length of each TMC ourselves using the **distance = rate \* time** method, taking the average of this over all records for each TMC in the input data.

##### Processing in Google BigQuery and in MS Access

The volume of raw speed and travel-time data delivered by INRIX (around three-quarters of a TeraByte), made it impossible to use a conventional database, such as PostgreSQL, to process this data. In 2012, Google introduced BigQuery a cloud-hosted, massively parallel database suitable for handling PetaByte workloads. BigQuery was ideally suited to the task at hand, and we proceeded to process the CMP data using BigQuery. Part-way through this process, it became clear that the data had been filtered down to a size that could be handled by a conventional database. Because of the background of the people working on the project at the time, MS Access was used to perform the final processing of the 2012 CMP data.

A collection of Windows batch scripts (.bat files) which call the Google Cloud command-line interface for BigQuery were created to load the data into BigQuery and process it there. Once this processing was completed, the results were downloaded and loaded into an MS Access database for final processing. Unfortunately, as of May 2023, the neither the batch scripts nor the ‘working’ MS Access database for the 2012 CMP could be found on the lilliput file server.

#### 2015 CMP

In 2016, CTPS obtained funding from the MPO to purchase speed and travel-time data for the four calendar quarters of 2015 for the seven counties listed in the previous section. The raw speed and travel-time data was delivered in 28 ZIP archives, one for each (calendar quarter, county) pair as was the case for the 2012 data.

As was the case for the 2012 data, a shapefile for the TMCs (road segments) in Massachusetts was also shipped with the raw speed and travel-time data. This was a new, and different, shapefile. The TMC network can, and does change from year to year. Changes to the TMCs for expressways are relatively infrequent, and when they do occur, minor. Changes to the TMC network for arterials are more significant from one year to the next. Over the course of three years (from 2012 to 2015), changes can be quite substantial. This required us to re-conflate the TMC network with the Road Inventory - which itself had changed in the intervening three years.

The spatial accuracy of the INRIX TMC shapefile for 2015 was substantially better than it was in 2012. In addition, nearly all errors in the directionality of TMCs had been corrected. Nonetheless, we continued to use the Road Inventory geometry as **the** geometry to use for analysis and display of CMP performance data. This meant that we continued to calculate the length of each TMC ourselves rather than relying on its length as given in the TMC shapefile.

The partitioning of processing between Google BIgQuery and MS Access for the 2015 CMP was the same as for the 2012 CMP. For the 2015 CMP, however, the assets used to perform the analysis (Windows scripts to drive BigQuery and MS Access database) have been preserved.

##### Processing in Google BigQuery

The Windows scripts driving Google BigQuery queries for the calculation of the 2015 CMP performance metrics are found in: \\\\lilliput\\groups\\Traffic\_and\_Design\\11123 CMP 2015 INRIX\\batch\_files, which contains a number of sub-folders for each ‘stage’ of processing. The following naming convention was adopted for these scripts. Each script was given a name of the form:

-   n - a number indicating the major processing stage
-   an underscore
-   m - a number indicating the step within the given stage
-   a descriptive name

Thus, for example, the SQL script **1\_2\_load\_inrix\_2012\_quarter1\_suffolk.bat** was the second step in the first stage of processing, and is concerned with the data for Suffolk county for calendar quarter 1 of 2012.

The stages and steps are as follows:

-   Stage 1 - process data for 1st calendar quarter
    -   Step 1 - create table
    -   Step 2 - load data into table
    -   Step 3 - prune TMCs to delete
    -   Step 4 - merge data for all counties
    -   Step 5 - select TMCs to include in this year's CMP
    -   Step 6 - select records with confidence\_score = 30 and cvalue >= 75
-   Stage 2 - ditto, for the 2nd calendar quarter
-   Stage 3 - ditto, for the 3rd calendar quarter
-   Stage 4 - ditto, for the 4th calendar quarter
-   Stage 5 - merge data for all 4 quarters into individual tables
    -   Step 1 - create table
    -   Step 2 - filter records for use in CMP
    -   Step 3 - filter records with valid cvalues
    -   Step 4 - filter records for expressway TMCs
    -   Step 5 - filter records for arterial TMCs
    -   Step 6 - filter records with valid cvalues for expressway TMCs
    -   Step 7 - filter records with valid cvalues for arterial TMCs
    -   Step 8 - filter records for CMP dates for expressway TMCs
    -   Step 9 - filter records for CMP dates for arterial TMCs
    -   Step 10 - filter records with valid cvalues and CMP dates for expressway TMCs
    -   Step 11 - filter records with valid cvalues and CMP dates for arterial TMCs
    -   Step 12 - filter records for CMP dates in the AM peak period for expressway TMCs
    -   Step 13 - filter records for CMP dates in the PM peak period for expressway TMCs
    -   Step 14 - filter records for CMP dates in the AM peak period for arterial TMCs
    -   Step 15 - filter records for CMP dates in the PM peak period for arterial TMCs
    -   Step 16 - filter records with valid cvalues for CMP dates in the AM peak period for expressway TMCs
    -   Step 17 - filter records with valid cvalues for CMP dates in the PM peak period for expressway TMCs
    -   Step 18 - filter records with valid cvalues for CMP dates in the AM peak period for arterial TMCs
    -   Step 19 - filter records with valid cvalues for CMP dates in the PM peak period for arterial TMCs
    -   Step 20 - create count of all records for each TMC
    -   Step 21 - create count of all records with valid cvalues for each TMC
    -   Step 22 - create count of all records for all expressway and arterial TMCs
    -   Step 23 - create count of all records with valid cvalues for all expressway and arterial TMCs
    -   Step 24 - create count of all records for all expressway TMCs
    -   Step 25 - create count of all records with valid cvalues for all expressway TMCs
    -   Step 26 - calculate sum\_distance and sum\_time for all arterial TMCs for CMP dates
    -   Step 27 - ditto, but for AM peak period
    -   Step 28 - ditto, for PM peak period
    -   Step 29 - calculate sum\_distance and sum\_time for all arterial TMCs from records with valid cvalues on CMP dates
    -   Step 30 - ditto, for AM peak period
    -   Step 31 - ditto, for PM peak period
    -   Step 32 - calculate sum\_distance and sum\_time for all expressway TMCs for CMP dates
    -   Step 33 - ditto, for AM peak period
    -   Step 34 - ditto, for PM peak period
    -   Step 35 - calculate sum\_distance and sum\_time for all expressway TMCs from records with valid cvalues on CMP dates
    -   Step 36 - ditto, for AM peak period
    -   Step 37 - ditto, for PM peak period
    -   Step 38 - get speed for all arterial TMCs from records with valid values on CMP dates
    -   Step 39 - ditto, for the AM peak period
    -   Step 40 - ditto, for the PM peak period
    -   Step 41 - get speed for all expressway TMCs from records with valid values on CMP dates
    -   Step 42 - ditto, for the AM peak period
    -   Step 43 - ditto, for the PM peak period
    -   Step 44 - calculate average speed and standard deviation of speed for all arterial TMCs from records with valid cvalues on CMP dates
    -   Step 45 - ditto, for the AM peak period
    -   Step 46 - ditto, for the PM peak period
    -   Step 47 - calculate average speed and standard deviation of speed for all expressway TMCs from records with valid cvalues on CMP dates
    -   Step 48 - ditto, for the AM peak period
    -   Step 49 - ditto, for the PM peak period
-   Stage 6 - calculate CMP performance metrics - 32 steps, organized in 7 groups
    -   Group 1
        -   Step 1 - create table for calculating performance metrics for expressway TMCs, AM and PM peak periods
        -   Step 2 - create table for calculating performance metrics for arterial TMCs, AM and PM peak periods
        -   Step 3 - calculate average length for all expressway TMCs
        -   Step 4 - calculate average length for all arterial TMCs
    -   Group 2
        -   Step 1 - create count of total records to be used for calculating AM peak period performance metrics for all expressway TMCs
        -   Step 2 - create count of total records to be used for calculating PM peak period performance metrics for all expressway TMCs
        -   Step 3 - create count of total records to be used for calculating AM peak period performance metrics for all arterial TMCs
        -   Step 4 - create count of total records to be used for calculating PM peak period performance metrics for all arterial TMCs
    -   Group 3
        -   Step 1 - create count of total number of ‘congested’ records for all expressway TMCs during the AM peak period
        -   Step 2 - create count of total number of ‘congested’ records for all expressway TMCs during the PM peak period
        -   Step 3 - create count of total number of ‘congested’ records for all arterial TMCs during the AM peak period
        -   Step 4 - create count of total number of ‘congested’ records for all arterial TMCs during the PM peak period
    -   Group 4
        -   Step 1 - create sum\_speed record for for all expressway TMCs during the AM peak period
        -   Step 2 - create sum\_speed record for for all expressway TMCs during the PM peak period
        -   Step 3 - create sum\_speed record for for all arterial TMCs during the AM peak period
        -   Step 4 - create sum\_speed record for for all arterial TMCs during the PM peak period
    -   Group 5
        -   Step 1 - create sum\_speed record from ‘congested’ records for all expressway TMCs during the AM peak period
        -   Step 2 - create sum\_speed record from ‘congested’ records for all expressway TMCs during the PM peak period
        -   Step 3 - create sum\_speed record from ‘congested’ records for all arterial TMCs during the AM peak period
        -   Step 4 - create sum\_speed record from ‘congested’ records for all arterial TMCs during the PM peak period
    -   Group 6
        -   Step 1 - calculate 5th percentile speed for all expressway TMCs in the AM peak period
        -   Step 2 - calculate 5th percentile speed for all expressway TMCs in the PM peak period
        -   Step 3 - calculate 5th percentile speed for all arterial TMCs in the AM peak period
        -   Step 4 - calculate 5th percentile speed for all arterial TMCs in the PM peak period
    -   Group 7
        -   Step 1 - calculate 85th percentile speed for all expressway TMCs between midnight and 2 AM
        -   Step 2 - calculate 85th percentile speed for all arterial TMCs between midnight and 2 AM
        -   Step 3 - calculate 85th percentile speed for all expressway TMCs between 2AM and 4 AM
        -   Step 4 - calculate 85th percentile speed for all arterial TMCs between 2AM and 4 AM
        -   Step 5 - calculate 85th percentile speed for all expressway TMCs between midnight and 4 AM
        -   Step 6 - calculate 85th percentile speed for all arterial TMCs between midnight and 4 AM
        -   Step 7 - calculate 85th percentile speed for all expressway TMCs between midnight and 4 AM; alternative calculation using records with cvalues ≥ 50
        -   Step 8 -  calculate 85th percentile speed for all arterial TMCs between midnight and 4 AM; alternative calculation using records with cvalues ≥ 50

##### Processing in MS Access

The MS Access database in which the final calculation of performance metrics was performed is found in: \\\\lilliput\\groups\\Traffic\_and\_Design\\11123 CMP 2015 INRIX\\performance\_measures\\inrix\_2015\_perf\_meas.mdb. This database contains the downloaded data, the queries to process it, and the tables containing the final results.

**TO BE CONTINUED**

#### 2019 CMP

##### Access to INRIX Data via RITIS

n 2019 MassDOT, as part of the Eastern Transportation Coalition (formerly the I-95 Corridor Coalition) signed a multi-year contract with INIRX and the [Center for Advanced Transportation Technology (CATT) Laboratory](https://www.cattlab.umd.edu/) ('CATT Lab) at the University of Maryland for the CATT Lab to house INRIX data and to develop client software providing access to it. The MassDOT provided for all departments in MassDOT and all MPOs in the state to have access to RITS. Initially, RITIS would be populated with data for the previous few years, but over time it would be populated with INRIX data going back to 2015. (This is now the case.)

Given this, it was no longer necessary for CTPS to purchase speed and travel-time data directly from INRIX. Moreover, since RITIS client software (the ‘Massive Data Downloader') would provide the ability to query for specific TMCs, specific dates and specific times, CTPS's processing of this data would be much simpler than in the past. Essentially, we could start with (almost) the output of **Stage 5**, above. (The only hitch, as mentioned elsewhere, is that the RITIS GUI does not support filtering by the value of the **cvalue** field.) After downloading data from RITIS, the first part of CTPS's processing would be limited to filtering out records with poor **cvalue**s and ‘exploding’ the INRIX **measurment\_tstamp** field.

##### 2019 INRIX TMC Shapefile

As noted elsewhere, the shapefile for the full INRIX TMC network (as opposed to one for the NPMRDS subset) is not available from RITIS. As was the case in the past, it was obtained directly from INRIX. 

##### 2019 CMP for Expressways

Examination the INRIX TMC shapefile for 2019 showed that there had been no changes to the TMC network for CMP express highways between 2015 and 2019. Consequently, the results of the 2015 conflation of the INRIX TMC network and the Road Inventory could be used as-is for expressway TMCs. Substantial changes were noted in the arterial TMCs; these would need to be re-conflated.

Work on 2019 CMP for Expressways was begun in January 2020 and was completed in March 2020, days before the onset of the Covid-19 “lockdown.” As noted above, after download of data from RITIS, work essentially began with what would have been the results of Stage 5, above - modulo having to filter records on the cvalue field and ‘explode’ the INRIX measurement\_tstamp. The result was uploaded to Google BigQuery. Processing proceeded in BigQuery as described above under **Stage 5**. The results were then downloaded from BigQuery and loaded into MS Access. Processing was completed as described above under **Processing Performed in MS Access** for the 2015 Expressway CMP.

##### 2019 CMP for Arterials

Work on the 2019 CMP for arterials was interrupted by the onset of the Covid-19 pandemic. It became clear from even a cursory examination of the 2019 INRIX TMC shapefile that the conflation results from 2015 could not be re-used for the arterials: there were a large number of changes either to the TMC network or to the Road Inventory between 2015 and 2019. It was clear that the conflation would have to be re-done. A discussion of this conflation is found below under **Conflate Road Inventory with INRIX TMC Network.**

The following section describes the calculation of performance metrics for arterials for the 2019 CMP.

### Methods - High-level Overview

In high-level terms, the process is as follows:

1.  Get set up to use Google cloud tools
2.  Conflate Road Inventory with INRIX TMC network
3.  Obtain INRIX speed and travel time data
4.  ‘Explode’ INRIX timestamp field
5.  Upload data to Google Cloud Storage and make it available to Google BigQuery
6.  Conflate Road Inventory with INRIX TMC network
7.  Calculate performance metrics for each TMC - this step is divided between processing done in Google BigQuery and processing done in a ‘desktop’ database
8.  Produce final data product by joining the tabular performance data with the corresponding spatial data

Step 2 (conflation) can be performed in parallel with Steps 1, 3, and 4. It must be completed before starting Step 5.

A detailed description of each step follows.

### Getting Set Up to Use Google Cloud Tools

Many of the steps described in this document are performed by executing Google BigQuery queries submitted from the Google Cloud SDK (software development kit) for Windows. The Google Cloud SDK for Windows must first be installed on your PC and then initialized. See [https://cloud.google.com/sdk/](https://cloud.google.com/sdk/) for more information on downloading and installing the SDK.

After the SDK has been installed, open a Windows Command Prompt (“DOS Box”) and enter the following command to initialize it:

gcloud init

The completion of this step will involve your logging into (“authenticating”) to your Google account in a page that will be opened in your web browser. Part of this authentication step will involve selecting the cloud project to “use” during your *gcloud* session: the project to select is **ctps-trafic-1**.

### Conflate Road Inventory with INRIX TMC Network

Before work calculating CMP performance metrics can begin, the “linework” for the MassDOT Road Inventory must be conflated with the “linework” from INRIX TMC shapefile. (In practice, the INRIX TMC shapefile is loaded into an ESRI Feature Class for conflation and other processing.) This is done so that two attributes from the Road Inventory (**speed limi**t and **number of travel lanes**) may be harvested from the Road Inventory and associated with the relevant INRIX TMCs.

The conflation process is complex and, in particular for arterials, subtle. Although a ‘first draft’ conflation can be performed by automated tools, the output of automated conflation requires substantial ‘editorial’ review and review and correction before it is usable.

Because the geometry of expressways doesn't change much from year to year, it has been possible to re-use the result of the conflation of the expressway TMCs performed for the 2015 CMP for the 2019 CMP. The results of this conflation are found in the table **Inrix\_2019\_cmp\_exp\_roadinv\_attrib** in the MS Access database

\\\\lilliput\\groups\\Traffic\_and\_Design\\11123 CMP 2019 INRIX\\performance\_measures\\inrix\_2019\_perf\_meas.mdb

The result of conflating the TMCs for the CMP arterial routes from the 2019 INRIX TMC shapefile and the 2019 Road Inventory are found in:

**TBD**

The schema of both these conflation-result tables is:

|     |     |     |
| --- | --- | --- |
| **Column** | **Contents** | **Data Type** |
| tmc | TMC ID string | string |
| tmctype | The type of TMC Code. “P1” is the typical TMC Code. “P3” indicates national, state, and county boundaries, rest areas, toll plazas, major bridges, etc. “P4” is for ramps.  **Harvested from TMC shapefile.** | string |
| route\_id | MassDOT Route ID (see below). **Harvested from Road Inventory.** | string |
| route\_num | MassDOT Route Number. **Harvested from Road Inventory.** | string |
| direction | MassDOT Route Direction (not abbreviated). **Harvested from INRIX TMC shapefile.** | string |
| firstnm | The cross street and/or interchange associated with the internal segment of the 5-digit location ID part of the TMC ID. **Harvested from  from INRIX TMC shapefile.** | string |
| from\_meas | Displacement of TMC start location, in miles, from beginning of Mass DOT Route ID. **Calculated by conflation process.** | floating point |
| to\_meas | Displacement of TMC end location, in miles, from beginning of Mass DOT Route ID. **Calculated by conflation process.** | floating point |
| speed\_limit | Speed Limit. **Harvested from Road Inventory.** | integer |
| num\_lanes | Number of Travel Lanes. **Harvested from Road Inventory**. | integer |
| community | Comma-separated list of one or more towns through which the TMC passes. **Produced by conflation process.** | text |
| rid | CMP route ID (as text). **Produced by conflation process.** | text |
| seg\_begin | Description of TMC beginning location. **Produced by conflation process.** | text |
| seg\_end | Description of TMC end location. **Produced by conflation process.** | text |
| road\_name | The local name of the road, for road segments that have either no road number or both a name and number. (NULL for expressways.) **Harvested from INRIX TMC shapefile.** | text |

### Obtain INRIX Speed and Travel Time Data

The URL for the RITIS Probe Data Analytics (PDA) suite is: [https://pda.ritis.org/suite/download/](https://pda.ritis.org/suite/download/)

Submit queries on this page for (1) the data for express highways, and (2) the data for arterial highways. 
Queries must be specified interactively using the RITIS GUI. \(There is a very primitive programmatic API for the RITIS Massive Data Downloader - see
 the commented code in [this GitHub repository](https://github.com/CTPSSTAFF/ritis-api-test) for more information - but it is unusable for practical purposes.\)

The necessary parameters for each query are:

-   A list of TMCs
-   A list of dates
-   A list of time periods
-   Other parameters

#### **TMC Lists**

The list of all TMCs for the CMP express highways (within the former “CTPS model region”) is found in the file: 

\\\\lilliput\\groups\\Traffic*and*Design\\11123 CMP 2019 INRIX\\TMC\_lists\\raw\\all\_expressways\_in\_model\_region.txt

A comprehensive list of all TMCs for the CMP arterial highways within the MPO is found in the file:

\\\\lilliput\\groups\\Traffic*and*Design\\11123 CMP 2019 INRIX\\arterial\_conflation\\TMC\_Lists\\raw\\all-tmcs-2022-09-30.txt

#### **Dates and Times**

See discussion above, under Inputs.

#### **Other Parameters**

The other parameters to specify are:

-   (1) Select roads using segments from INRIX, specifying segment (TMC) codes
-   (2) Select days of week: Tuesday, Wednesday, Thursday
-   (3) Select data sources and measures:
    -   Data source: INRIX
    -   Speed
    -   Historic average speed
    -   Reference speed
    -   Travel Time
    -   C-value
    -   Confidence score:
        -   Check 30
        -   Un-check 20
        -   Un-check 10
-   (4) Select units for travel time: minutes
-   (5) Do not include null records
-   (6) Select averaging: Don’t average
-   (7) Provide title (for download file) – your choice
-   (8) Notification – Send email when export is ready to download

### ‘Explode’ INRIX Timestamp Field

The INRIX measurement\_tstamp field is very awkward to work with. For example, it makes it difficult for anyone not fluent in the use of SQL string and sub-string operators to query the data for specific days of the week, ranges of dates, ranges of time periods, etc. In order to ameliorate this, the INRIX measurement\_tstamp field is programmatically ‘exploded’ into the following field*s* before the data is loaded into a database:

|     |     |     |
| --- | --- | --- |
| month | integer | month on which sample was taken |
| day | integer | day of month on which sample was taken |
| dow | integer | day-of-the-week on which sample was taken |
| hour | integer | hour at which sample was taken |
| minute | integer | minute at which sample was taken |

The relevant code is stored on the lilliput file server:

\\\\lilliput\\groups\\Traffic*and*Design\\11123 CMP 2019 INRIX\\\\python\_scripts\\filter\_downloaded\_csv.py 

This script also filters out records with a **cvalue** field value less than 75.0 and records with empty/null **speed** or **cvalue** fields. 

### Upload data to Google Cloud Storage and Google BigQuery

**Step 1**

Using the Google Cloud Storage web console, created a new bucket in the ctps-traffic-1 project named ‘inrix\_cmp\_yyyy’, where yyyy is the relevant year.

**Step 2**

Using the Google Cloud Storage web console, upload the CSV file(s) produced in Step (1) to the inrix\_cmp\_yyyy bucket to a file named either (as appropriate):

-   cmp\_yyyy\_expressways\_all\_processed\_v1.csv
-   cmp\_yyyy\_arterials\_all\_processed\_v1.csv

### Calculate Performance Metrics for Each TMC

Because of the quantity of data to be processed (particularly when working with the unfiltered datasets purchased directly from INRIX before the advent of RITIS), processing is performed in two stages:

-   Stage 1 - processing performed in Google BigQuery
-   Stage 2 - processing performed in a ‘desktop’ database at CTPS

Between Steps 1 and 2 the result of processing data in BigQuery must be downloaded and loaded into a ‘desktop’ database.

**MS Access** was used as the ‘desktop’ database for the 2012 Expressway and Arterial CMPs, the 2015 Expressway and Arterial CMPs, and the 2019 Expressway CMP. Work on  the 2019 Arterial CMP was interrupted in March 2019 by the onset of the COVID-19 pandemic. At that point, conflation of the 2019 arterial TMCs with the Road Inventory had not taken place. This work resumed in the summer of 2022. At that point, it had become abundantly clear that MS Access was getting very long in the tooth, and migration to a more up-to-date platform was advisable. We chose to migrate from MS Access to **SQLite** a more modern desktop database. This allowed us to leverage the SQL queries used in the past, albeit with a non-trivial amount of massaging to account for differences in the dialect of SQL supported by the two databases. This document is written with the assumption that SQLite will continue to be used going forward.

The process of calculating CMP performance metrics varies slightly - but significantly - between expressways and arterials. Each is discussed separately.

### Calculate Performance Metrics for Each TMC - Expressways

### Calculate Performance Metrics for Each TMC - Arterials

This section documents the calculation of CMP arterial highway performance metrics for 2019. The queries named below are stored in Google BigQuery in the “folder” **ctps-traffic-1 > Saved queries > Project queries**. The names of all these queries end in ‘\_v2’, which reflects the fact that the list of TMCs to use in the analysis was updated part way through the process in 2019.

All these queries have hard-wired input and output table names (containing the string ‘2019’). These will have to be either edited for use in future years, or the script driving the execution of each will need to be parameterized to take ‘year’ as an input parameter.

As noted above, the calculation of CMP arterial performance measures proceeds in two stages.

 **Stage 1**, executed in Google BigQuery, is as follows:

1.  Combine AM and PM tables into a single table: run the query **6\_1\_00a\_create\_Inrix\_2019\_cmp\_art\_ampm\_v2**
2.  Calculate the average length of each TMC from the INRIX data: run the query **6\_1\_01\_create\_Inrix\_2019\_cmp\_art\_avg\_length\_v2** (See note below on this step.)
3.  Calculate the record count for each TMC: run the queries: **6\_2\_03\_create\_Inrix\_2019\_cmp\_art\_count\_all\_tmc\_am\_v**2 and **6\_2\_04\_create\_Inrix\_2019\_cmp\_art\_count\_all\_tmc\_pm\_v2**
4.  Calculate the number of congested records (speed < 19 mph) for each TMC: run the queries **6\_3\_04\_create\_inrix\_2019\_cmp\_art\_count\_cong\_tmc\_am\_v2** and **6\_3\_04\_create\_inrix\_2019\_cmp\_art\_count\_cong\_tmc\_pm\_v2**
5.  Calculate the sum of all speed values for each TMC: run the queries **6\_4\_03\_create\_inrix\_2019\_cmp\_art\_sum\_speed\_all\_tmc\_am\_v2** and **6\_4\_04\_create\_inrix\_2019\_cmp\_art\_sum\_speed\_all\_tmc\_pm\_v2**
6.  Calculate the sum of all speed values for congested records, i.e., records for which the speed is < 19 mph for each TMC: run the queries **6\_5\_03\_create\_inrix\_2019\_cmp\_art\_sum\_speed\_cong\_tmc\_am\_v2** and **6\_5\_04\_create\_inrix\_2019\_cmp\_art\_sum\_speed\_cong\_tmc\_pm\_v2**
7.  Calculate 5th percentile of speed distribution for each TMC: run the queries **6\_6\_03\_create\_inrix\_2019\_cmp\_art\_5pct\_speed\_tmc\_am\_v2** and **6**\_**6\_04\_create\_inrix\_2019\_cmp\_art\_5pct\_speed\_tmc\_pm\_v2**
8.  Calculate 50th and 80th percentile travel times for each TMC: run the queries **6\_20\_01\_cr\_Inrix\_2019\_cmp\_art\_80pct\_tt\_tmc\_am\_v2**, **6\_20\_02\_cr\_Inrix\_2019\_cmp\_art\_80pct\_tt\_tmc\_pm\_v2**, **6\_20\_03\_cr\_Inrix\_2019\_cmp\_art\_50pct\_tt\_tmc\_am\_v2** and **6\_20\_04\_cr\_Inrix\_2019\_cmp\_art\_50pct\_tt\_tmc\_pm\_v2**
9.  Estimate free-flow speeds for each TMC: run the query **6\_7\_09b\_create\_Inrix\_2019\_cmp\_art\_free\_flow\_speed\_v2\_incomplete**. **NOTE**: there were no records during the free flow period (12-5AM) for a number of the TMCs in the arterial TMC list. To remedy this, missing free-flow speeds were estimated as the posted speed limit. After downloading the table **INRIX\_2019\_cmp\_art\_free\_flow\_speed\_v2\_incomplete**, missing values were filled in in ArcGIS Pro by harvesting the posted speed limit from the Road Inventory.

At this point, the following tables were downloaded in CSV format from Google BigQuery and loaded into SQLite for further processing:

1.  INRIX\_2019\_cmp\_art\_avg\_length\_v2
2.  INRIX\_2019\_cmp\_art\_count\_all\_tmc\_am\_v2
3.  INRIX\_2019\_cmp\_art\_count\_all\_tmc\_pm\_v2
4.  INRIX\_2019\_cmp\_art\_count\_cong\_tmc\_am\_v2
5.  INRIX\_2019\_cmp\_art\_count\_cong\_tmc\_pm\_v2
6.  INRIX\_2019\_cmp\_art\_sum\_speed\_all\_tmc\_am\_v2
7.  INRIX\_2019\_cmp\_art\_sum\_speed\_all\_tmc\_pm\_v2
8.  INRIX\_2019\_cmp\_art\_sum\_speed\_cong\_tmc\_am\_v2
9.  INRIX\_2019\_cmp\_art\_sum\_speed\_cong\_tmc\_pm\_v2
10.  INRIX\_2019\_cmp\_art\_5pct\_speed\_tmc\_am\_v2
11.  INRIX\_2019\_cmp\_art\_5pct\_speed\_tmc\_pm\_v2
12.  INRIX\_2019\_cmp\_art\_free\_flow\_speed\_v2\_incomplete
13.  INRIX\_2019\_cmp\_art\_80pct\_tt\_am\_tmc\_v2
14.  INRIX\_2019\_cmp\_art\_80pct\_tt\_pm\_tmc\_v2
15.  INRIX\_2019\_cmp\_art\_50pct\_tt\_am\_tmc\_v2
16.  INRIX\_2019\_cmp\_art\_50pct\_tt\_pm\_tmc\_v2

**Stage 2**, executed in SQLite, is as follows:

### Produce Final Data Product

## _Point of Contact_

[Ben Krepp](mailto://bkrepp@ctps.org), [Seth Sturmwasser](mailto://ssturmwasser@ctps.org), [David Knudsen](mailto://dknudsen@ctps.org), [Ryan Hicks](mailto://rhicks@ctps.org)

## _Date Updated_

October 18, 2022; edited, updated, and converted to this format April 28-May 1-2, 2023.