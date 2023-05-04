# Data Fetch Process

## The Process Prior to 2019
Before the 2012 CMP, CTPS obtained speed and travel-time data for the roadways monitored for the CMP by __direct, manual data collection._ 
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

### Tabular Data
The raw tabular speed and travel-time data for 2012 was delivered to CTPS in 2014 in 28 ZIP archives, one for each (calendar quarter, county) pair.
This process was repeated for the 2015 CMP: the raw tabular speed and travel-time data was delivered to CTPS in 28 ZIP archives.

Once the ZIP archives have been obtained, they are 'exploded' and the CSV data files are uploaded to Google BigQuery.
A discussion of this follows in the document on __Data\_Preparation__.

### Spatial Data
A shapefile for the TMCs (road segments) in Massachusetts for 2012 was also shipped to CTPS with the raw speed and travel-time data. 

This shapefile was found to be severely defective. 
In general, the spatial accuracy of the linework was poor, with many line segments 10 or more meters from their ‘ground truth’ location. 
Further complicating maters, the directionality of the linework was inconsistent: for example, one segment of a north-south route might be drawn north-south, 
whereas the next might be drawn south-north. No ‘rhyme or reason’ was found to account for this variability.

In short, the shapefile delivered by INRIX was found to be only partially usable. 
The *approximate* locations of TMCs in it could be used, but review and correction would be needed to establish their their exact location directionality. 
Consequently, it was necessary to conflate the INRIX TMCs with the MassDOT Road Inventory in order to establish the correct location and directionality of each TMC. 
This conflation was further required because the __posted speed limit__ in effect in each TMC was required in order to calculate many performance metrics, 
and the __number of travel lanes__ was required for other reporting purposes. 
These two attributes - posted speed limit and number of travel lanes could be harvested from the Road Inventory. 

Although some parts of a conflation process can be performed by automation, inevitably the output of an automated process requires manual review and cleanup. 
Because of the very poor spatial quality of the 2012 INRIX TMC shapefile, manual review and editing/correction required approximately __1 person-year__:
four GIS temps working for approximately 3 calendar months.

Because of the very poor spatial accuracy of the INRIX TMC network, the decision was made to use the Road Inventory geometry as __the__ geometry for use 
in the calculation and reporting of CMP performance metrics. 
One consequence of this is that the length of each TMC in the INRIX shapefile could not be used as a reliable indication of TMC segment length. 
The approach taken was to calculate the length of each TMC ourselves using the __distance = rate * time__ method, 
taking the average of this over all records for each TMC in the input data.

For the 2015 CMP, the conflation process was performed again. In the intervening years, the spatial accuracy and 'directionality' of the INRIX
TMC file improved markedly. However, given our experience with the 2012 TMC shapefile, "to be on the safe side" we continued to use the 
Road Inventory geometry as __the__ geometry for the CMP and to calculate TMC lengths ourselves, rather than relying on the TMC length as given
in the TMC shapefile.

## The Process from 2019
In 2019 MassDOT, as part of the Eastern Transportation Coalition \(formerly the I-95 Corridor Coalition\) signed a multi-year contract with INIRX 
and the [Center for Advanced Transportation Technology (CATT) Laboratory](https://www.cattlab.umd.edu/) \(CATT Lab\) at the University of Maryland
 for the CATT Lab to house INRIX data and to develop client software providing access to it. 
 The MassDOT contract provided for all departments in MassDOT and all MPOs in the state to have access to RITS. 
 Initially, RITIS would be populated with data for the previous few years, but over time it would be populated with INRIX data going back to 2015. (This is now the case.)

Given this, it was no longer necessary for CTPS to purchase speed and travel-time data directly from INRIX. 
Moreover, since RITIS client software (the ‘Massive Data Downloader') would provide the ability to query for specific TMCs, specific dates and specific times, 
CTPS's processing of this data would be much simpler than in the past. 
Aside from its lack of a usable programmatic API, the only notable shortcoming of the RITIS GUI is that it does not support filtering records 
on the __cvalue__ field; this filtering must continue to be performed by CTPS. 

### Tabular Data
The URL for the RITIS Probe Data Analytics (PDA) suite is: [https://pda.ritis.org/suite/download/](https://pda.ritis.org/suite/download/)

Submit queries on this page for (1) the data for express highways, and (2) the data for arterial highways. 
Queries must be specified interactively using the RITIS GUI. \(There is a very primitive programmatic API for the RITIS Massive Data Downloader - see
 the commented code in [this GitHub repository](https://github.com/CTPSSTAFF/ritis-api-test) for more information - but it is unusable for practical purposes.\)

The necessary parameters for each query are:

-   A list of TMCs
-   A list of dates
-   A list of time periods
-   Other parameters

The list of relevant TMCs, dates, and times is discussed elsewhere.  
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

The user receives an email notification that the RITIS Massive Data Downloader job has completed.
The email message includes a link to click to download a ZIP archive that contains the query result in CSV format.
Once the data has been extracted, it is uploaded to Google BigQuery.
A discussion of this follows in the document on __Data\_Preparation__.

### Spatial Data
Examination the INRIX TMC shapefile for 2019 showed that there had been no changes to the TMC network for CMP express highways between 2015 and 2019. 
Consequently, the results of the 2015 conflation of the INRIX TMC network and the Road Inventory could be used as-is for expressway TMCs. 
Substantial changes were, however, noted in the arterial TMCs. These would need to be re-conflated.

Work on the 2019 CMP for expressways was completed in early Mach 2020, just before the onset of the Covid-19 pandemic.
Conflation of the arterial TMCs with the 2019 Road Inventory was performed during the Summer of 2022, and work on the 2019 arterial CMP was completed in the fall of 2022.