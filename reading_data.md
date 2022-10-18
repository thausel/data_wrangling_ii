Reading Data from the Web
================
Tim Hauser

## Initial setup

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.3.6      ✔ purrr   0.3.4 
    ## ✔ tibble  3.1.8      ✔ dplyr   1.0.10
    ## ✔ tidyr   1.2.0      ✔ stringr 1.4.1 
    ## ✔ readr   2.1.2      ✔ forcats 0.5.2 
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(rvest)
```

    ## 
    ## Attaching package: 'rvest'
    ## 
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

## Scraping a table (rvest package)

I want the first table from [this
page](http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm)

Read in the HTML:

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"
drug_use_html = read_html(url)

drug_use_html
```

    ## {html_document}
    ## <html lang="en">
    ## [1] <head>\n<link rel="P3Pv1" href="http://www.samhsa.gov/w3c/p3p.xml">\n<tit ...
    ## [2] <body>\r\n\r\n<noscript>\r\n<p>Your browser's Javascript is off. Hyperlin ...

Extract the table(s), focus on the first one

``` r
table_marj =
  drug_use_html %>%
  html_nodes(css = "table") %>% 
# this gives us all 15 tables from the webpage
  first() %>%
  html_table() %>% 
  slice(-1) %>% 
  as_tibble()
```

## Learning assessment

Create a data frame that contains the cost of living table for New York
from this page.

``` r
nyc_cost = 
  read_html("https://www.bestplaces.net/cost_of_living/city/new_york/new_york") %>%
  html_table(header = TRUE) %>%
  first()
```

## Star wars movie info – CSS Selectors

