# Definition of Metrics
This document defines the roadway performance metrics calculated for the CMP.

## Average Speed
The __average speed_ for a road segment is defined as:
```
Average Speed = Total speed value for all data samples / total number of data samples
```
Average Speed for specific roadway Traffic Message Channels (TMCs) and is calculated using travel times and segment lengths. 
The average speed is a good indicator of lack of mobility in a roadway network and is used for determining solutions to mobility problems.  
This metric is also known as "Average Travel Speed" and "Average Observed Travel Speed."

## Speed Index
The __speed index__ of a road segment is defined as:
```
Speed Index = average speed / posted speed limit (of a TMC)
```
Speed index indicates congestion more accurately than travel speed alone because low travel speed may be a result of low speed limits on certain facilities.

## Travel Time Index
The __travel time index__ for a road segment is defined as:
```
Travel time index = average peak-period travel time / free-flow travel time
```
Travel Time Index directly compares peak-period travel time conditions with free-flow travel time conditions. 
Travel time Index indicates how much contingency time should be considered to ensure an on-time arrival during the peak period versus optimum travel times.

## Planning Time Index
The __planning time index__ for a road segment is defined as:
```
Planning Time Index = 95 percentile travel time / free-flow travel time
```
Planning Time Index compares near-worst-case travel time to free-flow travel time to determine the contingency time needed to ensure on-time arrival 95 percent of the time. 
For example, a value of 2.5 means that to arrive on time 95 percent of the time, a traveler should budget an additional 45 minutes for a trip that takes 30 minutes during free-flow conditions.

## Congested Time
The __congested time__ for a road segment is defined as:
```
Congested Time = (number of samples with congestion below speed threshold / number of samples used) x number of minutes in measured period
```
Congested Time (measured in minutes per peak-period hour) is the average number of minutes that drivers experience congested conditions during the peak period. 
Congestion is considered to persist when the average speed is less than 35 miles per hour (MPH) on a limited-access roadway.

## Delay per Mile
The __delay per mile__ for a road segment is defined as:
```
Delay per mile = (average travel time - free flow travel time) / segment length
```
Delay per mile (displayed in seconds per mile) shows the extra time needed to traverse a road segment. 
This performance measure is presented in a per-mile metric to emphasize the intensity of congestion.

## Delay
The __delay__ for a road segment is defined as:
```
Formula to be provided.
``` 
Delay is expressed in minutes. 
This performance measure is only calculated for arterials; it is different from delay-per-mile.

## Level of Travel-time Reliability
The __level of travel-time reliability__ \(LOTTR\) for a road segment is defined as:
```
Level of Travel Time Reliability =  80th percentile travel time during peak period / 50th percentile travel time during peak period
```













