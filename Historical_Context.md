# Historical Context
## Introduction
The overall methods for calculating the CMP performance metrics is summarized in the __Calculation\_of\_Metrics chapter.
The details of the implementation of these methods are inexorably tied up with the history of CTPS's work with INRIX data for the CMP between 2014 and the present. 
The following is an summary of the most salient points of this historical context.

### 2012 CMP
Before the 2012 CMP, CTPS obtained speed and travel-time data for the roadways monitored for the CMP by _direct, manual data collection._ 
In summary, this method entailed two staff members traveling in a car from one end of each CMP route to the other several times, 
capturing GPS points periodically and recording speed and elapsed time manually by pencil-and-paper. 
These data were subsequently transformed into digital form, with the GPS point-traces used to create GIS linework.

Needless to say, this method is labor intensive and readily subject to error. 
A better way to obtain speed and travel-time data became available when INRIX began to offer such a product commercially. 
At least at that time, INRIX's speed and travel time data was sold in temporal units of __calendar quarter__ and spatial units of __county__. 

The Boston Region MPO area comprises all or part of the following seven counties:
-   Bristol county
-   Essex county
-   Middlesex county
-   Norfolk county
-   Plymouth county
-   Suffolk county
-   Worcester county

In 2014, CTPS obtained funding from the MPO to purchase speed and travel-time data for the four calendar quarters of 2012 for the seven counties listed above.
The raw speed and travel-time data was delivered in 28 ZIP archives, one for each (calendar quarter, county) pair.

A shapefile for the TMCs (road segments) in Massachusetts was also shipped with the raw speed and travel-time data. 
This shapefile was found to be severely defective. In general, the spatial accuracy of the linework was poor, with many line segments 10 or more meters from their ‘ground truth’ location. 
Further complicating maters, the directionality of the linework was inconsistent: for example, one segment of a north-south route might be drawn north-south, whereas the next might be drawn south-north. No ‘rhyme or reason’ was found to account for this variability.

In short, the shapefile delivered by INRIX was found to be only partially usable. The _approximate_ locations of TMCs in it could be used, 
ut review and correction would be needed to establish their their exact location directionality. 
Consequently, it was necessary to conflate the INRIX TMCs with the MassDOT Road Inventory in order to establish the correct location and directionality of each TMC. 
This conflation was further required because the __posted speed limit__ in effect in each TMC was required in order to calculate many performance metrics, 
and the __number of travel lanes__ was required for other reporting purposes. 
These two attributes - posted speed limit and number of travel lanes could be harvested from the Road Inventory. 

Although some parts of a conflation process can be performed by automation, inevitably the output of an automated process requires manual review and cleanup. 
Because of the very poor spatial quality of the 2012 INRIX TMC shapefile, manual review and editing/correction required 
approximately __1 person-year_: four GIS temps working for approximately 3 calendar months.

Because of the very poor spatial accuracy of the INRIX TMC network, the decision was made to use the Road Inventory geometry as __the__ geometry for use 
in the calculation and reporting of CMP performance metrics. One consequence of this is that the length of each TMC in the INRIX shapefile could not be used as a reliable indication of TMC segment length. The approach taken was to calculate the length of each TMC ourselves using the **distance = rate \* time** method, taking the average of this over all records for each TMC in the input data.

#### Processing in Google BigQuery and in MS Access
The volume of raw speed and travel-time data delivered by INRIX (around three-quarters of a TeraByte), made it impossible to use a conventional database, such as PostgreSQL, 
to process this data. In 2012, Google introduced BigQuery a cloud-hosted, massively parallel database suitable for handling PetaByte workloads. 
BigQuery was ideally suited to the task at hand, and we proceeded to process the CMP data using BigQuery. 
Part-way through this process, it became clear that the data had been filtered down to a size that could be handled by a conventional database. 
Because of the background of the people working on the project at the time, MS Access was used to perform the final processing of the 2012 CMP data.

A collection of Windows batch scripts (.bat files) which call the Google Cloud command-line interface for BigQuery were created to load the data into BigQuery and process it there. 
Once this processing was completed, the results were downloaded and loaded into an MS Access database for final processing. Unfortunately, as of May 2023, the neither the batch scripts nor the ‘working’ MS Access database for the 2012 CMP could be found on the lilliput file server.

