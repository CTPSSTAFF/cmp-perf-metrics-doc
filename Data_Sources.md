# Data Sources
The primary data sources for the calculation of CMP performance metrics are:
-   INRIX speed and travel time data
-   INRIX TMC network for Massachusetts
-   Conflation of INIRX TMC network with Massachusetts Road Inventory

In addtion for any given year's CMP, the following are required:
-   List of CMP routes
-   List of TMCs for Express Highways in the region
-   List of TMCs for Arterial Highways in the MPO area
-   List of dates used for this year's CMP
-   List of AM and PM peak periods for Express Highways
-   List of AM and PM peak periods for Arterial Highways

Each of these is discussed individually.

## INRIX Speed and Travel Time Data
Raw INRIX speed and travel time data downloaded from RITIS (or purchased directly from INRIX) is in CSV file format, and has the following columns:

| **Field** | **Contents** | **Datatype** |
| --- | --- | --- |
| tmc\_code | TMC identifier | text |
| measurement\_tstamp | date and time (see below for format) | text |
| speed | average observed speed for the time period beginning *measurement\_tstamp* | floating point |
| average\_speed | see below | integer |
| reference\_speed | see below | integer |
| travel\_time\_minutes | average travel time through the TMC for the time period beginning *measurement\_tstamp*, in minutes | floating point |
| confidence\_score | always = 30; see below | integer |
| cvalue | see below | floating point |

Notes on the individual fields:
The **measurement\_tstamp** field is in the format: \[m\]m/\[d\]d/yyyy \[h\]h:mm \(UNIX regular expression syntax\)  

Examples:
-   4/19/2023 9:59
-   4/19/2023 10:00
-   11/5/2022 0:01
-   11/10/2022 23:59

**average\_speed** - INRIX-computed average speed

**reference\_speed** - INRIX-computed reference speed

__confidence\_score__ -This field has 3 possible values: 30, 20, 10. However, CTPS only uses (and thus only downloads) records with
 a __confidence\_score__ of 30. Descriptions of the 3 possible values:

**30**: Real-time data. Any segment that has adequate data, at any time of day, will report real time data.

**20**: Historical average.  Between 4 am and 10 pm, any segment without sufficient real time data will show the historical average for that segment during that day/time period (15 minute granularity).

**10**: Reference speed.  From 10 pm to 4 am, any segment without sufficient real time data will show the reference speed for that segment. Any segment that does not have calculated historical averages will show the reference speed 24 hours a day if there is not sufficient real time data.

**cvalue** - This field indicates the probability that the current probe reading represents the actual roadway conditions based on recent and historical trends. 
CTPS only uses records with a **cvalue** of 75 or higher. 

**Note:** RITIS does not support filtering of data for extraction and download based on the value of the cvalue field (it does support filtering for extraction and download based
on the value of the **confidence\_score** field); consequently, this filtering must be done as part of CTPS's processing.

