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

As a first step, I loaded the packages that I was going to use throughout this milestone, as well as the final dataset produced at the end of the Milestone 2

``` r
library(datateachr) 
library(tidyverse)
library(lubridate)
library(here)
steam_games_m3 = readRDS(here("Milestone_2", "steam_games_mda2_final"))
```

## Exercise 1: Special data types

For this exercise, I completed 2 of the 3 tasks specified in the [instructions](https://stat545.stat.ubc.ca/mini-project/mini-project-3/). In the following subsections, I specified which ones I chose and which question I used.

### Task 1

> Task: If your data has some sort of time-based column like a date (but something more granular than just a year):
> - Make a new column that uses a function from the lubridate or tsibble package to modify your original time-based column. (3 points)
> - Note that you might first have to make a time-based column using a function like ymd(), but this doesn’t count.
> - Examples of something you might do here: extract the day of the year from a date, or extract the weekday, or let 24 hours elapse on your dates.
> - Then, in a sentence or two, explain how your new column might be useful in exploring a research question. (1 point for demonstrating understanding of the function you used, and 1 point for your justification, which could be subtle or speculative).
> - For example, you could say something like “Investigating the day of the week might be insightful because penguins don’t work on weekends, and so may respond differently”.
> Question: *Do recent released games have better reviews than the older ones?*

### Task 2

> Task: Produce a new plot that groups some factor levels together into an “other” category (or something similar), using the forcats package (3 points). Then, in a sentence or two, briefly explain why you chose this grouping (1 point here for demonstrating understanding of the grouping, and 1 point for demonstrating some justification for the grouping, which could be subtle or speculative.)
> Question: *Are the most popular games the ones with the best overall review?*

This task asked me to modify a plot that was previously produced in Milestone 2, which involved plotting across at least three groups. For the reader's convenience, said plot was recreated here:

``` r
#Recreating the plot of Milestone 2's task 1.2Q1
steam_games_m3 %>% 
  filter(!is.na(popularity), #Remove the games that have NA in the number of reviews
         all_reviews_category %in% (steam_games_m3$all_reviews_category %>% 
                                      table() %>% 
                                      names())[1:9]) %>% #Remove the games that did not have enough reviews to have an overall review (discussed in Q1 - Extract categories) 
  ggplot(aes(x = all_reviews_category, y = popularity, fill = all_reviews_category))+
  geom_violin( alpha = 0.5) +
  geom_boxplot(width= 0.1, alpha = 0.3) +
  stat_summary(fun = mean, colour = "black", shape = 8)+
  scale_y_log10(label = scales::label_dollar(prefix = "")) +
  ylab("Game popularity")+
  xlab("Overall review")+
  guides(fill = "none")  +
  theme_classic()
```

![](mda_milestone_3_files/figure-markdown_github/unnamed-chunk-2-1.png)

To complete the task, I decided to modify the number of categories that are presented in said plot, which were too many (Overwhelmingly Positive, Very Positive, Positive, Mostly Positive, Mixed, Mostly Negative, Negative, Very Negative and Overwhelmingly Negative)

Besides being potentially confusing to have a high number of catefories, some of them tend to have overlaping ranges of positive/negative reviews. For example, the Overwhelmingly Positive category tends to be assigned to games with 95-100% of positive reviews, while Very Positive to games with 80-97%. Additionally, some games with the category Positive, which is in a lower rank of "positiveness" have 100% positive reviews, as you can see in this example:

``` r
steam_games_m3 %>% 
  filter(id == "42") %>% 
  knitr::kable(format = "markdown")
```

<table>
<colgroup>
<col width="2%" />
<col width="15%" />
<col width="7%" />
<col width="11%" />
<col width="36%" />
<col width="11%" />
<col width="10%" />
<col width="6%" />
</colgroup>
<thead>
<tr class="header">
<th align="right">id</th>
<th align="left">name</th>
<th align="left">release_date</th>
<th align="left">release_date_category</th>
<th align="left">all_reviews</th>
<th align="left">all_reviews_category</th>
<th align="right">all_reviews_number</th>
<th align="right">popularity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="right">42</td>
<td align="left">GOD WARS The Complete Legend</td>
<td align="left">2019-06-13</td>
<td align="left">New</td>
<td align="left">Positive,(11),- 100% of the 11 user reviews for this game are positive.</td>
<td align="left">Positive</td>
<td align="right">2</td>
<td align="right">11</td>
</tr>
</tbody>
</table>

As it can be observed, the game review categorization seems to be quite subjective in the different levels of positiveness and negativeness; the previous example is just one of the many cases where a game has a percentage of positive or negative reviews that is in the range of more than 1 category. Therefore, I decided to colapse some levels as following:

-   Positive: comprising the original categories Overwhelmingly Positive, Very Positive, Positive and Mostly Positive
-   Mixed: comprising the original category Mixed
-   Negative: comprising the original categories Overwhelmingly Negative, Very Negative, Negative and Mostly Negative

After grouping the redundant factor levels together, I plotted again with the reduced number of categories.

``` r
#grouping some factor levels together into an "other" category
task_1.2 = (steam_games_m3 %>% 
              mutate(all_reviews_category = fct_collapse(all_reviews_category,
                                                         Positive = c("Overwhelmingly Positive", "Very Positive", "Positive" , "Mostly Positive"),
                                                         Mixed = "Mixed",
                                                         Negative = c("Overwhelmingly Negative", "Very Negative", "Negative" , "Mostly Negative"),
                                                         other_level = "Undetermined" ))
)

#Plot again with the new collapsed categories
task_1.2 %>% 
  filter(!is.na(popularity), #Remove the games that have NA in the number of reviews
         all_reviews_category != "Undetermined") %>% #Remove the games that did not have enough reviews to have an overall review (discussed in Q1 - Extract categories) 
  ggplot(aes(x = all_reviews_category, y = popularity, fill = all_reviews_category))+
  geom_violin( alpha = 0.5) +
  geom_boxplot(width= 0.1, alpha = 0.3) +
  stat_summary(fun = mean, colour = "black", shape = 8)+
  scale_y_log10(label = scales::label_dollar(prefix = "")) +
  ylab("Game popularity")+
  xlab("Overall review")+
  guides(fill = "none")  +
  theme_classic()
```

![](mda_milestone_3_files/figure-markdown_github/unnamed-chunk-4-1.png)

## Exercise 2: Modelling