---
title: "Vignette - transportation safety data"
author:
  - Miao Cai^[Department of Epidemiology and Biostatistics, Saint Louis University. Email address [miao.cai@slu.edu](miao.cai@slu.edu)]
  - Amir Mehdizadeh^[Department of Industrial & Systems Engineering, Auburn University]
  - Fadel M. Megahed^[Farmer School of Business, Miami University.  This author can be reached by email at [fmegahed@miamioh.edu](mailto:fmegahed@miamioh.edu).]
  
date: "2018-09-06"
output:
  html_document:
    toc: true
    toc_float: true
    number_sections: true
    df_print: paged
    code_folding: hide
---



The first step to make accurate crash prediction is obstaining high quality data. This vignette is a reproducible demonstration on extracting online transportation safety data.




# Weather data

In this part, we show how to get both historical and real-time weather data using [DarkSky API](https://darksky.net/dev/docs/libraries). It can be used in both [Python](https://github.com/bitpixdigital/forecastiopy3) and [R](https://github.com/hrbrmstr/darksky). Before using the DarkSky API to get weather data, you need to register for a API key on [its official website](https://darksky.net/dev/register). The first 1000 API requests you make each day are free, but each API request over the 1000 daily limit will cost you $0.0001, which means a million extra API requests will cost you 100 USD. 

The DarkSky API provides the following meteoreological variables:

- Apparent (feels-like) temperature
- Atmospheric pressure
- Cloud cover
- Dew point
- Humidity
- Liquid precipitation rate
- Moon phase
- Nearest storm distance
- Nearest storm direction
- Ozone
- Precipitation type
- Snowfall
- Sun rise/set
- Temperature
- Text summaries
- UV index
- Wind gust
- Wind speed
- Wind direction


The data provided by darksky API includes 3 parts: 

- hourly weather data. 24 hourly observations for each 15 weather variables in that day.
- daily weather data. 1 observations for each 34 weather variables in that day.
- current weather data. 1 observations for each 15 weather variables at the assigned time point.

In this part of the vignette, we will show the readers how to get historical and real-time weather data for a sample of 20 observations. The orginal data include only three variables: longitude, latitude, and the local time, which are the required variables to obtain weather data from DarkSky API. Below is how the orginal data look like:


```r
gps_sample = 
  structure(list(
    from_lat = c(41.3473127, 41.8189037, 32.8258477, 
40.6776808, 40.2366043, 41.3945561, 32.6320605, 40.5413856, 33.6287422, 
40.0692742, 41.347986, 37.7781459, 43.0843081, 41.48026, 43.495149, 
41.5228684, 41.5763081, 47.6728665, 41.0918361, 41.1537819),
    from_lon = c(-74.2850908, -73.0835104, -97.0306677, -75.1450753, 
    -76.9367494, -72.8589916, -96.8538145, -74.8547061, -113.7671634, 
    -76.762612, -74.284785, -77.4615586, -76.0977384, -73.2107541, 
    -73.7727896, -74.0739204, -88.1529175, -117.3224667, -74.1554972, 
    -74.1887031), 
  beg_time = structure(c(1453101738, 1437508088, 
    1436195038, 1435243088, 1454270680, 1432210106, 1438937772, 
    1446486480, 1450191622, 1449848630, 1457597084, 1432870446, 
    1457968284, 1451298724, 1431503502, 1443416864, 1438306368, 
    1445540454, 1452619392, 1436091072), class = c("POSIXct", 
    "POSIXt"), tzone = "UTC")), .Names = c("from_lat", "from_lon", 
"beg_time"), row.names = c(NA, 20L), class = c("tbl_df", "tbl", 
"data.frame"))

gps_sample
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["from_lat"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["from_lon"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["beg_time"],"name":[3],"type":["S3: POSIXct"],"align":["right"]}],"data":[{"1":"41.34731","2":"-74.28509","3":"2016-01-18 07:22:18","_rn_":"1"},{"1":"41.81890","2":"-73.08351","3":"2015-07-21 19:48:08","_rn_":"2"},{"1":"32.82585","2":"-97.03067","3":"2015-07-06 15:03:58","_rn_":"3"},{"1":"40.67768","2":"-75.14508","3":"2015-06-25 14:38:08","_rn_":"4"},{"1":"40.23660","2":"-76.93675","3":"2016-01-31 20:04:40","_rn_":"5"},{"1":"41.39456","2":"-72.85899","3":"2015-05-21 12:08:26","_rn_":"6"},{"1":"32.63206","2":"-96.85381","3":"2015-08-07 08:56:12","_rn_":"7"},{"1":"40.54139","2":"-74.85471","3":"2015-11-02 17:48:00","_rn_":"8"},{"1":"33.62874","2":"-113.76716","3":"2015-12-15 15:00:22","_rn_":"9"},{"1":"40.06927","2":"-76.76261","3":"2015-12-11 15:43:50","_rn_":"10"},{"1":"41.34799","2":"-74.28478","3":"2016-03-10 08:04:44","_rn_":"11"},{"1":"37.77815","2":"-77.46156","3":"2015-05-29 03:34:06","_rn_":"12"},{"1":"43.08431","2":"-76.09774","3":"2016-03-14 15:11:24","_rn_":"13"},{"1":"41.48026","2":"-73.21075","3":"2015-12-28 10:32:04","_rn_":"14"},{"1":"43.49515","2":"-73.77279","3":"2015-05-13 07:51:42","_rn_":"15"},{"1":"41.52287","2":"-74.07392","3":"2015-09-28 05:07:44","_rn_":"16"},{"1":"41.57631","2":"-88.15292","3":"2015-07-31 01:32:48","_rn_":"17"},{"1":"47.67287","2":"-117.32247","3":"2015-10-22 19:00:54","_rn_":"18"},{"1":"41.09184","2":"-74.15550","3":"2016-01-12 17:23:12","_rn_":"19"},{"1":"41.15378","2":"-74.18870","3":"2015-07-05 10:11:12","_rn_":"20"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>



```r
library(darksky)
library(tidyverse)

Sys.setenv(DARKSKY_API_KEY = "9f219edf4689a0f26a83aa4d9a46f25a")

t = get_forecast_for(38.642105, -90.244440, Sys.time())
```


## Historical data (daily)

### Historical daily data

The following 39 meteoreological variables can be obtained from DarkSky API.

- time            
- summary                
- icon          
- sunriseTime      
- sunsetTime             
- moonPhase                  
- precipIntensity           
- precipIntensityMax       
- precipIntensityMaxTime    
- precipProbability        
- precipType                 
- temperatureHigh        
- temperatureHighTime      
- temperatureLow            
- temperatureLowTime 
- apparentTemperatureHigh 
- apparentTemperatureHighTime
- apparentTemperatureLow     
- apparentTemperatureLowTime
- dewPoint                 
- humidity         
- pressure                  
- windSpeed           
- windGust                 
- windGustTime         
- windBearing    
- cloudCover        
- uvIndex          
- uvIndexTime           
- visibility     
- ozone                   
- temperatureMin    
- temperatureMinTime         
- temperatureMax     
- temperatureMaxTime        
- apparentTemperatureMin 
- apparentTemperatureMinTime 
- apparentTemperatureMax    
- apparentTemperatureMaxTime 

A example of these daily variables for the observation t is shown as below:


```r
as.data.frame(t[[2]])
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["time"],"name":[1],"type":["S3: POSIXct"],"align":["right"]},{"label":["precipIntensity"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["precipProbability"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["precipIntensityError"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["precipType"],"name":[5],"type":["chr"],"align":["left"]}],"data":[{"1":"2018-09-06 02:30:00","2":"0.000","3":"0.00","4":"NA","5":"NA"},{"1":"2018-09-06 02:31:00","2":"0.000","3":"0.00","4":"NA","5":"NA"},{"1":"2018-09-06 02:32:00","2":"0.000","3":"0.00","4":"NA","5":"NA"},{"1":"2018-09-06 02:33:00","2":"0.000","3":"0.00","4":"NA","5":"NA"},{"1":"2018-09-06 02:34:00","2":"0.000","3":"0.00","4":"NA","5":"NA"},{"1":"2018-09-06 02:35:00","2":"0.000","3":"0.00","4":"NA","5":"NA"},{"1":"2018-09-06 02:36:00","2":"0.000","3":"0.00","4":"NA","5":"NA"},{"1":"2018-09-06 02:37:00","2":"0.000","3":"0.00","4":"NA","5":"NA"},{"1":"2018-09-06 02:38:00","2":"0.000","3":"0.00","4":"NA","5":"NA"},{"1":"2018-09-06 02:39:00","2":"0.000","3":"0.00","4":"NA","5":"NA"},{"1":"2018-09-06 02:40:00","2":"0.000","3":"0.00","4":"NA","5":"NA"},{"1":"2018-09-06 02:41:00","2":"0.000","3":"0.00","4":"NA","5":"NA"},{"1":"2018-09-06 02:42:00","2":"0.000","3":"0.00","4":"NA","5":"NA"},{"1":"2018-09-06 02:43:00","2":"0.002","3":"0.01","4":"0.000","5":"rain"},{"1":"2018-09-06 02:44:00","2":"0.002","3":"0.02","4":"0.000","5":"rain"},{"1":"2018-09-06 02:45:00","2":"0.002","3":"0.03","4":"0.000","5":"rain"},{"1":"2018-09-06 02:46:00","2":"0.002","3":"0.04","4":"0.000","5":"rain"},{"1":"2018-09-06 02:47:00","2":"0.002","3":"0.05","4":"0.000","5":"rain"},{"1":"2018-09-06 02:48:00","2":"0.002","3":"0.06","4":"0.000","5":"rain"},{"1":"2018-09-06 02:49:00","2":"0.002","3":"0.08","4":"0.000","5":"rain"},{"1":"2018-09-06 02:50:00","2":"0.002","3":"0.09","4":"0.000","5":"rain"},{"1":"2018-09-06 02:51:00","2":"0.002","3":"0.12","4":"0.000","5":"rain"},{"1":"2018-09-06 02:52:00","2":"0.002","3":"0.13","4":"0.000","5":"rain"},{"1":"2018-09-06 02:53:00","2":"0.002","3":"0.16","4":"0.000","5":"rain"},{"1":"2018-09-06 02:54:00","2":"0.002","3":"0.16","4":"0.000","5":"rain"},{"1":"2018-09-06 02:55:00","2":"0.002","3":"0.19","4":"0.000","5":"rain"},{"1":"2018-09-06 02:56:00","2":"0.002","3":"0.21","4":"0.000","5":"rain"},{"1":"2018-09-06 02:57:00","2":"0.002","3":"0.21","4":"0.000","5":"rain"},{"1":"2018-09-06 02:58:00","2":"0.002","3":"0.23","4":"0.000","5":"rain"},{"1":"2018-09-06 02:59:00","2":"0.002","3":"0.23","4":"0.000","5":"rain"},{"1":"2018-09-06 03:00:00","2":"0.002","3":"0.25","4":"0.000","5":"rain"},{"1":"2018-09-06 03:01:00","2":"0.002","3":"0.26","4":"0.000","5":"rain"},{"1":"2018-09-06 03:02:00","2":"0.002","3":"0.26","4":"0.000","5":"rain"},{"1":"2018-09-06 03:03:00","2":"0.002","3":"0.27","4":"0.000","5":"rain"},{"1":"2018-09-06 03:04:00","2":"0.002","3":"0.28","4":"0.000","5":"rain"},{"1":"2018-09-06 03:05:00","2":"0.002","3":"0.27","4":"0.000","5":"rain"},{"1":"2018-09-06 03:06:00","2":"0.002","3":"0.28","4":"0.000","5":"rain"},{"1":"2018-09-06 03:07:00","2":"0.002","3":"0.28","4":"0.000","5":"rain"},{"1":"2018-09-06 03:08:00","2":"0.002","3":"0.28","4":"0.000","5":"rain"},{"1":"2018-09-06 03:09:00","2":"0.002","3":"0.28","4":"0.000","5":"rain"},{"1":"2018-09-06 03:10:00","2":"0.002","3":"0.27","4":"0.000","5":"rain"},{"1":"2018-09-06 03:11:00","2":"0.002","3":"0.28","4":"0.000","5":"rain"},{"1":"2018-09-06 03:12:00","2":"0.002","3":"0.27","4":"0.000","5":"rain"},{"1":"2018-09-06 03:13:00","2":"0.002","3":"0.27","4":"0.000","5":"rain"},{"1":"2018-09-06 03:14:00","2":"0.002","3":"0.26","4":"0.000","5":"rain"},{"1":"2018-09-06 03:15:00","2":"0.002","3":"0.26","4":"0.000","5":"rain"},{"1":"2018-09-06 03:16:00","2":"0.002","3":"0.26","4":"0.000","5":"rain"},{"1":"2018-09-06 03:17:00","2":"0.002","3":"0.25","4":"0.000","5":"rain"},{"1":"2018-09-06 03:18:00","2":"0.002","3":"0.24","4":"0.000","5":"rain"},{"1":"2018-09-06 03:19:00","2":"0.002","3":"0.24","4":"0.000","5":"rain"},{"1":"2018-09-06 03:20:00","2":"0.002","3":"0.22","4":"0.000","5":"rain"},{"1":"2018-09-06 03:21:00","2":"0.002","3":"0.22","4":"0.000","5":"rain"},{"1":"2018-09-06 03:22:00","2":"0.002","3":"0.22","4":"0.000","5":"rain"},{"1":"2018-09-06 03:23:00","2":"0.002","3":"0.22","4":"0.000","5":"rain"},{"1":"2018-09-06 03:24:00","2":"0.002","3":"0.22","4":"0.000","5":"rain"},{"1":"2018-09-06 03:25:00","2":"0.002","3":"0.21","4":"0.000","5":"rain"},{"1":"2018-09-06 03:26:00","2":"0.002","3":"0.20","4":"0.000","5":"rain"},{"1":"2018-09-06 03:27:00","2":"0.002","3":"0.21","4":"0.000","5":"rain"},{"1":"2018-09-06 03:28:00","2":"0.002","3":"0.20","4":"0.000","5":"rain"},{"1":"2018-09-06 03:29:00","2":"0.002","3":"0.19","4":"0.000","5":"rain"},{"1":"2018-09-06 03:30:00","2":"0.002","3":"0.19","4":"0.001","5":"rain"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Historical hourly data

The following 18 meteoreological variables can be obtained from DarkSky API. For each variabele, there are 24 observations for them, which represents the hourly weather data in that day.

- time               
- summary            
- icon
- precipIntensity 
- precipProbability
- temperature        
- apparentTemperature
- dewPoint           
- humidity          
- pressure           
- windSpeed        
- windGust           
- windBearing        
- cloudCover         
- uvIndex            
- visibility         
- ozone             
- precipType         

The hourly historical weather data for the observation t is shown below:



```r
t[[1]]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["time"],"name":[1],"type":["S3: POSIXct"],"align":["right"]},{"label":["summary"],"name":[2],"type":["chr"],"align":["left"]},{"label":["icon"],"name":[3],"type":["chr"],"align":["left"]},{"label":["precipIntensity"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["precipProbability"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["temperature"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["apparentTemperature"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["dewPoint"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["humidity"],"name":[9],"type":["dbl"],"align":["right"]},{"label":["pressure"],"name":[10],"type":["dbl"],"align":["right"]},{"label":["windSpeed"],"name":[11],"type":["dbl"],"align":["right"]},{"label":["windGust"],"name":[12],"type":["dbl"],"align":["right"]},{"label":["windBearing"],"name":[13],"type":["int"],"align":["right"]},{"label":["cloudCover"],"name":[14],"type":["dbl"],"align":["right"]},{"label":["uvIndex"],"name":[15],"type":["int"],"align":["right"]},{"label":["visibility"],"name":[16],"type":["dbl"],"align":["right"]},{"label":["ozone"],"name":[17],"type":["dbl"],"align":["right"]},{"label":["precipType"],"name":[18],"type":["chr"],"align":["left"]}],"data":[{"1":"2018-09-06 00:00:00","2":"Humid","3":"clear-night","4":"0.0000","5":"0.00","6":"78.61","7":"79.97","8":"72.22","9":"0.81","10":"1019.55","11":"2.57","12":"3.34","13":"354","14":"0.10","15":"0","16":"9.15","17":"275.32","18":"NA"},{"1":"2018-09-06 01:00:00","2":"Humid","3":"clear-night","4":"0.0000","5":"0.00","6":"77.90","7":"79.27","8":"72.14","9":"0.83","10":"1019.72","11":"0.89","12":"3.45","13":"30","14":"0.00","15":"0","16":"9.15","17":"274.72","18":"NA"},{"1":"2018-09-06 02:00:00","2":"Clear","3":"clear-night","4":"0.0000","5":"0.00","6":"77.25","7":"78.58","8":"71.67","9":"0.83","10":"1019.75","11":"1.72","12":"2.90","13":"182","14":"0.06","15":"0","16":"10.00","17":"274.40","18":"NA"},{"1":"2018-09-06 03:00:00","2":"Partly Cloudy","3":"partly-cloudy-night","4":"0.0000","5":"0.00","6":"76.01","7":"77.29","8":"70.99","9":"0.85","10":"1019.85","11":"2.36","12":"3.88","13":"153","14":"0.26","15":"0","16":"10.00","17":"274.51","18":"NA"},{"1":"2018-09-06 04:00:00","2":"Clear","3":"clear-night","4":"0.0005","5":"0.02","6":"74.36","7":"75.63","8":"70.52","9":"0.88","10":"1020.16","11":"2.90","12":"5.22","13":"117","14":"0.21","15":"0","16":"10.00","17":"274.56","18":"rain"},{"1":"2018-09-06 05:00:00","2":"Clear","3":"clear-night","4":"0.0004","5":"0.02","6":"73.09","7":"74.31","8":"69.83","9":"0.90","10":"1020.25","11":"2.60","12":"5.45","13":"138","14":"0.14","15":"0","16":"10.00","17":"274.68","18":"rain"},{"1":"2018-09-06 06:00:00","2":"Clear","3":"clear-night","4":"0.0000","5":"0.00","6":"71.82","7":"73.05","8":"69.50","9":"0.92","10":"1020.43","11":"2.50","12":"4.84","13":"123","14":"0.14","15":"0","16":"10.00","17":"274.87","18":"NA"},{"1":"2018-09-06 07:00:00","2":"Partly Cloudy","3":"partly-cloudy-day","4":"0.0000","5":"0.00","6":"71.27","7":"72.54","8":"69.59","9":"0.94","10":"1020.83","11":"2.88","12":"4.93","13":"118","14":"0.33","15":"0","16":"10.00","17":"274.87","18":"NA"},{"1":"2018-09-06 08:00:00","2":"Partly Cloudy","3":"partly-cloudy-day","4":"0.0015","5":"0.03","6":"73.24","7":"74.38","8":"69.29","9":"0.87","10":"1020.97","11":"3.13","12":"5.22","13":"124","14":"0.39","15":"1","16":"10.00","17":"274.52","18":"rain"},{"1":"2018-09-06 09:00:00","2":"Partly Cloudy","3":"partly-cloudy-day","4":"0.0004","5":"0.02","6":"76.93","7":"77.96","8":"69.31","9":"0.77","10":"1021.30","11":"3.68","12":"5.74","13":"127","14":"0.51","15":"2","16":"10.00","17":"274.06","18":"rain"},{"1":"2018-09-06 10:00:00","2":"Partly Cloudy","3":"partly-cloudy-day","4":"0.0004","5":"0.02","6":"80.74","7":"83.99","8":"69.51","9":"0.69","10":"1021.28","11":"4.39","12":"6.77","13":"134","14":"0.41","15":"4","16":"10.00","17":"273.64","18":"rain"},{"1":"2018-09-06 11:00:00","2":"Partly Cloudy","3":"partly-cloudy-day","4":"0.0016","5":"0.03","6":"83.21","7":"86.74","8":"68.84","9":"0.62","10":"1021.16","11":"5.47","12":"7.86","13":"131","14":"0.44","15":"5","16":"10.00","17":"273.47","18":"rain"},{"1":"2018-09-06 12:00:00","2":"Partly Cloudy","3":"partly-cloudy-day","4":"0.0069","5":"0.03","6":"84.72","7":"88.38","8":"68.65","9":"0.59","10":"1020.77","11":"5.94","12":"8.42","13":"125","14":"0.53","15":"6","16":"10.00","17":"273.36","18":"rain"},{"1":"2018-09-06 13:00:00","2":"Partly Cloudy","3":"partly-cloudy-day","4":"0.0129","5":"0.06","6":"85.41","7":"89.12","8":"68.55","9":"0.57","10":"1020.40","11":"6.83","12":"8.91","13":"120","14":"0.44","15":"7","16":"10.00","17":"273.12","18":"rain"},{"1":"2018-09-06 14:00:00","2":"Partly Cloudy","3":"partly-cloudy-day","4":"0.0179","5":"0.09","6":"85.42","7":"88.89","8":"68.16","9":"0.56","10":"1019.92","11":"6.97","12":"8.97","13":"110","14":"0.59","15":"6","16":"10.00","17":"272.88","18":"rain"},{"1":"2018-09-06 15:00:00","2":"Possible Light Rain","3":"rain","4":"0.0311","5":"0.15","6":"84.72","7":"88.06","8":"68.10","9":"0.58","10":"1019.37","11":"7.40","12":"9.43","13":"99","14":"0.70","15":"5","16":"10.00","17":"272.51","18":"rain"},{"1":"2018-09-06 16:00:00","2":"Possible Light Rain","3":"rain","4":"0.0496","5":"0.21","6":"82.79","7":"86.51","8":"69.38","9":"0.64","10":"1018.79","11":"7.57","12":"9.59","13":"94","14":"0.73","15":"3","16":"9.95","17":"272.17","18":"rain"},{"1":"2018-09-06 17:00:00","2":"Possible Light Rain","3":"rain","4":"0.0393","5":"0.21","6":"82.19","7":"85.97","8":"69.79","9":"0.66","10":"1018.78","11":"7.86","12":"10.36","13":"81","14":"0.93","15":"2","16":"10.00","17":"271.66","18":"rain"},{"1":"2018-09-06 18:00:00","2":"Possible Light Rain","3":"rain","4":"0.0309","5":"0.21","6":"80.62","7":"84.20","8":"70.53","9":"0.71","10":"1018.78","11":"7.69","12":"11.09","13":"74","14":"0.89","15":"1","16":"10.00","17":"271.28","18":"rain"},{"1":"2018-09-06 19:00:00","2":"Possible Light Rain","3":"rain","4":"0.0304","5":"0.23","6":"79.76","7":"82.93","8":"70.41","9":"0.73","10":"1018.43","11":"6.66","12":"11.74","13":"73","14":"0.84","15":"0","16":"9.45","17":"270.98","18":"rain"},{"1":"2018-09-06 20:00:00","2":"Possible Light Rain","3":"rain","4":"0.0362","5":"0.26","6":"77.41","7":"78.66","8":"71.11","9":"0.81","10":"1019.08","11":"7.41","12":"12.73","13":"71","14":"0.89","15":"0","16":"9.81","17":"270.93","18":"rain"},{"1":"2018-09-06 21:00:00","2":"Light Rain","3":"rain","4":"0.0474","5":"0.31","6":"76.19","7":"77.46","8":"71.00","9":"0.84","10":"1019.47","11":"7.60","12":"13.88","13":"66","14":"0.91","15":"0","16":"9.67","17":"271.12","18":"rain"},{"1":"2018-09-06 22:00:00","2":"Light Rain","3":"rain","4":"0.0511","5":"0.36","6":"75.49","7":"76.81","8":"71.22","9":"0.87","10":"1019.41","11":"7.34","12":"14.52","13":"67","14":"0.99","15":"0","16":"9.33","17":"271.26","18":"rain"},{"1":"2018-09-06 23:00:00","2":"Possible Light Rain","3":"rain","4":"0.0324","5":"0.24","6":"74.64","7":"75.94","8":"70.82","9":"0.88","10":"1019.63","11":"7.42","12":"14.28","13":"69","14":"0.93","15":"0","16":"9.67","17":"271.27","18":"rain"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>



## Real-time data (<= 1 hour)

- time               
- summary  
- icon               
- precipIntensity  
- precipProbability  
- temperature       
- apparentTemperature
- dewPoint     
- humidity        
- pressure       
- windSpeed   
- windGust   
- windBearing     
- cloudCover  
- uvIndex      
- visibility 
- ozone   

The sample real-time data for the observation t is shown below:


```r
as.data.frame(t[[3]])
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["time"],"name":[1],"type":["S3: POSIXct"],"align":["right"]},{"label":["summary"],"name":[2],"type":["chr"],"align":["left"]},{"label":["icon"],"name":[3],"type":["chr"],"align":["left"]},{"label":["sunriseTime"],"name":[4],"type":["S3: POSIXct"],"align":["right"]},{"label":["sunsetTime"],"name":[5],"type":["S3: POSIXct"],"align":["right"]},{"label":["moonPhase"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["precipIntensity"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["precipIntensityMax"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["precipIntensityMaxTime"],"name":[9],"type":["S3: POSIXct"],"align":["right"]},{"label":["precipProbability"],"name":[10],"type":["dbl"],"align":["right"]},{"label":["precipType"],"name":[11],"type":["chr"],"align":["left"]},{"label":["temperatureHigh"],"name":[12],"type":["dbl"],"align":["right"]},{"label":["temperatureHighTime"],"name":[13],"type":["int"],"align":["right"]},{"label":["temperatureLow"],"name":[14],"type":["dbl"],"align":["right"]},{"label":["temperatureLowTime"],"name":[15],"type":["int"],"align":["right"]},{"label":["apparentTemperatureHigh"],"name":[16],"type":["dbl"],"align":["right"]},{"label":["apparentTemperatureHighTime"],"name":[17],"type":["int"],"align":["right"]},{"label":["apparentTemperatureLow"],"name":[18],"type":["dbl"],"align":["right"]},{"label":["apparentTemperatureLowTime"],"name":[19],"type":["int"],"align":["right"]},{"label":["dewPoint"],"name":[20],"type":["dbl"],"align":["right"]},{"label":["humidity"],"name":[21],"type":["dbl"],"align":["right"]},{"label":["pressure"],"name":[22],"type":["dbl"],"align":["right"]},{"label":["windSpeed"],"name":[23],"type":["dbl"],"align":["right"]},{"label":["windGust"],"name":[24],"type":["dbl"],"align":["right"]},{"label":["windGustTime"],"name":[25],"type":["int"],"align":["right"]},{"label":["windBearing"],"name":[26],"type":["int"],"align":["right"]},{"label":["cloudCover"],"name":[27],"type":["dbl"],"align":["right"]},{"label":["uvIndex"],"name":[28],"type":["int"],"align":["right"]},{"label":["uvIndexTime"],"name":[29],"type":["int"],"align":["right"]},{"label":["visibility"],"name":[30],"type":["int"],"align":["right"]},{"label":["ozone"],"name":[31],"type":["dbl"],"align":["right"]},{"label":["temperatureMin"],"name":[32],"type":["dbl"],"align":["right"]},{"label":["temperatureMinTime"],"name":[33],"type":["S3: POSIXct"],"align":["right"]},{"label":["temperatureMax"],"name":[34],"type":["dbl"],"align":["right"]},{"label":["temperatureMaxTime"],"name":[35],"type":["S3: POSIXct"],"align":["right"]},{"label":["apparentTemperatureMin"],"name":[36],"type":["dbl"],"align":["right"]},{"label":["apparentTemperatureMinTime"],"name":[37],"type":["S3: POSIXct"],"align":["right"]},{"label":["apparentTemperatureMax"],"name":[38],"type":["dbl"],"align":["right"]},{"label":["apparentTemperatureMaxTime"],"name":[39],"type":["S3: POSIXct"],"align":["right"]}],"data":[{"1":"2018-09-06","2":"Light rain starting in the afternoon.","3":"rain","4":"2018-09-06 06:36:15","5":"2018-09-06 19:25:13","6":"0.89","7":"0.0162","8":"0.0511","9":"2018-09-06 22:00:00","10":"0.74","11":"rain","12":"85.42","13":"1536260400","14":"70.63","15":"1536325200","16":"89.12","17":"1536256800","18":"71.98","19":"1536325200","20":"70.05","21":"0.76","22":"1019.92","23":"4.37","24":"14.52","25":"1536289200","26":"96","27":"0.51","28":"7","29":"1536256800","30":"10","31":"273.17","32":"71.27","33":"2018-09-06 07:00:00","34":"85.42","35":"2018-09-06 14:00:00","36":"72.54","37":"2018-09-06 07:00:00","38":"89.12","39":"2018-09-06 13:00:00"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>


We can also customize our data by pooled the daily, hourly and real-time weather variables together in one table using loops. One example has been shown as below:



