```{r setup, include=FALSE}
chooseCRANmirror(graphics=FALSE, ind=1)
knitr::opts_chunk$set(echo = TRUE)
```

# Time Series Analysis with R
## Introduction to Facebook's Prophet (Oct 2019)
author: "Kerstin Poetzl"
Hochschule der Medien Stuttgart
2019/10/23


# References:

* [Facebook Prophet documentation](https://facebook.github.io/prophet/docs/quick_start.html#r-api) <br>
--> referenced as [FB]<br>

* Taylor SJ, Letham B. (2017) [Forecasting at scale. PeerJ Preprints 5:e3190v2](https://doi.org/10.7287/peerj.preprints.3190v2) <br>
--> referenced as [Taylor/Letham 1]<br>

* Taylor SJ, Letham B. (2017) [Prophet: forecasting at scale](https://research.fb.com/prophet-forecasting-at-scale/) <br>
--> referenced as [Taylor/Letham 2]<br>

* Ruan van der Merwe (2018) [Implementing Facebook Prophet efficiently](https://towardsdatascience.com/implementing-facebook-prophet-efficiently-c241305405a3)<br>
--> referenced as [Merwe]<br>

* Tutorial [Time Series ARIMA and Prophet](https://www.kirenz.de/project/python-time-series/), from Prof. Dr. Jan Kirenz, Hochschulde der Medien, [Homepage](https://www.kirenz.de/) <br>
--> referenced as [Kirenz]

* Dmitry Storozhenko (2018) [Surviving financial turmoil: time series forecasting with Prophet and R](https://medium.com/@storozhenko.dmitry/surviving-financial-turmoil-time-series-forecasting-with-prophet-and-r-105f4f4f0766)<br>
--> referenced as [Storozhenko]<br>

* [Peyton Manning at Wikipedia](https://de.wikipedia.org/wiki/Peyton_Manning)<br>
--> [Manning@Wiki]

* Automatic Forecasting Procedure (2019) [Package ‘prophet’](https://cran.r-project.org/web/packages/prophet/prophet.pdf)<br>
--> [Package ‘prophet’]


# Installed Packages and Libraries

```{r eval=FALSE, message=FALSE, warning=FALSE, results='hide'}
install.packages("ggplot2", repos = "http://cran.us.r-project.org")
install.packages("forecast", repos = "http://cran.us.r-project.org")
install.packages("plotly")
install.packages("prophet", type="source")
install.packages("DT")
```

```{r, message=FALSE, warning=FALSE, results='hide'}
library(ggplot2)
library(forecast)
library(plotly)
library(prophet)
library(DT)
```


# Motivation for this Tutorial

Business Forecasting is a central data science task and has to consider many activities within an company. A Forecast is a complex problem for machines and most analysts. Typical business examples where a forecast is needed are effective capacity planning, goal setting which needs a baseline for performance measurement and anomaly detection, etc. The Business Analysts have ususally a deep domain expertise but hardly knowledge of time series forecasting techniques which is a challenge in order to produce a high-quality forecast. [Taylor/Letham 1, page 1]

This lack of knowledge of many Business Analysts is the major motivation of this tutorial. The Tutorial shall provide useful guidance how to produce in a simply way a time series forecast of good quality.


# Introduction

"Prophet is a recently from Facebook developed procedure for forecasting time series data based on an additive model where non-linear trends are fit with yearly, weekly, and daily seasonality, plus holiday effects.

According to Facebook, it works best with time series that have strong seasonal effects and several seasons of historical data. Prophet is robust to missing data and shifts in the trend, and typically handles outliers well.

Prophet is open source software released by Facebook’s Core Data Science team." [Kirenz]

How To forecast with Prophet:

In R, we use the normal model fitting API. We provide a prophet function that performs fitting and returns a model object. You can then call predict and plot on this model object." [FB]


# Import data

First step of this tuturial is to read in the data and to create the outcome variable. 
As data set we use the wikipedia log number of views of Peypton Manning who is an ex US American Football Player.

Source:<br>
- [Peyton Manning at Wikipedia](https://de.wikipedia.org/wiki/Peyton_Manning)<br>
- [Data from github](https://github.com/facebook/prophet/blob/master/examples/example_wp_log_peyton_manning.csv)

Peyton Williams Manning (born March 24, 1976 in New Orleans, Louisiana) is a former American football player on the quarterback's position. He played 14 years for the Indianapolis Colts in the National Football League (NFL) and won with them the Super Bowl XLI. He then played four years for the Denver Broncos, with whom he won the 2016 Super Bowl 50 in his last season. [Manning@Wiki]

Apply the function `read.table()` with the option to choose the file directly from the local directory, header is True and the separator is in our data file a comma.

```{r}
data <- read.table(file.choose(), header = T, sep = ",") # header is False if no header
```


# Data Examination

Apply the function `View()` to visualize the data in a separate table tab and `dim()` to understand how many observations the dataframe contains:

```{r}
View(data)
dim(data)
```

<img src="C:/Users/kpoetzl/Documents/R/R Online Phase/Assignments/SourcePictures/View.png" alt="">

The dataframe contains 2 columns ds and y, which contain the date and the associated numeric value. There are 2905 oberserations in Total. 

Table format looks ok, data is properly separated and the table has a valid header.

Apply the function `str()` to get a brief glimpse on the data:

```{r}
str(data)
```

The date column "ds" is currently stored as a "Factor" data type.

**Prerequisition for Prophet:**

"Prophet needs as input a dataframe with columns for date (ds) and the associated numeric values (y). The date format needs to be YYYY-MM-DD for a date or YYYY-MM-DD HH:MM:SS for a timestamp." [FB]

To ensure to have the proper date format and also for better visualization purposes this should be converted to a "date-time" class which can be displayed as a continuous variable.

Apply the function `as.Date()` on the column "ds" which holds the date information and replace the old values: 

```{r}
data$ds <- as.Date(data$ds, format = "%Y-%m-%d")
str(data)
```

The data type has been successfully changed from "Factor" to "Date".

Apply the function `head()` to retrieve only the first 6 values from the data set to check the data: 

```{r}
head(data)
```

or a certain column:

```{r}
head(data$ds)
```

For the last few rows use `tail()` instead.

There are more alternative ways to show the data.
Call the name of the dataset to retrieve all values or only a certain column.
Alternatively `print()` can be used as well.

```{r}
#data 
#data$ds
#print(data)
```


Another option is to generate a table as output, where you can navigate through the rows.
Apply the function `datatable()` to check the values of the data:

```{r}
DT::datatable(data, options = list(pageLength = 4))
```

Apply the function `summary()` to retrieve a brief statistical summary of the data for Min, Max, Mean, 1st and 3rd Quartile on the dataframe:

```{r}
summary(data)
```

In case that the Minimum "Min" would be zero, it has to be decided if this needs to be cleaned up or not.

Although Facebooks Prophet is robust against missing values, other forecast models like ARIMA would require that missing data need to be handled to have the right forecast. For demonstration purposes we assign NA to all data points where the observation in the dataframe is zero:

```{r}
data$y[data$y==0] <- NA
```

# Data Visualization

It is a good practice to plot the data and understand how they look like before fitting any models.
Apply the function `ggplot()` here with a selection of simple attributes to define the layout of the scatter plot:

```{r}
plot <- ggplot(data, aes(x= ds, y= y))+
        geom_point(           # used to create scatterplots
        color= "black",     # color of the data points
        shape = 1,          # type of data point (circle, square, filled vs open, etc)
        alpha = 0.4,        # defines the color transparency
        size = 0.6)         # defines the size of the data point
      
```

Some sources for how to change the aestethic:<br>
- [Cheatsheet](https://rstudio.com/wp-content/uploads/2015/03/ggplot2-cheatsheet.pdf)<br>
- [Grafiken mit ggplot aus dem package library(ggplot2)](http://md.psych.bio.uni-goettingen.de/mv/unit/ggplot2/ggplot2.html)<br>
- [ggplot2 part of tidyverse](https://ggplot2.tidyverse.org/reference/)<br>
- [Aesthetic specifications](https://ggplot2.tidyverse.org/articles/ggplot2-specs.html)<br>

Apply the function `ggplotly()` to create a plotly object, adjust the size and make the plot interactive, see also [ggplotly](https://www.rdocumentation.org/packages/plotly/versions/4.9.0/topics/ggplotly):

```{r}
ggplotly(plot, width = 700, height = 400)
```

In case the data set would have several columns and we would like to reduce the data set only to the needed columns, just create a new data frame with the needed values for date and observation.

Not needed here for the excercise but for demonstration purposes done:

Store the values for "date" in the variable "ds" for data stamp information:

```{r}
ds <- data$ds
```


**Logarithmic scales**

If the observations have a wide spread over several orders of magnitude, with some observations having a much higher value than others, it's a practical approach to convert the axis on a log scale. See also [wikipedia for logarithmic scales](https://en.wikipedia.org/wiki/Logarithmic_scale)

Store the values for "observations" in the variable "y" and use the `log()` function to convert those to a log scale:

```{r}
y <- log10(data$y)
```

Save the 2 variables as new dataframe "df"

```{r}
df <- data.frame(ds,y)
```

Print again the scatter plot for the dataframe "df". 

```{r}
plot <- ggplot(df, aes(x= ds, y= y))+
      geom_point(
      color= "black",
      shape = 1,
      alpha = 0.4, 
      size = 0.6
      ) 
ggplotly(plot, width = 700, height = 400)
```

If the initial data had a wide spread on the observations the seasonality would be now much more visible. Our data didn't had such a wide spread, therefore the plots are more or less identically.

As there seems to be no difference we continue with the historical dataframe "data" with initial decimal scale.


# Prophet FCST - fit the Model

Basically Prophet provides a very simple interface: with a column of dates and a column of associated numbers it produces a forecast for the time series.

First step is to fit a model on the historical data. It can be done by calling the `prophet()` function using the dataframe as the input. [FB]

Store the model for the dataframe "data" in "m" by applying the `prophet()` function:

```{r}
m <- prophet(data)
```

Have a look into the model and its details:

```{r}
m
```


# Prophet FCST - create future dates

Prophet provides a built-in helper function `make_future_dataframe` which easily creates a dataframe with the future dates. 
The `make_future_dataframe` function requires only the input of frequency and number of future periods to be used. 
The historical data is kept by default for evaluation purposes later.[FB]

Store the future dates in "future". 
Keep frequency set to days as default. Set the period argument to 365 as forecast into the future.

Check start and end data by applying the `head()` and `tail()`function:

```{r}
future <- make_future_dataframe(m, periods = 365)
head(future)
tail(future)
```

The data starts with the historical data and ends with the additional 365th value.

# Prophet FCST - create the prediction

Next step deploys the model to the future dates. Use the `predict()` function to create the predictions for all rows (historic and future) in the "future" dataframe. The result is a forecast object as dataframe which provides a column "yhat" containing the forecast. There are additional columns available to represent uncertainty intervals and seasonal components. [FB]

The function takes the data and trains the model on a specified period. Next It will predict your specified future period. Further Prophet will retrain the data on a bigger period, predict again which will repeat until the end point is reached. [Merwe]

Create a new dataframe assigned to the "forecast" variable. Apply `predict()` on the prophet model object to generate the forecast:

```{r}
forecast <- predict(m, future)
```

Show first and last 2 items of the prediction (yhat) with lower (yhat_lower) and upper (yhat_upper) limits. Recognize that the generated data start with the historic data and ends with the forecast data:

```{r}
head(forecast[c("ds", "yhat", "yhat_lower", "yhat_upper")])
```


```{r}
tail(forecast[c("ds", "yhat", "yhat_lower", "yhat_upper")])
```


# Prophet FCST - Plot Forecast

To visualize the forecast Prophet provides another built-in helper function `plot()` which only needs the model and the forecast as input [FB]:

- "m" = prophet object from `prophet(data)`<br>
- "forecast" = dataframe of `predict(m, future)` function<br>
- `add_changepoints_to_plot()` to visualize the same [Package ‘prophet’]:

Legend for the plot:

- **black dots** represent the number of views of Manning's Wikipedia page until end of 2016
- **blue line** represents the prediction of the prophet model including the historic values
- **light blue lines** visualizes the y upper and lower limits presenting the uncertainty interval

```{r}
p <- plot(m, forecast) + add_changepoints_to_plot(m)
plot(p, size=3)
```

To be recognized: The Prophet forecast automatically detects the saisonality, here related to NFL seasons. 

Alternative: To generate an interactive plot ggplotly() can be easily applied [Storozhenko]:

```{r, fig.height=5, fig.width=9, warning=FALSE}
p <- plot(m, forecast) + add_changepoints_to_plot(m)
p_int <- ggplotly(p, tooltip="all", orignalData = True)
p_int
```


To visualize the forecast components Prophet provides one more built-in helper function which is `prophet_plot_components()`.
The plot provides the forecast broken down into "trend", "weekly" and "yearly" seasonality. [FB]

This plot visualizes how Prophet has modeled the trend and seasonality for the data:

```{r}
prophet_plot_components(m, forecast)
```

Interpretation of the plot:

As Peyton Manning is an American football player, yearly seasonality is important, but also weekly seasonality is present. Further certain football events (like playoff games) can also be modeled. [Taylor/Letham 2]

- **Trend**: is showing the trend and a confidence intervall getting bigger at the end for the predition. Highest peek was in 2012
- **Weekly seasonality**: highest for Monday, lowest for Saturday. 
- **Yearly seasonality**: highest in beginning of the Year followed by another peak in September. Which fits to the regular football season starting in September followed by the Play-offs, Super Bowl and Pro Bowl until February. The downward trend after 2014 corresponds to the retirement of Manning in 2016.

Another way to create an interactive plot of the forecast is using Dygraphs command `dyplot.prophet(m, forecast)`. [FB]

- "m" = prophet object from `prophet(data)`<br>
- "forecast" = dataframe of `predict(m, future)` function<br>

```{r, dpi=65, fig.width=14}
dyplot.prophet(m, forecast)
```

# Fazit

"Prophet makes it easy for experts and non-experts to make a high quality forecasts.
But Prophet is not for all forecasting problems the method to be used.
Prophet is optimized for business forecast tasks with following characteristics:

- hourly, daily, or weekly data 
- minimum of a few months (better year) of historical data
- strong seasonalities: day of week and time of year
- important holidays / events known in advance (e.g. the Super Bowl)
- a reasonable number of missing observations
- a reasonable number of large outliers
- historical trend changes (e.g. product launches, logging changes)
- non-linear growth trends
- trend has a natural limit or saturates"

[Taylor/Letham 2]

"We have found Prophet’s default settings to produce forecasts that are often accurate as those produced by skilled forecasters, with much less effort. With Prophet, you are not stuck with the results of a completely automatic procedure if the forecast is not satisfactory — an analyst with no training in time series methods can improve or tweak forecasts using a variety of easily-interpretable parameters. We have found that by combining automatic forecasting with analyst-in-the-loop forecasts for special cases, it is possible to cover a wide variety of business use-cases." [Taylor/Letham 2]

"More details about the options available for each method are available in the docstrings, for example, via ?prophet or ?fit.prophet. This documentation is also available in the [reference](https://cran.r-project.org/web/packages/prophet/prophet.pdf) manual on CRAN." [FB]

---------------




