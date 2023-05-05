# Calculation of Metrics

## Introduction
The calculation of CMP metrics is carried out in two phases:
1. processing performed in Google BigQuery
2. processing peformed in a 'desktop' database

The job of phase \(1\) is, essentially, to 'boil down' the data to the 
point that it is small enough to be processed in a 'desktop' database.
While moving the final phase processing into a 'desktop' database may not be
required froma a purely theoretical point of view, it has been done
in the past to help reduce BigQuery costs - particularly when the 
CMP project and Data Resources budgets were under great stress.

## Processing Performed in Google BigQuery

### Logical Process
Abstractly the processing performed in Google BigQuery does the following:
0. combine the tables for the AM and PM peak periods into a single table; this is used to calculate TMC length
1. calculation of the length of each TMC
2. calculation of the total number of records for the AM  and PM peak periods for each TMC
3. calculation of the total number of congested records for the AM and PM peak periods for each TMC
5. calculation of the sum of observed speeds across all records for the AM and PM peak periods for each TMC;
   this is a preliminary to calculating the 85th and 5th percentile speeds during the two peak periods for each TMC 
6. calculation of the 5th and 85th percentile speed for AM and PM peak periods for each TMC
7. calculation of the 85th percentile speed for midnight to 2 AM, 2 AM to 4 AM, and midnight to 4 AM for each TMC;
   this is used to help estimate the free-flow speed for each TMC
8. calculation of the 50th and 80th percentile travel time for each TMC; these metrics are used to calculate
   the level of travel-time reliability for each TMC
10. estimation of the free-flow speed for each TMC

In order to calculate the length of each TMC the data for the separate AM and PK peak-period tables
is also combined into a single table so as to provide a larger sample-size for this calculation.

### Implementation
The abstract process described above is implemented bu running a series of queries in Google BigQuery, 
using Windows batch (.bat) scripts that call the Google Clould Command Line Interface.

Fore each CMP year, these batch scripts are stored in a subfolder in the Traffic\_and\_Design group folder on the lilliput file server.
For the 2015 CMP, they are stored in:  
\\lilliput\groups\Traffic_and_Design\11123 CMP 2015 INRIX\batch_files\_06\_perf\_meas  

For the 2019 CMP, they are stored in:  
\\lilliput\groups\Traffic_and_Design\11123 CMP 2019 INRIX\batch_files

The following subsections document the scripts that implement each of the logical steps above.

#### Combination of AM and PM Tables
This operation is performed by the scripts:
* 6\_1\_00a\_create\_inrix\_2019\_cmp\_art\_ampm.bat 
* 6\_1\_00b\_create\_inrix\_2019\_cmp\_exp\_ampm.bat

#### Calculation of TMC Length
The length of each TMC is calculated by the scripts:
* 6\_1\_03\_create\_inrix\_2019\_cmp\_art\_avg\_length.bat art
* 6\_1\_03\_create\_inrix\_2019\_cmp\_exp\_avg\_length.bat 

#### Calculation of Total Number of Records
This calculation is performed for each TMC for each of the two peak periods.
The relevant scripts are: 
* 6\_2\_01\_create\_inrix\_2019\_cmp\_art\_count\_all\_tmc\_am.bat
* 6\_2\_01\_create\_inrix\_2019\_cmp\_exp\_count\_all\_tmc\_am.bat
* 6\_2\_02\_create\_inrix\_2019\_cmp\_art\_count\_all\_tmc\_pm.bat
* 6\_2\_02\_create\_inrix\_2019\_cmp\_exp\_count\_all\_tmc\_pm.bat

#### Calculation of Total Number of Congested Records
This calculation is performed for each TMC for each of the two peak periods.
The relevant scripts are:  
* 6\_3\_01\_create\_inrix\_2019\_cmp\_art\_count\_cong\_tmc\_am.bat
* 6\_3\_01\_create\_inrix\_2019\_cmp\_exp\_count\_cong\_tmc\_am.bat
* 6\_3\_02\_create\_inrix\_2019\_cmp\_art\_count\_cong\_tmc\_pm.bat
* 6\_3\_02\_create\_inrix\_2019\_cmp\_exp\_count\_cong\_tmc\_pm.bat

Note that an __expressway segment__is considered congested__ when the average speed falls below __35 MPH__,
and an __artierial segment__ is considered congested when the average speed falls below __19 MPH__.

#### Calculation of "Total Speed"
This calculation is performed for each TMC for each of the two peak periods by the scripts:  
* 6\_5\_01\_create\_inrix\_2019\_cmp\_art\_sum\_speed\_cong\_tmc\_am.bat
* 6\_5\_01\_create\_inrix\_2019\_cmp\_exp\_sum\_speed\_cong\_tmc\_am.bat
* 6\_5\_02\_create\_inrix\_2019\_cmp\_art\_sum\_speed\_cong\_tmc\_pm.bat
* 6\_5\_02\_create\_inrix\_2019\_cmp\_exp\_sum\_speed\_cong\_tmc\_pm.bat

