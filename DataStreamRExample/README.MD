
# DataStream Web Service R Example

This example demonstrates how to use [DataStream Web Service](https://developers.refinitiv.com/eikon-apis/datastream-web-service) with R on Jupyter Notebook. It uses the [DatastreamDSWS2R](https://github.com/CharlesCara/DatastreamDSWS2R/tree/master/R) package to retrieve data from DataStream Web Service and uses [Plotly](https://plot.ly/r/getting-started/) package to draw charts. 

To setup Jupyter Notebook environment for R or install DatastreamDSWS2R, please refer to this [article](https://developers.refinitiv.com/article/setup-jupyter-notebook-r).

## The first step is loading the required libraries:

- [DatastreamDSWS2R](https://github.com/CharlesCara/DatastreamDSWS2R): Functions and a R5 class to get data from the DSWS server
- [xts](https://www.rdocumentation.org/packages/xts/versions/0.11-2): eXtensible Time Series for uniform handling of R's different time-based data classes 
- [dplyr](https://www.rdocumentation.org/packages/dplyr/versions/0.7.8): A fast, consistent tool for working with data frame like objects, both in memory and out of memory
- [plotly](https://www.rdocumentation.org/packages/plotly/versions/4.9.0): Create interactive web graphics from 'ggplot2' graphs and/or a custom interface to the (MIT-licensed) JavaScript library 'plotly.js'



```R
library(DatastreamDSWS2R)
library(xts)
library(dplyr)
library(plotly)
```

## Next, set DataStream Web Service username and password

There are two ways to set DataStream Web Service username and password.

1. Set credentials in environment variables by adding to the **.Renviron** file the following lines

```
DatastreamUsername=YOURUSERNAME
DatastreamPassword=YOURPASSWORD
```

2. Put credentials into the **options()** method which allows the user to set and examine a variety of global options. You can also add these lines to the **.RProfile** file.

```
options(Datastream.Username = "YOURUSERNAME")
options(Datastream.Password = "YOURPASSWORD")
```

The below code calls the **options()** method to set the DataStream Web Service credentials. The below code can be skipped if the credentials have been set in the **.Renviron** or **.RProfile** file.


```R
options(Datastream.Username = "YOURUSERNAME")
options(Datastream.Password = "YOURPASSWORD")
```

## Then, create an instance of the DataStream Web Service 


```R
mydsws <- dsws$new()
```

Now, the DataStream Web Service is ready to be used. Next, we will show several use cases for the DataStream Web Service.

## The following examples show several use cases for the DataStream Web Service

### 1. Retrieve static information for given instruments on a specific date

The following code calls the **snapshotRequest** function to get the name (NAME), official closing price (P) and opening price (PO) of ABR, RIO, and WPP instruments on the last trading day. 

The function returns a data frame with fields in columns and instruments as rows.


```R
data1 <- mydsws$snapshotRequest(instrument = c("ABF","RIO","WPP"),
                               datatype = c("NAME","P","PO"),
                               requestDate ="0D")
data1
```

### 2. Retrieve time series data and plot OHLC chart

The following code calls the **timeSeriesListRequest** function to retrieve daily historical OPEN, HIGH, LOW, CLOSE fields from one year ago to the last trading day of IBM.

The format is **"ByDatatype"** so the function returns xts data (eXtensible Time Series) with fields in columns and data points in rows.


```R
xtsOHLCData <- mydsws$timeSeriesListRequest(instrument = c("U:IBM"),
                                            datatype = c("PO", "PH","PL","P"),
                                            startDate = "-1Y",
                                            endDate = "-0D",
                                            frequency = "D",
                                            format="ByDatatype")

xtsOHLCData
```

#### Change the column name to OPEN, HIGH, LOW, CLOSE

The column names in the xts data are the instrument name so the **colnames()** function can be used to change the column names.


```R
colnames(xtsOHLCData)[[1]] <- "OPEN"
colnames(xtsOHLCData)[[2]] <- "HIGH"
colnames(xtsOHLCData)[[3]] <- "LOW"
colnames(xtsOHLCData)[[4]] <- "CLOSE"
xtsOHLCData
```

#### Use the xts data to plot an OHLC chart

Call the **plot_ly** function to plot an OHLC chart with the index (Date), OPEN, HIGH, LOW, and CLOSE columns.


```R
chart1 <- plot_ly(x = index(xtsOHLCData), 
             type = "ohlc",
             open = coredata(xtsOHLCData)[,'OPEN'],
             high = coredata(xtsOHLCData)[,'HIGH'],
             low = coredata(xtsOHLCData)[,'LOW'],
             close = coredata(xtsOHLCData)[,'CLOSE']) %>% layout(title = "Basic OHLC Chart")
chart1
```

### 3. List request

Datastream also supports Constituent Lists of instruments, e.g. LFTSE100, LS&PCOMP, LDAXINDX, LSTOKYOSE, etc. Only one list is permitted per request. The list instruments can be used with the **listRequest** and **timeSeriesListRequest** methods.


####  Retrieve static information for a given list instrument on a specific date

The following code calls the **listRequest** function with the **LS&PCOMP** which represents S&P 500 to get the name (NAME), mnemonic (MNEM), official closing price (P) and opening price (PO) of instruments in the **LS&PCOMP** on the last trading day. Mnemonic is a unique identification code, assigned by Datastream.

The function returns a data frame with fields in columns and instruments as rows.


```R
listDataFrame <- mydsws$listRequest(instrument = "LS&PCOMP",
                                    datatype = c("NAME","MNEM","P","PO"), 
                                    requestDate=Sys.Date())
listDataFrame
```

####  Retrieve time series data for a given list instrument

The following code calls the **timeSeriesListRequest** function with the **LS&PCOMP** which represents S&P 500 to get the daily official closing price (P) for the last thirty days.

The function returns xts data (eXtensible Time Series) with items' data in columns and data points in rows. If the function shows the **Server error : Internal Server Error : Server error: (500) Internal Server Error**, please set the ChunkLimit to 20 by using the below code:
```
mydsws$chunkLimit <- 20L
```


```R
mydsws$chunkLimit <- 20L
xtsData <- mydsws$timeSeriesListRequest(instrument = "LS&PCOMP",
                       datatype = "P",
                       startDate = "-30D",
                       endDate = "-0D",
                       frequency = "D")

xtsData
```

##### Change the column names

The column names represent the datastream codes for instruments in the list. For readability, the following code changes the column names from the datastream codes to the mnemonic retrieved from the previous step.


```R
for(i in 1:ncol(xtsData)){
    colnames(xtsData)[[i]] <- listDataFrame$MNEM[i]
}
xtsData
```

### 4. Expressions

Expressions are Datastream functions which are statistical and display operators that allow you to calculate and view data in the way you want. Please refer to the [Datastream help page](http://product.datastream.com/Navigator/AdvanceHelpFiles/Functions/WebHelp/HFUNC.htm) for more information. 

#### Use an expression with the snapshotRequest method

The **snapshotRequest** method accepts an expression through the **expression** parameter. 

The following code uses the Annualised growth rate, first and last values expression ([GRFL#](http://product.datastream.com/Navigator/AdvanceHelpFiles/Functions/WebHelp/Annualised_growth_rate_first_and_last_values_GRFL.htm)) to calculate the annualised growth rate between two dates. It calls the **snapshotRequest** method to get the annualised growth rate in the ABF, RIO, and WPP share prices for the previous quarter. 



```R
myData <- mydsws$snapshotRequest(instrument = c("ABF","RIO","WPP"), expression = "GRFL#(XXXX,-1Q)", requestDate = "0D")
myData
```

#### Use an expression with the timeSeriesListRequest method

The following code uses the percentage change expression ([PCH#](http://product.datastream.com/Navigator/AdvanceHelpFiles/Functions/WebHelp/HFUNC.htm#Percentage_change_PCH.htm)) to calculate the percentage change for one day period. It requests the daily historical data for one month period of the ABF, RIO, and WPP instruments.

The function returns xts data (eXtensible Time Series) with items' data in columns and data points in rows.


```R
xtsPCHData <- mydsws$timeSeriesListRequest(instrument = c("ABF","RIO","WPP"),
expression ="PCH#(XXXX,1D)",                 
 startDate = "-1M",
 endDate = "-0D",
 frequency = "D")

xtsPCHData
```

#### Use the xts data to plot a line chart

The xts data contains time series data for three instruments. The following code calls the **plot_ly** function to create a line chart and then plot time series data for each instrument. 


```R
chart2 <- plot_ly()
for(i in 1:ncol(xtsPCHData)){
    chart2 <- add_trace(chart2, 
                        x = index(xtsPCHData), 
                        y = coredata(xtsPCHData)[,i], 
                        name = colnames(xtsPCHData)[[i]], 
                        type = 'scatter', 
                        mode='lines') 
}
chart2
```

### 5. Symbology

The **snapshotRequest** method can also be used to find instrument codes from other codes. 

The following example calls the **snapshotRequest** method to get instrument codes for these instruments:

- **TH:BDMS** (Code - Local)
- **US0231351067** (Code - ISIN)
- **2005973** (Code - SEDOL)
- **894371** (Code - Datastream)

The data types used by the example are:

|Data Type|Name|Description|
|---------|----|-----------|
|RIC|Reuters Instrument Code (RIC)|The Reuters Instrument Code|
|ISIN|Code - Isin|ISIN (International Security Identification Number) is a code that uniquely identifies a security|
|SECD|Code - Sedol|This is an identification code based on the code issued by the London Stock Exchange|
|LOC|Code - Local|This is an identification code based on the official local exchange code|
|DSCD|Code - Datastream|This is the unique six-digit identification code for every stock, allocated by Datastream|

The function returns a data frame with fields in columns and instruments as rows.


```R
symbology <- mydsws$snapshotRequest(instrument = c("TH:BDMS","US0231351067","2005973","894371"),
                                    datatype = c("NAME","RIC","ISIN", "SECD","LOC","DSCD"), 
                                    requestDate=Sys.Date())
symbology
```

### 6. User Statistics

The **snapshotRequest** method can also be used to retrieve monthly usage in terms of data points used per month by using **STATS** as instruments and **DS.USERSTATS** as data type. Only snapshot requests are supported and by default the current month’s usage stats are returned. Previous months’ data can be returned by simply adding a valid start date in request of any previous
month.

The following code requests the user statistics of the current month.


```R
myStat <- mydsws$snapshotRequest(instrument = "STATS", 
                                 datatype = "DS.USERSTATS", 
                                 requestDate = Sys.Date())
myStat
```


```R

```
