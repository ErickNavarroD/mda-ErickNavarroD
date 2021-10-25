Mini Data Analysis Milestone 3
================
Erick Navarro
17/10/2021

## Introduction

In the previous milestones I explored the datasets contained in the `datateachr` package and, at the end of the milestone 1, I chose to work with the `steam_games()` dataset.

In milestone 2, I summarized some variables contained in that dataset, and formulated two final research questions to focus on, which are the following:

1.  *Do recent released games have better reviews than the older ones?*
2.  *Are the most popular games the ones with the best overall review?*

This is the last milestone of the Mini Data Analysis project. Its objective is to manipulate special data types in R, such as factors and/or dates and times, fitting a model object to your data, and reading and writing data as separate files. The tasks presented here are intended to sharpen some of the results I obtained in the previous stages of the project, in order to reach a deeper conclusions of my research questions.

## Setup

As a first step, I loaded the packages that I was going to use throughout this milestone

``` r
library(datateachr) 
library(tidyverse)
library(lubridate)
```