#### Calculation of 5th Percentile of Speed Distribution
This calculation is performed for each TMC for each of the two peak periods by the scripts:
* 6\_6\_01\_create\_inrix\_2015\_cmp\_art\_5pct\_speed\_tmc\_am.bat
* 6\_6\_01\_create\_inrix\_2015\_cmp\_exp\_5pct\_speed\_tmc\_am.bat
* 6\_6\_02\_create\_inrix\_2019\_cmp\_art\_5pct\_speed\_tmc\_pm.bat
* 6\_6\_02\_create\_inrix\_2019\_cmp\_exp\_5pct\_speed\_tmc\_pm.bat

### Calculation of 85th Percentile "Early Morning" Speed
These calculations are performed for each TMC for the time periods midnight to 2 AM, 2 AM to 4 AM, and midnight to 4 AM by the following scripts:
* 6\_7\_01\_create\_inrix\_2019\_cmp\_art\_85pct\_speed\_0\_to\_2\_tmc.bat
* 6\_7\_01\_create\_inrix\_2019\_cmp\_exp\_85pct\_speed\_0\_to\_2\_tmc.bat
* 6\_7\_03\_create\_inrix\_2019\_cmp\_art\_85pct\_speed\_2\_to\_4\_tmc.bat
* 6\_7\_03\_create\_inrix\_2019\_cmp\_exp\_85pct\_speed\_2\_to\_4\_tmc.bat
* 6\_7\_05\_create\_inrix\_2019\_cmp\_art\_85pct\_speed\_0\_to\_4\_tmc.bat
* 6\_7\_05\_create\_inrix\_2019\_cmp\_exp\_85pct\_speed\_0\_to\_4\_tmc.bat

#### Calculation of 50th and 80th Percentile Travel Time 
The 50th and 80th percentile travel times  are used as inputs tofor the calculation of
the level of travel-time reliability \(LOTTR\) metric.
These calculation of these two metrics were performed for the 2019 arterial CMP for each TMC by the scripts:
* 6\_20\_01\_cr\_Inrix\_2019\_cmp\_art\_80pct\_tt\_tmc\_am\_v2 
* 6\_20\_02\_cr\_Inrix\_2019\_cmp\_art\_80pct\_tt\_tmc\_pm\_v2
* 6\_20\_03\_cr\_Inrix\_2019\_cmp\_art\_50pct\_tt\_tmc\_am\_v2
* 6\_20\_04\_cr\_Inrix\_2019\_cmp\_art\_50pct\_tt\_tmc\_pm\_v2

The suffix '_v2' indicates that these queries were run twice, after an error in the set of TMCs to be included in the analysis was found and corrected.

The \(LOTTR\) metric was introduced in 2022, after work on the 2019 expressway CMP had been completed.
Calculation of the Level of Travel-time Reliability metrics for the 2019 CMP was implemented 
_ex-post-facto_ using a Jupyter notebook stored in [this GitHub repository](https://github.com/CTPSSTAFF/cmp-2019-expressway-lottr-metrics).

#### Estimation of Free-flow Speed
In general, free-flow speed is taken as the average speed during the early morning hours, from midnight to 4 AM.
Because of the volume of traffic for expressways, there are always "real" data samples (__confidence\_score__ = 30) records available during this time period.
This is not always the case for arterials.
For a number of arterials, there are TMCs for which there are no "real" data samples available during the CMP days for a specified year.
To estimate free-flow speed for arterials, the script 6\_7\_09b\_create\_Inrix\_2019\_cmp\_art\_free\_flow\_speed\_v2\_incomplete is run.
This produces the table INRIX\_2019\_cmp\_art\_free\_flow\_speed\_v2\_incomplete, which is then downloaded;
missing free-flow speed values are filled in manually in ArcGIS Pro by harvesting the posted speed limit for each segment from the Road Inventory.

## Download of Data from Google BigQuery
At this point, the tables produced by processing in Google BigQuery are downloaded in CSV format from Google BigQuery and loaded into a 'desktop' database \(see below\) for further processing.
The names of the relevant tables are of the form: for the 2019 arterial CMP are:

1.  INRIX\_yyyy\_cmp\_XXX\_avg\_length
2.  INRIX\_yyyy\_cmp\_XXX\_count\_all\_tmc\_am
3.  INRIX\_yyyy\_cmp\_XXX\_count\_all\_tmc\_pm
4.  INRIX\_yyyy\_cmp\_XXX\_count\_cong\_tmc\_am
5.  INRIX\_yyyy\_cmp\_XXX\_count\_cong\_tmc\_pm
6.  INRIX\_yyyy\_cmp\_XXX\_sum\_speed\_all\_tmc\_am
7.  INRIX\_yyyy\_cmp\_XXX\_sum\_speed\_all\_tmc\_pm
8.  INRIX\_yyyy\_cmp\_XXX\_sum\_speed\_cong\_tmc\_am
9.  INRIX\_yyyy\_cmp\_XXX\_sum\_speed\_cong\_tmc\_pm
10.  INRIX\_yyyy\_cmp\_XXX\_5pct\_speed\_tmc\_am
11.  INRIX\_yyyy\_cmp\_XXX\_5pct\_speed\_tmc\_pm
12.  INRIX\_yyyy\_cmp\_XXX\_free\_flow\_speed
13.  INRIX\_yyyy\_cmp\_XXX\_80pct\_tt\_am\_tmc
14.  INRIX\_yyyy\_cmp\_XXX\_80pct\_tt\_pm\_tmc
15.  INRIX\_yyyy\_cmp\_XXX\_50pct\_tt\_am\_tmc
16.  INRIX\_yyyy\_cmp\_XXX\_50pct\_tt\_pm\_tmc

Where __yyyy__ indicates the year, and __XXXX_ is either __exp__ \(for expressways\) or __art__ \(for artierials\).

For example, for the 2019 arterial CMP, the tables were named:
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

As noted above:
* The suffix '_v2' indicates that the relevant query was run twice, after an error in the set of TMCs to be included in the analysis was found and corrected.
* Missing free-flow speed values in INRIX\_2019\_cmp\_art\_free\_flow\_speed\_v2\_incomplete are filled in manually.

## Processing Performed in a 'Desktop' Database

### Logical Process
The logical (or 'abstract') processing peformed in a 'desktop' database is as follows:
* Calculate congested speed percentage and congested minutes per peak hour
* Calculate average speed for am and pm peaks
* Calculate average congested speed for am and pm peaks
* In some cases, generally those in which the calculations were performed on a very small number of observations, the estimated free-flow speed is less than one
   or both peak period speeds. In these cases, set the free-flow speed to the maximum of the AM and PM peak average speeds. __NOT NEEDED NOT DONE for EXP__
* Calculate the speed index for the AM and PM peak periods as the ratio of average speed to posted speed limit
* Calculate the average travel time for the am and pm peaks equal to  60 * segment length / average speed
* Calculate the average free flow travel time equal to 60 * segment length / free flow speed
* Calculate the 5th percentile travel time for the am and pm peaks equal to  60 * segment length / 5th percentile speed
* Calculate the average delay as the difference between peak period average travel time and free flow travel time
* Calculate the delay per mile as the ratio of average delay to segment length
* Calculate the travel time index as the ratio of average travel time to free flow travel time
* Calculate the planning time index as the ratio of 5th percentile travel time to free flow travel time

### 2012 and 2015 CMPs and 2019 Expressway CMP
__MS Access__ was used as the ‘desktop’ database for the 2012 Expressway and Arterial CMPs, the 2015 Expressway and Arterial CMPs, and the 2019 Expressway CMP. 
Work on  the 2019 arterial CMP was interrupted in March 2019 by the onset of the COVID-19 pandemic. 
\(At that point, conflation of the 2019 arterial TMCs with the Road Inventory had not taken place.\)
The MS Access database in which the final calculation of performance metrics for the 2015 CMP was performed is found in: 

\\\\lilliput\\groups\\Traffic\_and\_Design\\11123 CMP 2015 INRIX\\performance\_measures\\inrix\_2015\_perf\_meas.mdb. 

This database contains the data downloaded from processing in BigQuery, the queries to process it in MS Access, and the tables containing the final results.

#### 2019 Expressway CMP
The MS Access queries used for the 2019 expressway CMP are as follows:


### 2019 Arterial CMP and Expressway and Arterial CMP Going Forward
Work on the 2019 artieral CMP resumed in the summer of 2022. 
By 2022 it had become abundantly clear that MS Access was getting very long in the tooth, and migration to a more up-to-date platform was advisable. 
We chose to migrate from MS Access to [SQLite](https://sqlite.org/index.html),a more modern desktop database that is used elsewhere in CTPS.
This allowed us to leverage the SQL queries used in the past, albeit with a non-trivial amount of massaging to account for differences in the dialect of SQL supported by the two databases. 
This document is written with the assumption that SQLite will continue to be used going forward.

#### Installation of SQLite
1. Download the SQLite executable (sqlite3.exe) from the SQLIte website (https://www.sqlite.org/index.html) and put it in the working directory for the performance measures.
2.	Open your working directory in Windows Explorer and type “cmd” in the path bar; this will open a command prompt window where you can enter SQLite queries.
3.	The performance measures calculations are performed in an SQLite database \(a .db file\). To create a new database, type “.open \[databasename\].db”

#### Calculation of Metrics for Arterials
Before running the queries, you’ll need to import the tables from BigQuery into the SQLite database. 
Use the code in the file  
\\lilliput\\groups\\Traffic\_and\_Design\\11123\11123 CMP 2019 INRIX\arterials\_SQLite\import_csvs.txt  
and enter it directly into SQLite.

You are now ready to enter the queries, which will create new tables in the SQLite database and populate them with the performance measures.