## INRIX TMC Network for Massachusetts
INRIX reports speeds and travel times against [Traffic Message Channel \(TMC\)](https://en.wikipedia.org/wiki/Traffic_message_channel) geometry.
TMCs are defined by an international consoritum. INRIX does not disclose what organization is its _map provider_ \(i.e., provider of TMC geometry\),
but it is widely believed to be [HERE Technologies](https://www.here.com/) \(formerly NavTeq\).
TMCs correspond roughly to the notion of "road segments", but TMCs rarely align with the segmentation of the Road Inventory in its "flat file" format.

The shapefile defining the INRIX CMP network for a given year must be obtained directly from INRIX; 
it is not available on the RITIS platform. (The shapefile for the much more limited NPMRDS CMP network, however, is available from RITIS.) 
Currently, CTPS's contact for this data is [Mike Massaro](mailto://mmassaro@inrix.com), CTPS's Sales Engineer at INRIX.

As delivered, the INRIX TMC shapefile uses the EPSG:4326 ("lat, lon") Spatial Reference System (SRS). 
The data is projected to the EPSG:26986 ("Mass State Plane, NAD 83, meters") SRS and stored in CTPS's enterprise Geodatabase ('mpodata'), currently hosted on appsrvr3. 
The name of the dataset is formed as follows:

mpodata.mpodata.INRIX\_MASSACHUSETTS\_TMC\_*yyyy*\_*n*

-   *yyyy* \- indicates the year of the data
-   *n* \- indicates the release number, within the given year, of the data

INRIX releases the TMC shapefile twice a year: once in the spring and once in the fall. The spring release is release #1; the fall release is release #2.  
Hence the value of *n* may only be 1 or 2.

The metadata for the TMC shapefiles can be found in [INRIX metadata (on page 101, in Appendix A)](https://tetcoalition.org/wp-content/uploads/2015/02/I-95_VPP_II_Inteface_Guide-March_2018.pdf)

## Conflation of INIRX TMC network with Massachusetts Road Inventory
This is a very complex process, requiring a non-trivial amount of manual intervention after the application of an automated process. 
The conflation process is to be documented elsewhere.

The output of the conflation process is a geographic feature class \(including attribute table\) with the following fields:

| **Column** | **Contents** | **Data Type** |
| --- | --- | --- |
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

## List of CMP Routes
The list of "routes" covered by the CMP is defined by the CMP manager, and has not varied for at least two decades.
__CMP routes__ do not necessarily align with __MassDOT routes__. 
Among other things:
* if a CMP route extends beyond the MPO area (for arterials) or the former CTPS 'model region' (for expressways) ,the portion of the route outside these boundaries is excluded
* routes of interest that are not a MassDOT route with a posted route number (e.g., Soldiers Field Road) are included 

As of 2023, the list of CMP routes with their associated __CMP route ID__ is given in the following table:  
|CMP route ID|route|direction|
|-----------|-----|---------|
|1|I-290 Eastbound|EB|
|2|I-290 Westbound|WB|
|3|I-495 Northbound|NB|
|4|I-495 Southbound|SB|
|5|I-90 Eastbound|EB|
|6|I-90 Westbound|WB|
|7|I-93 Northbound|NB|
|8|I-93 Southbound|SB|
|9|I-95 Northbound|NB|
|10|I-95 Southbound|SB|
|11|MA-128 Northbound|NB|
|12|MA-128 Southbound|SB|
|13|MA-2 Eastbound|EB|
|14|MA-2 Westbound|WB|
|15|MA-24 Northbound|NB|
|16|MA-24 Southbound|SB|
|17|MA-3 Northbound|NB|
|18|MA-3 Southbound|SB|
|19|US-1 Northbound|NB|
|20|US-1 Southbound|SB|
|21|US-3 Northbound|NB|
|22|US-3 Southbound|SB|
|101|Beacon Street Eastbound|EB|
|102|Beacon Street Westbound|WB|
|103|Main Streets Northbound|NB|
|104|Main Streets Southbound|SB|
|105|Massachusetts Ave Eastbound|EB|
|106|Massachusetts Ave Westbound|WB|
|107|Middlesex Tpk/Lowell St Northbound|NB|
|108|Middlesex Tpk/Lowell St Southbound|SB|
|109|Route 107 Northbound|NB|
|110|Route 107 Southbound|SB|
|111|Route 109 Eastbound|EB|
|112|Route 109 Westbound|WB|
|113|Route 114 Eastbound|EB|
|114|Route 114 Westbound|WB|
|115|Route 115 Northbound|NB|
|116|Route 115 Southbound|SB|
|117|Route 117 Eastbound|EB|
|118|Route 117 Westbound|WB|
|119|Route 119 Eastbound|EB|
|120|Route 119 Westbound|WB|
|121|Route 123 Eastbound|EB|
|122|Route 123 Westbound|WB|
|123|Route 126 Northbound|NB|
|124|Route 126 Southbound|SB|
|125|Routes 129 and 129A Eastbound|EB|
|126|Routes 129 and 129A Westbound|WB|
|127|Route 135 Eastbound|EB|
|128|Route 135 Westbound|WB|
|129|Route 138 Northbound|NB|
|130|Route 138 Southbound|SB|
|131|Route 139 East Eastbound|EB|
|132|Route 139 East Westbound|WB|
|133|Route 139 Eastbound|EB|
|134|Route 139 Westbound|WB|
|135|Route 14 Eastbound|EB|
|136|Route 14 Westbound|WB|
|137|Route 140 Eastbound|EB|
|138|Route 140 Westbound|WB|
|139|Route 16 East Eastbound|EB|
|140|Route 16 East Westbound|WB|
|141|Route 16 West Eastbound|EB|
|142|Route 16 West Westbound|WB|
|143|Route 18 Northbound|NB|
|144|Route 18 Southbound|SB|
|145|Route 1A Far North Northbound|NB|
|146|Route 1A Far North Southbound|SB|
|147|Route 1A North Northbound|NB|
|148|Route 1A North Southbound|SB|
|149|Route 1A South Northbound|NB|
|150|Route 1A South Southbound|SB|
|151|Route 20 Eastbound|EB|
|152|Route 20 Westbound|WB|
|153|Route 203 Eastbound|EB|
|154|Route 203 Westbound|WB|
|155|Route 228 Northbound|NB|
|156|Route 228 Southbound|SB|
|157|Route 27 Northbound|NB|
|158|Route 27 Southbound|SB|
|159|Route 28 North Northbound|NB|
|160|Route 28 North Southbound|SB|
|161|Route 28 South Northbound|NB|
|162|Route 28 South Southbound|SB|
|163|Route 2A East Eastbound|EB|
|164|Route 2A East Westbound|WB|
|165|Route 2A West Eastbound|EB|
|166|Route 2A West Westbound|WB|
|167|Route 30 East Eastbound|EB|
|168|Route 30 East Westbound|WB|
|169|Route 30 West Eastbound|EB|
|170|Route 30 West Westbound|WB|
|171|Route 37 Northbound|NB|
|172|Route 37 Southbound|SB|
|173|Route 38 Northbound|NB|
|174|Route 38 Southbound|SB|
|175|Route 3A South Northbound|NB|
|176|Route 3A South Southbound|SB|
|177|Route 4 and 225 Northbound|NB|
|178|Route 4 and 225 Southbound|SB|
|179|Route 4 Northbound|NB|
|180|Route 4 Southbound|SB|
|181|Route 53 Northbound|NB|
|182|Route 53 Southbound|SB|
|183|Route 60 Eastbound|EB|
|184|Route 60 Westbound|WB|
|185|Route 62 East Eastbound|EB|
|186|Route 62 East Westbound|WB|
|187|Route 62 West Eastbound|EB|
|188|Route 62 West Westbound|WB|
|189|Route 85 Northbound|NB|
|190|Route 85 Southbound|SB|
|191|Route 9 East Eastbound|EB|
|192|Route 9 East Westbound|WB|
|193|Route 9 West Eastbound|EB|
|194|Route 9 West Westbound|WB|
|195|Route 99 Northbound|NB|
|196|Route 99 Southbound|SB|
|197|US Route 1 Far North Northbound|NB|
|198|US Route 1 Far North Southbound|SB|
|199|US Route 1 South Northbound|NB|
|200|US Route 1 South Southbound|SB|
|201|US Routes 3 and 3A Northbound|NB|
|202|US Routes 3 and 3A Southbound|SB|
|203|VFW Pkwy/Providence Hwy Northbound|NB|
|204|VFW Pkwy/Providence Hwy Southbound|SB|
|205|Washington Street Northbound|NB|
|206|Washington Street Southbound|SB|
|207|Storrow Dr/Soldiers Field Rd Eastbound|EB|
|208|Storrow Dr/Soldiers Field Rd Westbound|WB|
|209|Memorial Dr/Fresh Pond Pk Eastbound|EB|
|210|Memorial Dr/Fresh Pond Pk Westbound|WB|

## List of TMCs for CMP Express Highways in the region
Once the conflation process has been completed, a list of all TMCs for CMP express highways in the region can be extracted.
The list of all TMCs for the CMP express highways (within the former “CTPS model region”) for 2019 is found in the file: 

\\\\lilliput\\groups\\Traffic*and*Design\\11123 CMP 2019 INRIX\\TMC\_lists\\raw\\all\_expressways\_in\_model\_region.txt

## List of TMCs for CMP Arterial Highways in the MPO area
A comprehensive list of all TMCs for the CMP arterial highways within the MPO for 2019 is found in the file:

\\\\lilliput\\groups\\Traffic*and*Design\\11123 CMP 2019 INRIX\\arterial\_conflation\\TMC\_Lists\\raw\\all-tmcs-2022-09-30.txt

## List of dates used for this year's CMP
The list of dates for which CMP performance metrics will be calculated for this year's CMP is obtained from CTPS's CMP Manager. 
These dates comprise the dates for roughly 60 Tuesdays, Wednesdays, and Thursdays in the spring and fall that do not fall on weeks that are deemed ‘special’, 
i.e., for which travel behavior may well vary from the norm. One such example week is school vacation weeks.

The dates for the 2019 CMP are:
-   March 5-7, 12-14, 19-21, 26-28
-   April 2-4. 9-11, 23-25, 30
-   May 1-2, 7-9, 14-16, 21-23
-   September 10-12, 17-19, 24-26
-   October 1-3, 8-10, 15-17, 22-24, 29-31
-   November 5-7, 12-14, 19-21

## List of AM and PM peak periods for Express Highways
The temporal bounds of the AM and PM peak periods for the CMP have not changed for more than 10 years. 
However, before setting out to begin calculating CMP performance metrics for any given years, check with the CTPS CMP manager to verify the bounds of the temporal periods to use.

As of 2023, the CMP peak periods for express highways are:
- 6:00 a.m. to 10:00 a.m.
- 3:00 p.m. to 7:00 p.m.

## List of AM and PM peak periods for Arterial Highways
The temporal bounds of the AM and PM peak periods for the CMP have not changed for more than 10 years. 
However, before setting out to begin calculating CMP performance metrics for any given years, check with the CTPS CMP manager to verify the bounds of the temporal periods to use.

- 6:30 a.m. to 9:30 a.m.
- 3:30 p.m. to 6:30 p.m.

