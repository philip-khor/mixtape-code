
<!-- README.md is generated from README.Rmd. Please edit that file -->

## Overview

This repo provides replication code for the book [“Casual Inference: The
Mixtape”](http://scunning.com/stata.html) by Scott Cunningham.

## Chapters

  - [Properties of
    regression](ch03)

  - [Directed acyclical
    graphs](ch04)

  - [Potential outcomes causal
    model](ch05)

  - [Matching and
    subclassification](ch06)

  - [Regression
    discontinuity](ch07)

  - [Instrumental variables](ch08)

  - Panel data

  - Differences-in-differences

  - Synthetic control

## Packages

Most of the code relies on the following packages; note these are not
explicitly loaded in the code chunks.

``` r
# tidyverse and tidyverse-adjacent
library(magrittr)
library(hms)
library(stringr)
library(lubridate)
library(forcats)
library(feather)
library(haven)
library(httr)
library(jsonlite)
library(readxl)
library(xml2)
library(rvest)
library(modelr)
library(broom)
library(blob)
library(dbplyr)
library(purrrlyr)
library(tidyverse)

# summary statistics
library(skimr)

# datasets
library(mixtape)

# figure styling
library(hrbrthemes)

# table styling
library(stargazer)
```
