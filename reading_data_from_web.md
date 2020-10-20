Data Science Midterm
================

``` r
library(tidyverse)
```

    ## -- Attaching packages ------------- tidyverse 1.3.0 --

    ## v ggplot2 3.3.2     v purrr   0.3.4
    ## v tibble  3.0.3     v dplyr   1.0.2
    ## v tidyr   1.1.2     v stringr 1.4.0
    ## v readr   1.3.1     v forcats 0.5.0

    ## -- Conflicts ---------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(rvest)
```

    ## Loading required package: xml2

    ## 
    ## Attaching package: 'rvest'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     pluck

    ## The following object is masked from 'package:readr':
    ## 
    ##     guess_encoding

``` r
library(httr)

knitr::opts_chunk$set(
  fig.width = 6, 
  fig.asp = .6,
  out.width = "90%"
)

theme_set(theme_minimal() + theme(legend.position = "bottom"))

options(
  ggplot2.continuous.colour = "viridis", 
  ggplot2.continuous.fill = "viridis"
)

scale_colour_discrete = scale_colour_viridis_d
scale_fill_discrete = scale_fill_viridis_d
```

\#Scrape a table

I want first table from this page:
<https://www.youtube.com/watch?v=2RjK6fYWdzM&feature=youtu.be>

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"

drug_use_html = read_html(url)
```

Extract the table - focus on 1st one

``` r
drug_use_html %>% 
  html_nodes(css = "table") %>% 
  first() %>% 
  html_table() %>% 
  slice(-1) %>% 
  as_tibble()
```

    ## # A tibble: 56 x 16
    ##    State `12+(2013-2014)` `12+(2014-2015)` `12+(P Value)` `12-17(2013-201~
    ##    <chr> <chr>            <chr>            <chr>          <chr>           
    ##  1 Tota~ 12.90a           13.36            0.002          13.28b          
    ##  2 Nort~ 13.88a           14.66            0.005          13.98           
    ##  3 Midw~ 12.40b           12.76            0.082          12.45           
    ##  4 South 11.24a           11.64            0.029          12.02           
    ##  5 West  15.27            15.62            0.262          15.53a          
    ##  6 Alab~ 9.98             9.60             0.426          9.90            
    ##  7 Alas~ 19.60a           21.92            0.010          17.30           
    ##  8 Ariz~ 13.69            13.12            0.364          15.12           
    ##  9 Arka~ 11.37            11.59            0.678          12.79           
    ## 10 Cali~ 14.49            15.25            0.103          15.03           
    ## # ... with 46 more rows, and 11 more variables: `12-17(2014-2015)` <chr>,
    ## #   `12-17(P Value)` <chr>, `18-25(2013-2014)` <chr>, `18-25(2014-2015)` <chr>,
    ## #   `18-25(P Value)` <chr>, `26+(2013-2014)` <chr>, `26+(2014-2015)` <chr>,
    ## #   `26+(P Value)` <chr>, `18+(2013-2014)` <chr>, `18+(2014-2015)` <chr>,
    ## #   `18+(P Value)` <chr>

Star wars movie imdb page: <https://www.imdb.com/list/ls070150896/>

(want df with title, run time, how much money it made)

``` r
url_ = "https://www.imdb.com/list/ls070150896/"

swm_html = read_html(url_)
```

Grab elements we want

``` r
titles_vec =
  swm_html %>%
  html_nodes(css = ".lister-item-header a") %>% 
  html_text()
  
gross_r_vec =
  swm_html %>%
  html_nodes(css = ".text-small:nth-child(7) span:nth-child(5)") %>% 
  html_text()

runtime_vec =
  swm_html %>%
  html_nodes(css = ".runtime") %>% 
  html_text()

swm_df = 
  tibble(
    title = titles_vec, 
    gross_rev = gross_r_vec,
    run_time = runtime_vec
  )
```

\#APIs

Get some water data

<https://data.cityofnewyork.us/Environment/Water-Consumption-In-The-New-York-City/ia2d-e54m>

``` r
nyc_water =
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.csv") %>% 
  content("parsed")
```

    ## Parsed with column specification:
    ## cols(
    ##   year = col_double(),
    ##   new_york_city_population = col_double(),
    ##   nyc_consumption_million_gallons_per_day = col_double(),
    ##   per_capita_gallons_per_person_per_day = col_double()
    ## )

``` r
#JSON structure?
#Same data but diff structure
nyc_water2 = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.json") %>% 
  content("text") %>% 
  jsonlite::fromJSON() %>% 
  as.tibble()
```

    ## Warning: `as.tibble()` is deprecated as of tibble 2.0.0.
    ## Please use `as_tibble()` instead.
    ## The signature and semantics have changed, see `?as_tibble`.
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_warnings()` to see where this warning was generated.

CDC API:
<https://chronicdata.cdc.gov/Behavioral-Risk-Factors/Behavioral-Risk-Factors-Selected-Metropolitan-Area/acme-vg9e>

``` r
brfss_2010 = 
  GET("https://chronicdata.cdc.gov/resource/acme-vg9e.csv") %>% 
  content("parsed") 
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character(),
    ##   year = col_double(),
    ##   sample_size = col_double(),
    ##   data_value = col_double(),
    ##   confidence_limit_low = col_double(),
    ##   confidence_limit_high = col_double(),
    ##   display_order = col_double(),
    ##   locationid = col_logical()
    ## )

    ## See spec(...) for full column specifications.

``` r
#changing query parameters
brfss_2010 = 
  GET("https://chronicdata.cdc.gov/resource/acme-vg9e.csv", query = list("$limit" = 5000)) %>% 
  content("parsed") 
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character(),
    ##   year = col_double(),
    ##   sample_size = col_double(),
    ##   data_value = col_double(),
    ##   confidence_limit_low = col_double(),
    ##   confidence_limit_high = col_double(),
    ##   display_order = col_double(),
    ##   locationid = col_logical()
    ## )
    ## See spec(...) for full column specifications.
