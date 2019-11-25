
# Elektron Data Platform Historical R Example

Elektron Data Platform (EDP) is our cloud-enabled, open platform, that brings together content, analytics and proprietary, customer and third-party data distribution and management technology. It provides simple web-based API access to a broad range of content. 

This example demonstrates how to retrieve historical data from the Elektron Data Platform with R on Jupyter Notebook. It uses the [httr](https://cran.r-project.org/web/packages/httr/vignettes/quickstart.html) package to send HTTP request messages and uses [Plotly](https://plot.ly/r/getting-started/) package to draw charts.

## The first step is loading the required libraries

- [httr](https://cran.r-project.org/web/packages/httr/index.html): Tools for working with URLs and HTTP
- [plotly](https://www.rdocumentation.org/packages/plotly/versions/4.9.0): Create interactive web graphics from 'ggplot2' graphs and/or a custom interface to the (MIT-licensed) JavaScript library 'plotly.js'
- [zoo](https://cran.r-project.org/web/packages/zoo/index.html): The zoo package consists of the methods for totally ordered indexed observations. It aims at performing calculations containing irregular time series of numeric vectors, matrices & factors


```R
library(httr)
library(plotly)
library(zoo)  
```

## Next, set URLs for the Elektron Data Platform services

This example uses the following Elektron Data Platform services:

- **/auth/oauth2/v1/token**: This service allows clients to get a token for password grant
- **/data/historical-pricing/v1**: This service is used to retrieve Intraday and Interday time-series historical pricing data




```R
REFINITIV_API_URL <- "https://api.refinitiv.com/"
OAUTH2_TOKEN_URL <- paste(REFINITIV_API_URL, "auth/oauth2/v1/token", sep="")
HISTORICAL_PRICING_URL <- paste(REFINITIV_API_URL, "data/historical-pricing/v1", sep="")
```

## Then, set the Elektron Data Platform credential

The Elektron Data Platform credential include:

- **USERNAME**: The resource owner username (typically email)
- **PASSWORD**: The resource owner password
- **CLIENTID**: (Aka Application ID, aka AppKey.) A unique ID defined for an application making the request. Users can generate/manage their application ID's [here](https://emea1.apps.cp.thomsonreuters.com/apps/AppkeyGenerator)




```R
USERNAME <- "<user name>"
PASSWORD <- "<password>"
CLIENTID <- "<client id>"
```

Instead of setting the Elektron Data Platform credential in the code, you can set credential information in the **.Renviron** file. Then, use the **Sys.getenv** method to retrieve the credential information. 

For example, the **.Renviron** file contains the following environment variables:

```
USERNAME=<user name>
PASSWORD=<password>
CLIENTID=<client id>
```
Then, use the following code to set variables.

```
Credentials <-Sys.getenv(c("USERNAME", "PASSWORD", "CLIENTID"))
USERNAME <- Credentials["USERNAME"]
PASSWORD <- Credentials["PASSWORD"]
CLIENTID <- Credentials["CLIENTID"]
```

If the HTTP requests must be sent through a proxy, the following code can be used to set a proxy.

```
set_config(use_proxy(url="<url>",port=<port>))
```

Now, it is ready to use the Elektron Data Platform services. Next, we will show how to retrieve historical data from the Elektron Data Platform services.


## Retrieve Intraday and Interday time-series historical pricing data

The following section will show how to use the **auth/oauth2** and **data/historical-pricing** services.

## 1. Login

Login is the first step for all EDP API requests. 

### Construct the body of the HTTP request message

To login, the HTTP post message must be sent to the **auth/oauth2/v1/token** service. The body of the HTTP request message contains the following parameters.

- **grant_type**: Supported values "password" and "refresh_token"
- **username**: The resource owner username (typically email)
- **password**: The resource owner password
- **scope**: The scope of the access request
- **takeExclusiveSignOnControl**: OPTIONAL. This is a Boolean that will allow the API Caller to create a session if the number of concurrent sessions has been reached
- **client_id**: A unique ID defined for an application making the request. Users can generate/manage their application ID's [here](https://emea1.apps.cp.thomsonreuters.com/apps/AppkeyGenerator)

For other parameters, please refer to the [API Documents](http://api.refinitiv.com/).

The following code creates a list that contains those parameter names and values.


```R
RequestTokenBody <- list(grant_type="password",
          username=USERNAME,
          password=PASSWORD,
          scope="trapi",
          takeExclusiveSignOnControl= "True",
          client_id=CLIENTID
          )
```

### Send the HTTP post message to retrieve tokens

The following code calls the **httr::post** method to send the HTTP post message which contains the login information in the message's body to the **auth/oauth2/v1/token** service. The login information is encoded with a URL encoded form (application/x-www-form-urlencoded) in the message's body. The HTTP post message also contains the **Authorization** HTTP header with the client id as its value. 

After receiving the response, it calls the **stop_for_status** function which converts HTTP errors to R errors or warnings. If the request was successful, it prints the response on the screen. Otherwise, it will stop.


```R
RequestTokenResponse <- httr::POST(OAUTH2_TOKEN_URL,
                 add_headers(Authorization = CLIENTID),                
                 body = RequestTokenBody,
                 encode = "form")
stop_for_status(RequestTokenResponse)
RequestTokenResponse
```

If the response status is not 200, please verify the URL or login information.

### Get an access token

If the response status is 200, the HTTP response will contain an access token. This access token will be used in subsequent requests. 

The data is in JSON format so it calls the **httr::content** method to extract the JSON content from the HTTP response. Then, it saves an access token into the **ACCESS_TOKEN** variable so it can be used in the subsequent requests.


```R
TokenContent <- httr::content(RequestTokenResponse, "parsed", "application/json", encoding="UTF-8")
ACCESS_TOKEN <- TokenContent$access_token
ACCESS_TOKEN
```

After getting the access token, we can use this token to call other Elektron Data Platform services.

## 2. Retrieve Interday Historical Pricing Summaries Data


The historical pricing service can be used to retrieve Interday historical pricing summaries data through the **/views/interday-summaries** URL. 

### Send a request


Interday Historical Pricing Summaries Data uses the HTTP GET request. The HTTP message has the **Authorization** HTTP header with the token type (bearer) and access token as its value. The requested instrument must be added at the end of the URL and other request parameters are in the query string.


The following code calls the **httr::GET** method to request the daily historical pricing data (OPEN, HIGH, LOW, and CLOSE) of IBM.N for one year period with the following parameters:

- **start**: The start date and timestamp of the query is in ISO8601 with UTC only e.g 2018-12-24T09:00:00.000000000Z
- **end**: The end date and timestamp of the query is in ISO8601 with UTC only e.g 2018-12-24T09:00:00.000000000Z
- **fields**: The comma-separated list of fields that are to be returned in the response
- **interval**: The consolidation interval in ISO8601. The support intervals are PT1M, PT5M, PT10M, PT30M, PT60M, and PT1H

The request parameters are created by using a list. For other parameters, please refer to the [API Documents](http://api.refinitiv.com/).

After receiving the response, it calls the **stop_for_status** to verify the error. If the request was successful, it prints the response on the screen. Otherwise, it will stop.


```R
INTERDAY_SUMMARY_URL <- paste(HISTORICAL_PRICING_URL, "/views/interday-summaries/", sep="")

HistoricalResponse <- httr::GET(paste(INTERDAY_SUMMARY_URL, "IBM.N", sep=""),
                                add_headers(Authorization = paste(TokenContent[["token_type"]], ACCESS_TOKEN)),
                                query=list(start=Sys.Date()-365,
                                           end=Sys.Date(),
                                           fields="OPEN_PRC,HIGH_1,LOW_1,TRDPRC_1",
                                           interval="P1D")
                 )  
stop_for_status(HistoricalResponse)
HistoricalResponse   

```

If the response status is 401, the access token may be expired. Please re-run the first step (**1. Login**) to get a new access token.


### Extract the JSON content

If the response status is 200, the HTTP response will contain the daily historical data in JSON tabular format.

The following code calls the **httr::content** method to extract the JSON content from the HTTP response and then print it on the screen.


```R
HistoricalContent <- httr::content(HistoricalResponse, "parsed", "application/json", encoding="UTF-8")
HistoricalContent
```

### Create a data frame from the JSON content

With R, it is easier to display or manipulate the data in the data frame. 

The following code creates a function called **JsonTabularToDataFrame** which accepts the JSON tabular content and returns a data frame. Next, it calls this function with the JSON content returned by the previous step in order to create the data frame. Then, it prints the data frame on the screen.


```R
JsonTabularToDataFrame <- function(jsonObj){
    temp_df <- NULL

    for(i in c(1:length(jsonObj[[1]]$headers))){
        tempData <- c()
        for(j in c(1:length(jsonObj[[1]]$data))){   
            tempData <- c(tempData,jsonObj[[1]]$data[[j]][[i]])       
        }
        if (is.null(temp_df)){
            temp_df = data.frame(tempColName = tempData)
            names(temp_df)[names(temp_df) == "tempColName"] <- jsonObj[[1]]$headers[[i]]$name
        }else{
            temp_df[jsonObj[[1]]$headers[[i]]$name] <- tempData
        }
    }
    return(temp_df)
}

df1 <- JsonTabularToDataFrame(HistoricalContent)
df1
```

### Calculate the simple moving average

This section uses the last prices (TRDPRC_1) in the data frame to calculate the 30-day simple moving average. It uses the **rollmean** function in the **[zoo](https://cran.r-project.org/web/packages/zoo/index.html)** package. The **rollmean** is a generic function for computing rolling means of ordered observations. 

Then, the 30-day simple moving average values are added into the data frame as a new column (**sma30**).


```R
df1 <- df1 %>% 
mutate(sma30 = rollmean(TRDPRC_1, k=30, fill=NA, align="left"))
df1
```

### Plot a chart

The following code calls the **plot_ly** function to plot a candlestick chart for the DATE, OPEN_PRC, HIGH_1, LOW_1, and TRDPRC_1 columns. It also plots a line chart for the **sma30** column.


```R
chart1 <- df1 %>%
  plot_ly(x = ~DATE, type="candlestick",
          open = ~OPEN_PRC, close = ~TRDPRC_1,
          high = ~HIGH_1, low = ~LOW_1, name="OHLC") %>%
  add_lines(x = ~DATE, y = ~sma30, line = list(color = 'black', width = 0.75), inherit = F, name="SMA30D") %>%
  layout(title = "Basic Candlestick Chart")
chart1

```

## 2. Retrieve the Time & Sales data

The historical pricing service can be used to retrieve time series pricing events data (i.e. trades, quotes, and corrections) through the **/views/events** URL. 

### Send a request

Time series pricing events use the HTTP GET request. The HTTP message has the **Authorization** HTTP header with the token type (bearer) and access token as its value. The requested instrument must be added at the end of the URL and other request parameters are in the query string.


The following code calls the **httr::GET** method to request the Times & Sales data of IBM.N with the following parameters:

- **eventTypes**: The list of market events (comma delimited) supports the values of trade, quote, and correction for event types
- **end**: The end date and timestamp of the query is in ISO8601 with UTC only e.g 2018-12-24T09:00:00.000000000Z
- **fields**: The comma-separated list of fields that are to be returned in the response
- **count**: The maximum number of data returned

The request parameters are created by using a list. For other parameters, please refer to the [API Documents](http://api.refinitiv.com/).

After receiving the response, it calls the **stop_for_status** to verify the error. If the request was successful, it prints the response on the screen. Otherwise, it will stop.


```R
PRICING_EVENTS_URL <- paste(HISTORICAL_PRICING_URL, "/views/events/", sep="")

EventsResponse <- httr::GET(paste(PRICING_EVENTS_URL, "IBM.N", sep=""),
                                add_headers(Authorization = paste(TokenContent[["token_type"]], ACCESS_TOKEN)),
                                query=list(eventTypes="trade",
                                           fields="TRDPRC_1",
                                           count=100)
                 )  
stop_for_status(EventsResponse)
EventsResponse   

```

If the response stat is 401, the access token may be expired. Please re-run the first step (**1. Login**) to get a new access token.

### Extract the JSON content

If the response status is 200, the HTTP response will contain the Times & Sales data in JSON tabular format.

The following code calls the **httr::content** method to extract the JSON content from the HTTP response and then print it on the screen.


```R
EventsPricingContent <- httr::content(EventsResponse, "parsed", "application/json", encoding="UTF-8")
EventsPricingContent
```

### Create a data frame from the JSON content

With R, it is easier to display or manipulate the data in the data frame. 

The following code calls the **JsonTabularToDataFrame** method to create a data frame from the retrieved JSON tabular data. Then, it prints the data frame on the screen.


```R
df2 <- JsonTabularToDataFrame(EventsPricingContent)
df2
```

### Plot a chart

The following code calls the **plot_ly** function to plot a line chart from the Time & Sales data.


```R
chart2 <- df2 %>%
  plot_ly(x = ~DATE_TIME, type="scatter",
          mode="lines",
          y = ~TRDPRC_1,
          name="Trade")  %>%
  layout(title = "Trade")
chart2

```

The historical pricing service can also be used to retrieve Intraday historical pricing summaries data and a single record of time series pricing events. Please refer to the [API Documents](http://api.refinitiv.com/) for more information.
