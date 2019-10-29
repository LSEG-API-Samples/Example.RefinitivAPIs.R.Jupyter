
# Eikon Data API R Example

This example demonstrates how to use [Eikon Data API](https://developers.refinitiv.com/eikon-apis/eikon-data-api) with R on Jupyter Notebook. It uses the [eikonapir](https://github.com/ahmedmohamedali/eikonapir) package to retrieve data from Eikon and uses [Plotly](https://plot.ly/r/getting-started/) package to draw charts. It also uses the [IRDisplay](https://www.rdocumentation.org/packages/IRdisplay) package to display news in HTML format.

To setup Jupyter Notebook environment for R or install Eikon Data API for R, please refer to this [article](https://developers.refinitiv.com/article/setup-jupyter-notebook-r).

### The first step is loading the **eikonapir**, **plotly**, and **IRDisplay** packages.


```R
library(eikonapir)
library(plotly)
library(IRdisplay)
```

### Next, call the **set_app_id** method with the application key. 

To create an application key, please refer to [Eikon Data API quick start guide](https://developers.refinitiv.com/eikon-apis/eikon-data-apis/quick-start).


```R
set_app_id('your application key')
```

## 1. Use the **get_data** function to retrieve the latest data

The following code calls the **get_data** function to retrieve the latest available close price, the volume of the latest trading day, and the low price for the latest trading day fields of IBM, GOOG.O, and MSFT.O instruments.

The function returns a data frame with fields in columns and instruments as rows.


```R
data_frame1 <- get_data(list("IBM", "GOOG.O", "MSFT.O"), list("TR.PriceClose", "TR.Volume", "TR.PriceLow"))
data_frame1
```

## 2. Use the **get_data** function to retrieve the historical data and plot an OHLC chart

The following code calls the **get_data** function to retrieve daily historical OPEN, HIGH, LOW, CLOSE fields from one year ago to the last trading day of IBM. 

The function returns a data frame with fields in columns and data points in rows.


```R
data_frame2 <- get_data("IBM", 
                        list("TR.OPENPRICE.Date","TR.OPENPRICE","TR.HIGHPRICE","TR.LOWPRICE","TR.CLOSEPRICE"),
                        list("Frq"="D","SDate"="0D","EDate"="-1AY"))
data_frame2
```

### Modify the data frame by converting the values in the Date column to date. 

To create a chart, we need to convert the values in the Date column to date by using the **mutate** function.



```R
data_frame2  <- data_frame2 %>%
    mutate(Date=as.Date(Date, format="%Y-%m-%d"))
data_frame2
```

### Use the data in the data frame to create an OHLC chart.

It calls the **plot_ly** function to create an OHLC chart with the **Date**, **Open Price**, **Close Price**, **High Price**, and **Low Price** columns.


```R
OHLCChart1 <- data_frame2 %>%
  plot_ly(x = ~Date, type="ohlc",
          open = ~`Open Price`, close = ~`Close Price`,
          high = ~`High Price`, low = ~`Low Price`) %>%
  layout(title = "Basic OHLC Chart")
```

### Display the OHLC chart


```R
OHLCChart1
```

## 3. Use the **get_timeseries** method to retrieve daily historical data

The following code calls the **get_timeseries** method to retrieve daily historical data of GOOG.O from 01 Jan 2019 to 30 Sep 2019.


```R
data_frame3 = get_timeseries(list("GOOG.O"),list("*"),"2019-01-01T00:00:00","2019-09-30T00:00:00","daily")
data_frame3
```

### Modify the data frame

In order to create a chart, the returned data frame will be modified:
- Changing the last column name from NA to RIC
- Converting the values in the TIMESTAMP column to date


```R
colnames(data_frame3)[[8]] = "RIC"
data_frame3  <- data_frame3 %>%
    mutate(TIMESTAMP=as.Date(TIMESTAMP, format="%Y-%m-%d"))
data_frame3
```

### Use the data in the data frame to create a candlestick chart

It calls the **plot_ly** function to create a candlestick chart with the **TIMESTAMP**, **OPEN**, **CLOSE**, **HIGH**, and **LOW** columns.


```R
CandleStickChart <- data_frame3 %>%
  plot_ly(x = ~TIMESTAMP, type="candlestick",
          open = ~OPEN, close = ~CLOSE,
          high = ~HIGH, low = ~LOW) %>%
  layout(title = "Basic Candlestick Chart")
```

### Display the candlestick chart


```R
CandleStickChart
```

## 4. Use the get_symbology function to convert instrument codes

The following code calls the **get_symbology** method to convert RICS names to ISIN instrument names.


```R
ISINList <- get_symbology(list("MSFT.O", "GOOG.O", "IBM.N"),
                          from_symbol_type="RIC", 
                          to_symbol_type="ISIN")
ISINList
```

The following code calls the **get_symbology** method to convert ISIN instrument names to CUSIP instrument names.


```R
CUSIPList <- get_symbology(list("US5949181045", "US02079K1079", "US4592001014"), 
                           from_symbol_type="ISIN", 
                           to_symbol_type="CUSIP")
CUSIPList
```

## 5. Use the get_news_headlines and get_news_story method to retrieve news

The following code calls the **get_news_headlines** method to retrieve 10 news headlines about IBM.N in English. For each headline, it calls the **get_news_story** with the story ID to retrieve a news story. The news headlines and stories are displayed as HTML.



```R
headlines <- get_news_headlines("R:IBM.N IN ENGLISH")
for (row in 1:nrow(headlines)) 
{   
    display_html(paste("<h1>",headlines[row,"text"],"</h1>"))
    story <- get_news_story(headlines[row, "storyId"])
    display_html(story)
}

```


```R

```