### 2015 CMP
In 2016, CTPS obtained funding from the MPO to purchase speed and travel-time data for the four calendar quarters of 2015 for the seven counties listed in the previous section. 
The raw speed and travel-time data was delivered in 28 ZIP archives, one for each (calendar quarter, county) pair as was the case for the 2012 data.

As was the case for the 2012 data, a shapefile for the TMCs (road segments) in Massachusetts was also shipped with the raw speed and travel-time data. 
This was a new, and different, shapefile. T
he TMC network can, and does change from year to year. Changes to the TMCs for expressways are relatively infrequent, and when they do occur, minor. 
Changes to the TMC network for arterials are more significant from one year to the next. 
Over the course of three years (from 2012 to 2015), changes can be quite substantial. This required us to re-conflate the 
TMC network with the Road Inventory - which itself had changed in the intervening three years.

The spatial accuracy of the INRIX TMC shapefile for 2015 was substantially better than it was in 2012. 
In addition, nearly all errors in the directionality of TMCs had been corrected. 
Nonetheless, we continued to use the Road Inventory geometry as __the__ geometry to use for analysis and display of CMP performance data. 
This meant that we continued to calculate the length of each TMC ourselves rather than relying on its length as given in the TMC shapefile.

The partitioning of processing between Google BIgQuery and MS Access for the 2015 CMP was the same as for the 2012 CMP. 
For the 2015 CMP, however, the assets used to perform the analysis (Windows scripts to drive BigQuery and MS Access database) have been preserved.

### 2019 CMP
#### Access to INRIX Data via RITIS
In 2019 MassDOT, as part of the Eastern Transportation Coalition (formerly the I-95 Corridor Coalition) signed a multi-year contract with 
INIRX and the [Center for Advanced Transportation Technology (CATT) Laboratory](https://www.cattlab.umd.edu/) ('CATT Lab) at the University of Maryland for 
the CATT Lab to house INRIX data and to develop client software providing access to it. 
The MassDOT provided for all departments in MassDOT and all MPOs in the state to have access to RITS. 
Initially, RITIS would be populated with data for the previous few years, but over time it would be populated with INRIX data going back to 2015. \(This is now the case.\)

Given this, it was no longer necessary for CTPS to purchase speed and travel-time data directly from INRIX. 
Moreover, since RITIS client software (the ‘Massive Data Downloader') would provide the ability to query for specific TMCs, specific dates and specific times, 
CTPS's processing of this data would be much simpler than in the past. 

Essentially, we could start with (almost) the output of **Stage 5**, above. (The only hitch, as mentioned elsewhere, is that the RITIS GUI does not support filtering by the value of the **cvalue** field.) After downloading data from RITIS, the first part of CTPS's processing would be limited to filtering out records with poor **cvalue**s and ‘exploding’ the INRIX **measurment\_tstamp** field.

#### 2019 INRIX TMC Shapefile
As noted elsewhere, the shapefile for the full INRIX TMC network (as opposed to one for the NPMRDS subset) is not available from RITIS. 
As was the case in the past, it was obtained directly from INRIX. 

Examination the INRIX TMC shapefile for 2019 showed that there had been no changes to the TMC network for CMP express highways between 2015 and 2019. 
Consequently, the results of the 2015 conflation of the INRIX TMC network and the Road Inventory could be used as-is for expressway TMCs. 
Work on 2019 CMP for Expressways was begun in January 2020 
As was the case for the 2012 and 2015 expressway CMPs, processing was divided between Google BigQuery and MS Access.
Because RITIS eliminated the need to upload an entire year's worth of data to Google BigQuery before calculation of performance metrics
could be begun, the processing performed in BigQuery for the 2019 expressway CMP was much simpler than for the 2012 and 2015 CMPs.
Work on the 2019 expressway CMP was completed in March 2020, days before the onset of the Covid-19 “lockdown.”

Substantial changes were noted in the TMC network for arterials TMCs between 2015 and 2019, and needless to say the Road Inventory
had changed significantly in the intervening years \(including its migration to the ESRI 'Roads and Highways' product\).
Consequently, the 2019 arterial TMC network would need to be conflated with the 2019 Road Inventory.
Because of the expected magnitude of this task, work on it did not resume until after CTPS has returned to a trial hybrid work 
schedule in the summer of 2022. A description of this conflation effort is __TO BE WRITTEN__.