I want the data from [here](https://www.imdb.com/list/ls070150896/).

``` r
swm_html = 
  read_html("https://www.imdb.com/list/ls070150896/")
```

Grab elements I want

``` r
title_vec = 
  swm_html %>%
  html_elements(".lister-item-header a") %>%
# I got this from the selector gadget
  html_text()

title_vec
```

    ## [1] "Star Wars: Episode I - The Phantom Menace"     
    ## [2] "Star Wars: Episode II - Attack of the Clones"  
    ## [3] "Star Wars: Episode III - Revenge of the Sith"  
    ## [4] "Star Wars"                                     
    ## [5] "Star Wars: Episode V - The Empire Strikes Back"
    ## [6] "Star Wars: Episode VI - Return of the Jedi"    
    ## [7] "Star Wars: Episode VII - The Force Awakens"    
    ## [8] "Star Wars: Episode VIII - The Last Jedi"       
    ## [9] "Star Wars: The Rise Of Skywalker"

``` r
gross_rev_vec = 
  swm_html %>%
  html_elements(".text-small:nth-child(7) span:nth-child(5)") %>%
  html_text()

gross_rev_vec
```

    ## [1] "$474.54M" "$310.68M" "$380.26M" "$322.74M" "$290.48M" "$309.13M" "$936.66M"
    ## [8] "$620.18M" "$515.20M"

``` r
runtime_vec = 
  swm_html %>%
  html_elements(".runtime") %>%
  html_text()

runtime_vec
```

    ## [1] "136 min" "142 min" "140 min" "121 min" "124 min" "131 min" "138 min"
    ## [8] "152 min" "141 min"

``` r
# combining the three tables:

swm_df = 
  tibble(
    title = title_vec,
    rev = gross_rev_vec,
    runtime = runtime_vec)

swm_df
```

    ## # A tibble: 9 × 3
    ##   title                                          rev      runtime
    ##   <chr>                                          <chr>    <chr>  
    ## 1 Star Wars: Episode I - The Phantom Menace      $474.54M 136 min
    ## 2 Star Wars: Episode II - Attack of the Clones   $310.68M 142 min
    ## 3 Star Wars: Episode III - Revenge of the Sith   $380.26M 140 min
    ## 4 Star Wars                                      $322.74M 121 min
    ## 5 Star Wars: Episode V - The Empire Strikes Back $290.48M 124 min
    ## 6 Star Wars: Episode VI - Return of the Jedi     $309.13M 131 min
    ## 7 Star Wars: Episode VII - The Force Awakens     $936.66M 138 min
    ## 8 Star Wars: Episode VIII - The Last Jedi        $620.18M 152 min
    ## 9 Star Wars: The Rise Of Skywalker               $515.20M 141 min

## Learning Assignment

This page contains the 10 most recent reviews of the movie “Napoleon
Dynamite”. Use a process similar to the one above to extract the titles
of the reviews:

``` r
dynamite_html = 
  read_html("https://www.amazon.com/product-reviews/B00005JNBQ/ref=cm_cr_arp_d_viewopt_rvwer?ie=UTF8&reviewerType=avp_only_reviews&sortBy=recent&pageNumber=1")

dynamite_html
```

    ## {html_document}
    ## <html lang="en-us" class="a-no-js" data-19ax5a9jf="dingo">
    ## [1] <head>\n<meta http-equiv="Content-Type" content="text/html; charset=UTF-8 ...
    ## [2] <body>\n<span id="cr-state-object" data-state='{"asin":"B00005JNBQ","devi ...

``` r
review_titles = 
  dynamite_html %>%
  html_elements(".a-text-bold span") %>%
  html_text()

review_stars = 
  dynamite_html %>%
  html_elements("#cm_cr-review_list .review-rating") %>%
  html_text()

review_text = 
  dynamite_html %>%
  html_elements(".review-text-content span") %>%
  html_text()

reviews = tibble(
  title = review_titles,
  stars = review_stars,
  text = review_text
)

reviews
```

    ## # A tibble: 10 × 3
    ##    title                                stars              text                 
    ##    <chr>                                <chr>              <chr>                
    ##  1 Watch to say you did                 3.0 out of 5 stars I know it's supposed…
    ##  2 Best Movie Ever!                     5.0 out of 5 stars We just love this mo…
    ##  3 Quirky                               5.0 out of 5 stars Good family film     
    ##  4 Funny movie - can't play it !        1.0 out of 5 stars Sony 4k player won't…
    ##  5 A brilliant story about teenage life 5.0 out of 5 stars Napoleon Dynamite de…
    ##  6 HUHYAH                               5.0 out of 5 stars Spicy                
    ##  7 Cult Classic                         4.0 out of 5 stars Takes a time or two …
    ##  8 Sweet                                5.0 out of 5 stars Timeless Movie. My G…
    ##  9 Cute                                 4.0 out of 5 stars Fun                  
    ## 10 great collectible                    5.0 out of 5 stars one of the greatest …

## NYC Water - Using an API (httr package)

Import as CSV and parse it:

``` r
nyc_water = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.csv") %>% 
  content("parsed")
```

    ## Rows: 43 Columns: 4
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (4): year, new_york_city_population, nyc_consumption_million_gallons_per...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
nyc_water
```

    ## # A tibble: 43 × 4
    ##     year new_york_city_population nyc_consumption_million_gallons_per_…¹ per_c…²
    ##    <dbl>                    <dbl>                                  <dbl>   <dbl>
    ##  1  1979                  7102100                                   1512     213
    ##  2  1980                  7071639                                   1506     213
    ##  3  1981                  7089241                                   1309     185
    ##  4  1982                  7109105                                   1382     194
    ##  5  1983                  7181224                                   1424     198
    ##  6  1984                  7234514                                   1465     203
    ##  7  1985                  7274054                                   1326     182
    ##  8  1986                  7319246                                   1351     185
    ##  9  1987                  7342476                                   1447     197
    ## 10  1988                  7353719                                   1484     202
    ## # … with 33 more rows, and abbreviated variable names
    ## #   ¹​nyc_consumption_million_gallons_per_day,
    ## #   ²​per_capita_gallons_per_person_per_day

Alternative is importing it as a JSON file (however, that approach needs
a bit more cleaning and hence a bit more complicated)

``` r
nyc_water_2 = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.json") %>% 
  content("text") %>%
  jsonlite::fromJSON() %>%
  as_tibble()

nyc_water_2
```

    ## # A tibble: 43 × 4
    ##    year  new_york_city_population nyc_consumption_million_gallons_per_…¹ per_c…²
    ##    <chr> <chr>                    <chr>                                  <chr>  
    ##  1 1979  7102100                  1512                                   213    
    ##  2 1980  7071639                  1506                                   213    
    ##  3 1981  7089241                  1309                                   185    
    ##  4 1982  7109105                  1382                                   194    
    ##  5 1983  7181224                  1424                                   198    
    ##  6 1984  7234514                  1465                                   203    
    ##  7 1985  7274054                  1326                                   182    
    ##  8 1986  7319246                  1351                                   185    
    ##  9 1987  7342476                  1447                                   197    
    ## 10 1988  7353719                  1484                                   202    
    ## # … with 33 more rows, and abbreviated variable names
    ## #   ¹​nyc_consumption_million_gallons_per_day,
    ## #   ²​per_capita_gallons_per_person_per_day

## BRFSS API

``` r
brfss_2010 = 
  GET("https://chronicdata.cdc.gov/resource/acme-vg9e.csv",
      query = list("$limit" = 5000)) %>% 
# by default only 1000 get downloaded
  content("parsed")
```

    ## Rows: 5000 Columns: 23
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (16): locationabbr, locationdesc, class, topic, question, response, data...
    ## dbl  (6): year, sample_size, data_value, confidence_limit_low, confidence_li...
    ## lgl  (1): locationid
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
brfss_2010
```

    ## # A tibble: 5,000 × 23
    ##     year locationa…¹ locat…² class topic quest…³ respo…⁴ sampl…⁵ data_…⁶ confi…⁷
    ##    <dbl> <chr>       <chr>   <chr> <chr> <chr>   <chr>     <dbl>   <dbl>   <dbl>
    ##  1  2010 AL          AL - M… Heal… Over… How is… Excell…      91    15.6    11  
    ##  2  2010 AL          AL - J… Heal… Over… How is… Excell…      94    18.9    14.1
    ##  3  2010 AL          AL - T… Heal… Over… How is… Excell…      58    20.8    14.1
    ##  4  2010 AL          AL - J… Heal… Over… How is… Very g…     148    30      24.9
    ##  5  2010 AL          AL - T… Heal… Over… How is… Very g…     109    29.5    23.2
    ##  6  2010 AL          AL - M… Heal… Over… How is… Very g…     177    31.3    26  
    ##  7  2010 AL          AL - J… Heal… Over… How is… Good        208    33.1    28.2
    ##  8  2010 AL          AL - M… Heal… Over… How is… Good        224    31.2    26.1
    ##  9  2010 AL          AL - T… Heal… Over… How is… Good        171    33.8    27.7
    ## 10  2010 AL          AL - M… Heal… Over… How is… Fair        120    15.5    11.7
    ## # … with 4,990 more rows, 13 more variables: confidence_limit_high <dbl>,
    ## #   display_order <dbl>, data_value_unit <chr>, data_value_type <chr>,
    ## #   data_value_footnote_symbol <chr>, data_value_footnote <chr>,
    ## #   datasource <chr>, classid <chr>, topicid <chr>, locationid <lgl>,
    ## #   questionid <chr>, respid <chr>, geolocation <chr>, and abbreviated variable
    ## #   names ¹​locationabbr, ²​locationdesc, ³​question, ⁴​response, ⁵​sample_size,
    ## #   ⁶​data_value, ⁷​confidence_limit_low

## Pokemon API

``` r
poke = 
  GET("http://pokeapi.co/api/v2/pokemon/1") %>%
  content()
```

``` r
poke$name
```

    ## [1] "bulbasaur"

``` r
poke$height
```

    ## [1] 7

``` r
poke$abilities
```

    ## [[1]]
    ## [[1]]$ability
    ## [[1]]$ability$name
    ## [1] "overgrow"
    ## 
    ## [[1]]$ability$url
    ## [1] "https://pokeapi.co/api/v2/ability/65/"
    ## 
    ## 
    ## [[1]]$is_hidden
    ## [1] FALSE
    ## 
    ## [[1]]$slot
    ## [1] 1
    ## 
    ## 
    ## [[2]]
    ## [[2]]$ability
    ## [[2]]$ability$name
    ## [1] "chlorophyll"
    ## 
    ## [[2]]$ability$url
    ## [1] "https://pokeapi.co/api/v2/ability/34/"
    ## 
    ## 
    ## [[2]]$is_hidden
    ## [1] TRUE
    ## 
    ## [[2]]$slot
    ## [1] 3
