# Data Preparation

## Background / Context
The sheer volume of raw data (around three-quarters of a TeraByte) for the 2012 CMP rendered use of a conventional database, such as Oracle or PostgreSQL, 
out of the question for at least some processing.
Happily, in 2012 Google introduced BigQuery a cloud-hosted, massively parallel database suitable for handling PetaByte workloads. 
BigQuery was ideally suited to the task at hand, and we proceeded to process the CMP data using BigQuery. 
Part-way through this process, it became clear that the data had been filtered down to a size that could be handled by a conventional database.

This chapter discusses:
1. Data preparation performed before CSV data is uploaded to Google BigQuery
2. Data preparation performed __in__ Google BigQuery

Item \(1\) is common to the 2012 and 2015 CMPs.
Much of item \(2\) was not required in 2019 \(and should not be required going forward\) as RITIS is able to perform the relevant filtering.

## Data Prep Before Upload to BigQuery
Data prep before upload to BigQuery is concerned with two matters:
1. 'Exploding' the __measurement\_tstamp__ field in the INRIX data
2. Filtering records on the __cvalue__ field in the INRIX data

The measurement\_tstamp field the raw INRIX data is very awkward to work with. 
For example, it makes it difficult for anyone not fluent in the use of SQL string and sub-string operators to query the data for specific days of the week, 
ranges of dates, ranges of time periods, etc. 
In order to ameliorate this, the INRIX measurement\_tstamp field is programmatically ‘exploded’ into the following fields:

| field | datatype | description |
| ------| -------- | ----------- |
| month | integer | month on which sample was taken |
| day | integer | day of month on which sample was taken |
| dow | integer | day-of-the-week on which sample was taken |
| hour | integer | hour at which sample was taken |
| minute | integer | minute at which sample was taken |

The relevant code is stored on the lilliput file server:

\\\\lilliput\\groups\\Traffic*and*Design\\11123 CMP 2019 INRIX\\\\python\_scripts\\filter\_downloaded\_csv.py 

This script is a variant of the first one created for this purpose.
For the 2019 CMP, it was updated to also filter records on the __cvalue__ field.

As noted in the documentation on the format of the INRIX data records,
The __cvalue__ field "indicates the probability that the current probe reading represents the actual roadway conditions based on recent and historical trends."
CTPS practice is to only use records with a __cvalue__ of 75.0 or higher.
The script above does this, as well as filtering out records with empty or null __cvalue__ or __speed__ fields.

## Data Prep in BigQuery
Extensive data prep was performed in BigQuery for the 2012 and 2015 CMPs.
Because the data was packaged in multiple "chunks" the data had to be uploaded to BigQuery 
one "chunk" at a time, and the results stitched together into a single table.
\(While it was theoretically posssible to concatenate all the "chunks" before upload,
in practice this wasn't possible. Because of the volume of data being transferred
between the CTPS network and Google, attemps to transfer very large quantities of
data 'in one go' would invariably time out - after many hours - and have to be restarted.
This process never succeded, and we were forced to resort to chunk-by-chunk upload.\)

For the 2012 CMP, Windows batch \(.bat\) scripts were developed to call the Google Cloud Command Line Interface
to perform data prep in BigQuery. The original scripts for the 2012 CMP can no longer be found on the lilliput
file server. However, the ones for the 2015 CMP are still in place, and can be found in:

\\\\lilliput\\groups\\Traffic\_and\_Design\\11123 CMP 2015 INRIX\\batch\_files, which contains a number of sub-folders for each ‘stage’ of processing. The following naming convention was adopted for these scripts. Each script was given a name of the form:

-   n - a number indicating the major processing stage
-   an underscore
-   m - a number indicating the step within the given stage
-   a descriptive name

Thus, for example, the SQL script **1\_2\_load\_inrix\_2012\_quarter1\_suffolk.bat** was the second step in the first stage of processing, 
and is concerned with loading the data for Suffolk county for calendar quarter 1 of 2012.

There are a total of 6 Stages of processing; the first 5 are purely 'data prep' stages.
With the advent of RITIS, the queries in Stages 1 through 5 should no longer be needed.
The queries in Stage 6 continue to be used for 'real' 
data processing. \(Starting with the 2019 arterial CMP, these queries are executed directly in BigQuery rather than via a Windows .bat script calling the 
Google Cloud Command Line Interface. 

