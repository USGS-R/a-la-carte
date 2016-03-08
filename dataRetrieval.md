# dataRetrival Tutorial
<b>USGS</b>: Laura DeCicco  
`r format(Sys.time(), '%d %B, %Y')`  



## Introduction

What is `dataRetrival`?

* R-package to get USGS/EPA water data into R

Where does the data come from?

* US Geological Survey water data
    + National Water Information System (NWIS)
* Water Quality Portal
    + USGS
    + EPA (EPA Storage and Retrieval Data Warehouse = STORET)
    + USDA
    + more being added....

What does `dataRetrival` do to the data?

* 

How to discover data?

* Examples will be provided



## Overview
<img src="../images/2015-12-15_13-52-29.png" alt="Overview" style="width: 800px;"/>

## Installation

`dataRetrieval` is available on the CRAN repository. The CRAN version is the most stable and user-tested:


```r
install.packages("dataRetrieval")
```

Bug fixes and feature upgrades are vetted through a version of `dataRetrival` that is available on a USGS-maintained R repository. To install from that repository:


```r
install.packages("dataRetrieval", 
                 repos=c("http://owi.usgs.gov/R",
                         getOption("repos")))
```

More information can be found [here](http://owi.usgs.gov/R/gran.html).

Finally, the absolute cutting-edge version of `dataRetrival` can be installed using the `devtools` package which pulls from GitHub:


```r
library(devtools)
install_github("USGS-R/dataRetrieval")
```

## dataRetrival Help

Once the `dataRetrieval` package has been installed, it needs to be loaded in order to use any of the functions:


```r
library(dataRetrieval)
```

There is a vignette that covers the full scope of the `dataRetrieval` package. It can be accessed with the following command:


```r
vignette("dataRetrieval",package = "dataRetrieval")
```

Additionally, each function has a help file. These can be accessed by typing a question mark, followed by the function name in the R console:


```r
?readNWISuv
```

Each function's help file has working examples to demonstrate the usage. The examples may have comments "## Not run". These examples CAN be run, they just are not run by the CRAN maintainors due to the external service calls.

Finally, if there are still questions that the vignette and help files don't answer, please post an issue on the `dataRetrieval` GitHub page:

<center>[https://github.com/USGS-R/dataRetrieval/issues](https://github.com/USGS-R/dataRetrieval/issues)</center>

## US Geological Survey Water Data Overview

National Water Information System (NWIS)

* Unit Data
    + "Real-time" data
    + Available data from 2007 (big improvement from 120 days!)
  
* Daily Data
    + Data aggregated from the unit data to a daily statistic
    + This data can go back many decades
    
* Discrete Data
    + Water quality data
    + Groundwater level
    + Rating curves
    + Surfacewater measurements
    
* Meta Data
    + Site information
    + Parameter information
    
## USGS Basic Web Retrievals

The USGS uses various codes for basic retrievals

* Site ID (often 8 or 15-digits)
* Parameter Code (5 digits)
    + [More information](http://help.waterdata.usgs.gov/code/parameter_cd_query?fmt=rdb&inline=true&group_cd=%)
* Statistic Code (for daily values)
    + [More information](http://help.waterdata.usgs.gov/code/stat_cd_nm_query?stat_nm_cd=%25&fmt=html)


Here are some examples of common codes:
<div class="columns-2">


------------------------------
 Parameter Codes   Short Name 
----------------- ------------
      00060        Discharge  

      00065       Gage Height 

      00010       Temperature 

      00400            pH     
------------------------------


------------------------------
 Statistic Codes   Short Name 
----------------- ------------
      00001         Maximum   

      00002         Minimum   

      00003           Mean    

      00008          Median   
------------------------------

</div>

The next 3 sections will show examples of using a few of `dataRetrieval`'s basic functions.

Following those basic examples, more complicated queries using `dataRetrival`'s more advanced functions will be introduced.

## USGS Basic Web Retrievals: readNWISuv

Knowing a site number (or site numbers), paremeter code (or codes), and start and end date. Let's start by asking for gage height (parameter code = 00065) data for Lake Mendota at Madison, WI (right off Tenney Park on the Yahara River). 


```r
siteNo <- "05428000"
pCode <- "00065"
start.date <- "2014-10-01"
end.date <- "2015-09-30"

yahara <- readNWISuv(siteNumbers = siteNo,
                     parameterCd = pCode,
                     startDate = start.date,
                     endDate = end.date)
```

## USGS Basic Web Retrievals: renameNWISColumns

From the Yahara example, let's look at the data. The column names are:


```r
names(yahara)
```

```
## [1] "agency_cd"        "site_no"          "dateTime"        
## [4] "X_00065_00011"    "X_00065_00011_cd" "tz_cd"
```


The names of the columns are based on the parameter and statistic codes. In many cases, you can clean up the names with a convenience function renameNWISColumns:


```r
yahara <- renameNWISColumns(yahara)
names(yahara)
```

```
## [1] "agency_cd"  "site_no"    "dateTime"   "GH_Inst"    "GH_Inst_cd"
## [6] "tz_cd"
```

## Explore Data


```r
head(yahara)
```

```
##   agency_cd  site_no            dateTime GH_Inst GH_Inst_cd tz_cd
## 1      USGS 05428000 2014-10-01 05:00:00    9.96          A   UTC
## 2      USGS 05428000 2014-10-01 05:15:00    9.95          A   UTC
## 3      USGS 05428000 2014-10-01 05:30:00    9.95          A   UTC
## 4      USGS 05428000 2014-10-01 05:45:00    9.96          A   UTC
## 5      USGS 05428000 2014-10-01 06:00:00    9.96          A   UTC
## 6      USGS 05428000 2014-10-01 06:15:00    9.97          A   UTC
```

The data is returned as a data frame with 5 columns:

---------------------
 Column Name   Type  
------------- -------
  agency_cd     chr  

   site_no      chr  

  dateTime    POSIXct

   GH_Inst      num  

 GH_Inst_cd     chr  

    tz_cd       chr  
---------------------
  
## Explore Data (cont.)

The returned data also has several attributes attached to the data frame. To see what the attributes are:


```r
names(attributes(yahara))
```

```
## [1] "names"         "row.names"     "class"         "url"          
## [5] "siteInfo"      "variableInfo"  "disclaimer"    "statisticInfo"
## [9] "queryTime"
```

Each `dataRetrieval` return should have the attributes url, siteInfo, and variableInfo. Additional attributes are available depending on the data.

To access the attributes:


```r
url <- attr(yahara, "url")
```

[Raw Data](http://nwis.waterservices.usgs.gov/nwis/iv/?site=05428000&format=waterml,1.1&ParameterCd=00065&startDT=2014-10-01&endDT=2015-09-30)

##  Explore Data (cont.)


```r
library(ggplot2)
ts <- ggplot(yahara,
             aes(dateTime, GH_Inst)) +
      geom_line()
ts
```

![](..\dataRetrieval_files/figure-html/unnamed-chunk-16-1.png)
    
##  Use attributes for metadata:



```r
parameterInfo <- attr(yahara, "variableInfo")
siteInfo <- attr(yahara, "siteInfo")
  
ts <- ts +
      xlab("") +
      ylab(parameterInfo$parameter_desc) +
      ggtitle(siteInfo$station_nm)
ts
```

![](..\dataRetrieval_files/figure-html/unnamed-chunk-17-1.png)

    
    
## Water Quality Portal

[Water Quality Portal](http://www.waterqualitydata.us/)

<img src="../images/WQP.png" alt="Overview" style="width: 300px;"/>


## More information:

NWIS

* [Water Services](http://waterservices.usgs.gov/)

* [Help](http://help.waterdata.usgs.gov/)

Water Quality Portal

* [Water Quality Portal](http://www.waterqualitydata.us/)


