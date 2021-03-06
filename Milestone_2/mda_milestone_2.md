Mini Data Analysis Milestone 2
================
Erick Navarro
17/10/2021

The objective of this assignment is to explore the concept of tidy data and investigate further my research questions.

The first step will be loading the data and and the packages that will be used:

``` r
library(datateachr) #This package contains the dataset steam_games, which is the one that I will use for this milestone
library(tidyverse)
library(lubridate)
library(knitr)
library(scales)
library(forcats)
library(ggridges)
library(here)
```

# Task 1: Process and summarize the data (15 points)

### 1.1 Research questions (2.5 points)

To guide the direction of this assignment, I will specify the 4 research questions that were defined in my milestone 1, which are the following:

1.  *Do recent released games have better reviews than the older ones?*
2.  *Are games with better reviews the most expensive ones?*
3.  *Are the most popular games (i.e. games with more reviews) the ones with the best overall review?*
4.  *Which developer produces the most popular games (i.e games with more reviews) or the ones with the best reviews?*

### 1.2 Summarize and graphing (10 points)

For each of the research questions, I chose one summarizing and one graphing task from the ones specified in the [instructions](https://stat545.stat.ubc.ca/mini-project/mini-project-2/). After that, a short line commenting weather it is useful or not to my research question is included.

> **Summarizing tasks:** 1. Compute the *range*, *mean*, and *two other summary statistics* of **one numerical variable** across the groups of **one categorical variable** from your data. 2. Compute the number of observations for at least one of your categorical variables. Do not use the function `table()`! 3. Create a categorical variable with 3 or more groups from an existing numerical variable. You can use this new variable in the other tasks! *An example: age in years into "child, teen, adult, senior".* 4. Based on two categorical variables, calculate two summary statistics of your choosing.

> **Graphing taks:** 5. Create a graph out of summarized variables that has at least two geom layers. 6. Create a graph of your choosing, make one of the axes logarithmic, and format the axes labels so that they are "pretty" or easier to read. 7. Make a graph where it makes sense to customize the alpha transparency. 8. Create 3 histograms out of summarized variables, with each histogram having different sized bins. Pick the "best" one and explain why it is the best.

#### Q1: Do recent released games have better reviews than the older ones?

> **Summarizing task**: Create a categorical variable with 3 or more groups from an existing numerical variable.

To explore the reviews in games that were released in different time lapses, I decided to create 3 categories for the dataset, depending on their release date:

1.  New (after 2011 to the present)

2.  Intermediate (from 2001 to 2011)

3.  Old (before 2001)

``` r
#Code chunk to create the three categories described above

#First, I check the format of the release date
head(steam_games$release_date)
```

    ## [1] "May 12, 2016" "Dec 21, 2017" "Apr 24, 2018" "Dec 13, 2018" "May 6, 2003" 
    ## [6] "NaN"

``` r
class(steam_games$release_date) 
```

    ## [1] "character"

``` r
#Creating the summarized tibble
steam_games_q1 = (steam_games %>%
                    mutate(release_date = mdy(release_date)) %>% #Use the mdy() lubridate function to change the data type from character to date
                    mutate(release_date_category = case_when( release_date > dmy("01-01-2012") ~ "New",
                                                             (dmy("31-12-2011") > release_date & release_date > dmy("01-01-2001")) ~ "Intermediate",
                                                             release_date < dmy("31-12-2000") ~ "Old")) %>%
                    mutate(release_date_category = as.factor(release_date_category)) %>% #Convert the categories to a factor
                    mutate(release_date_category = fct_relevel(release_date_category, "Old", "Intermediate")) #Relevel the factors (will be useful for plotting later)
                    )

#NOTE: The format of release_date was heterogeneous in some games; it included elements such as "??\\_(???)_/??", "(When) I'm ready!", or just the year. Those games (4418/40833) failed to parse.

#Explore the number of games we have in each category
table(steam_games_q1$release_date_category) %>% knitr::kable(format = "markdown", col.names = c("Category", "Frequency"))
```

| Category     |  Frequency|
|:-------------|----------:|
| Old          |        321|
| Intermediate |       1749|
| New          |      34339|

> **Graphing task**: Create a graph out of summarized variables that has at least two geom layers & Make a graph where it makes sense to customize the alpha transparency.

Now, after creating the summarized variables, I decided to plot the reviews distribution for each category (new, intermediate and old). I divided this task in 3 steps: extracting the reviews, converting them to a numerical value, and plotting the results.

*1. Extract the reviews*

As a first step, I extracted the reviews and created a new column that stores them

``` r
#Code chunk to create a vairable with the reviews, and convert them to a numerical value that can be plotted

steam_games_q1 = (steam_games_q1 %>%
                    bind_cols(., #Bind a new column to the original dataset
                              (str_split(steam_games_q1$all_reviews, pattern = ",", simplify = TRUE) %>% #separate the content of all_reviews using commas
                                 as_tibble()%>% #Convert the matrix output to a tibble
                                 select(V1)) %>% #Select only the first column that stores the category of the overall review
                                rename( all_reviews_category= V1)) %>%  #Rename the variable to a more informative name
                    mutate(all_reviews_category = factor(all_reviews_category)) #Make the categories a factor 
                  ) 
```

Then, I explored the frequencies of the review categories to decide which number I was going to assign to each category

``` r
table(steam_games_q1$all_reviews_category) %>% knitr::kable(format = "markdown", col.names = c("Category", "Frequency"))
```

| Category                |  Frequency|
|:------------------------|----------:|
| 1 user reviews          |       3023|
| 2 user reviews          |       1926|
| 3 user reviews          |       1469|
| 4 user reviews          |       1082|
| 5 user reviews          |        948|
| 6 user reviews          |        838|
| 7 user reviews          |        693|
| 8 user reviews          |        600|
| 9 user reviews          |        528|
| Mixed                   |       4680|
| Mostly Negative         |        782|
| Mostly Positive         |       3311|
| NaN                     |       2810|
| Negative                |        135|
| Overwhelmingly Negative |          7|
| Overwhelmingly Positive |        321|
| Positive                |       3551|
| Very Negative           |         37|
| Very Positive           |       4539|

Finally, I reordered the factors according to the correct order of categories, which was going to be useful in the future in case I wanted to plot.

``` r
steam_games_q1 = (steam_games_q1 %>% mutate(all_reviews_category = fct_relevel(all_reviews_category, "Overwhelmingly Positive",
                                                                        "Very Positive",
                                                                        "Positive",
                                                                        "Mostly Positive",
                                                                        "Mixed", 
                                                                        "Mostly Negative",
                                                                        "Negative",
                                                                        "Very Negative",
                                                                        "Overwhelmingly Negative")))
```

*2. Convert the review categories to a numerical value*

As it can be seen in the previous part, there were several games that had no review category because the number of reviews they had was very small (1-9); those games were decided to be filtered out of the plot. After looking at the table, I decided to assign the following numerical value to the categories:

-   Overwhelmingly Positive: 4
-   Very Positive: 3
-   Positive: 2
-   Mostly positive: 1
-   Mixed: 0
-   Mostly Negative: -1
-   Negative: -2
-   Very Negative: -3
-   Overwhelmingly Negative: -4

``` r
#Assign a number to each category in a new column
steam_games_q1 = (steam_games_q1 %>%
                    mutate(all_reviews_number = case_when(all_reviews_category == "Overwhelmingly Positive" ~ 4,
                                                          all_reviews_category == "Very Positive" ~ 3,
                                                          all_reviews_category == "Positive" ~ 2,
                                                          all_reviews_category == "Mostly Positive" ~ 1,
                                                          all_reviews_category == "Mixed" ~ 0,
                                                          all_reviews_category == "Mostly Negative" ~ -1,
                                                          all_reviews_category == "Negative" ~ -2,
                                                          all_reviews_category == "Very Negative" ~ -3,
                                                          all_reviews_category == "Overwhelmingly Negative" ~ -4
                                                          )))

#To check that the output tibble is correct, we look at the first entries
steam_games_q1 %>% 
  select(id, all_reviews, all_reviews_category,all_reviews_number) %>%
  head() %>% 
  knitr::kable(format = "markdown")
```

<table style="width:100%;">
<colgroup>
<col width="3%" />
<col width="65%" />
<col width="16%" />
<col width="15%" />
</colgroup>
<thead>
<tr class="header">
<th align="right">id</th>
<th align="left">all_reviews</th>
<th align="left">all_reviews_category</th>
<th align="right">all_reviews_number</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="right">1</td>
<td align="left">Very Positive,(42,550),- 92% of the 42,550 user reviews for this game are positive.</td>
<td align="left">Very Positive</td>
<td align="right">3</td>
</tr>
<tr class="even">
<td align="right">2</td>
<td align="left">Mixed,(836,608),- 49% of the 836,608 user reviews for this game are positive.</td>
<td align="left">Mixed</td>
<td align="right">0</td>
</tr>
<tr class="odd">
<td align="right">3</td>
<td align="left">Mostly Positive,(7,030),- 71% of the 7,030 user reviews for this game are positive.</td>
<td align="left">Mostly Positive</td>
<td align="right">1</td>
</tr>
<tr class="even">
<td align="right">4</td>
<td align="left">Mixed,(167,115),- 61% of the 167,115 user reviews for this game are positive.</td>
<td align="left">Mixed</td>
<td align="right">0</td>
</tr>
<tr class="odd">
<td align="right">5</td>
<td align="left">Mostly Positive,(11,481),- 74% of the 11,481 user reviews for this game are positive.</td>
<td align="left">Mostly Positive</td>
<td align="right">1</td>
</tr>
<tr class="even">
<td align="right">6</td>
<td align="left">NaN</td>
<td align="left">NaN</td>
<td align="right">NA</td>
</tr>
</tbody>
</table>

*3.Graphing*

Now that the tibble was correct for graphing and the reviews were converted to a numerical value, the plot could be done and I could observe the differences between all of the categories.

``` r
steam_games_q1 %>% 
  filter(!is.na(all_reviews_number), #Remove the games with no review category
         !is.na(release_date_category)) %>% #Remove the games with an inconsistent datetime element
  ggplot(aes(x = release_date_category, y = all_reviews_number, fill = release_date_category)) +
  geom_boxplot() + 
  geom_jitter(alpha = 0.15, width = 0.1)+ 
  stat_summary(fun = mean, colour = "white")+ #Add the mean as a white dot in the barplot
  theme_classic() + 
  ylab("Review score")+ #Change the y lab 
  xlab("Release date") +
  guides(fill="none") #Remove the fill legends (non necessary)
```

![](mda_milestone_2_files/figure-markdown_github/unnamed-chunk-7-1.png)

This task was helpful to answer my question (*Do recent released games have better reviews than the older ones?*) because, by dividing the games acoording to their release date in periods of 10 years, I could observe that there is a trend in old games to have better overall reviews than intermediate and new ones (which was contrary to what I was expecting). However, a further statistical analysis is needed to make a conclusion.

#### Q2: Are games with better reviews the most expensive ones?

> Summarizing task: Compute the *range*, *mean*, and *two other summary statistics* of **one numerical variable** across the groups of **one categorical variable** from your data.\*

For this research question, I calculated the summary statistics of the price per review category, to explore the distribution of prices. Since I needed a column that contains the review category of each game, which was already done in the previous question, I used `steam_games_q1` as my starting dataset.

``` r
steam_games_q1 %>% 
  filter(!is.na(original_price), #Remove the games that have NA in the original price
         all_reviews_category %in% (steam_games_q1$all_reviews_category %>% 
                                      table() %>% 
                                      names())[1:9]) %>% #Remove the games that did not have enough reviews to have an overall review (discussed in Q1 - Extract vategories)
  group_by(all_reviews_category) %>% 
  summarise(mean_price = mean(original_price), #Compute the summary statistics
            std_price = sd(original_price),
            median_price = median(original_price),
            min_price = min(original_price),
            max_price = max(original_price)) %>% 
  knitr::kable(format = "markdown")
```

| all\_reviews\_category  |  mean\_price|    std\_price|  median\_price|  min\_price|  max\_price|
|:------------------------|------------:|-------------:|--------------:|-----------:|-----------:|
| Overwhelmingly Positive |    14.886226|     30.185184|           9.99|        0.00|      501.87|
| Very Positive           |    15.014226|     35.954507|           9.99|        0.00|      501.87|
| Positive                |   201.936467|  11061.335331|           4.99|        0.00|   650560.00|
| Mostly Positive         |    16.436775|     45.270922|           8.99|        0.00|      624.74|
| Mixed                   |    16.025741|     51.907027|           5.99|        0.00|      624.74|
| Mostly Negative         |    12.190993|     36.675383|           4.99|        0.00|      624.74|
| Negative                |    10.335985|     17.196160|           4.99|        0.00|      110.61|
| Very Negative           |    13.148387|     12.300908|           9.99|        0.99|       59.99|
| Overwhelmingly Negative |     9.132857|      5.843189|           9.99|        0.99|       19.99|

> Plotting task: Create a graph of your choosing, make one of the axes logarithmic, and format the axes labels so that they are "pretty" or easier to read

To have a visual representation of the distribution of the variable that was summarized above (price) in each category and compare them, I decided to use the function `geom_density_ridges()`

``` r
steam_games_q1 %>% 
  filter(!is.na(original_price), #Remove the games that have NA in the original price
         all_reviews_category %in% (steam_games_q1$all_reviews_category %>% 
                                      table() %>% 
                                      names())[1:9]) %>% #Remove the games that did not have enough reviews to have an overall review (discussed in Q1 - Extract categories) 
  mutate(original_price = original_price + 1 ,
         all_reviews_category = fct_rev(all_reviews_category)) %>% #Add 1 to the price to avoid infinite values when applying the log10 transformation
  ggplot(aes(original_price, all_reviews_category))+
  ggridges::geom_density_ridges(aes(fill = all_reviews_category), alpha = 0.4) +
  scale_x_log10(labels = scales::label_dollar())+ #Add a log10 scale to the x axis
  scale_y_discrete("")+ #Remove the y axis title (non necessary)
  xlab("Original price")+
  guides(fill = "none") 
```

![](mda_milestone_2_files/figure-markdown_github/unnamed-chunk-9-1.png)

This task provided information that helped me to explore my question. By looking at the density of each group, I can see that, overall, they all have a very similar distribution around similar prices. There are some categories, such as Overwhelmingly Negative, that have a slightly different shape, and a strong peak. I wonder if those differences are enough for its prices to be statistically different compared to any of the other categories when doing a proper statistical test.

#### Q3: Are the most popular games (i.e. games with more reviews) the ones with the best overall review?

> Summarizing task: Compute the *range*, *mean*, and *two other summary statistics* of **one numerical variable** across the groups of **one categorical variable** from your data

Dor this research question, I decided to calculate the summary statistics of the number of reviews per each review category. Exploring the distribution of the number of reviews per category, which I will use as a proxy of popularity, can help me to answer this question.

Since I needed a column that contained the review category of each game, which was already done in the research question 1, I used `steam_games_q1` as my starting point.

The first step then, was to extract the number of reviews per game, which was embedded in the column all\_reviews, as it can be observed here:

``` r
head(steam_games_q1$all_reviews)
```

    ## [1] "Very Positive,(42,550),- 92% of the 42,550 user reviews for this game are positive."  
    ## [2] "Mixed,(836,608),- 49% of the 836,608 user reviews for this game are positive."        
    ## [3] "Mostly Positive,(7,030),- 71% of the 7,030 user reviews for this game are positive."  
    ## [4] "Mixed,(167,115),- 61% of the 167,115 user reviews for this game are positive."        
    ## [5] "Mostly Positive,(11,481),- 74% of the 11,481 user reviews for this game are positive."
    ## [6] "NaN"

To get the number of reviews and make it a new independent column, I could not use the same approach that I used in the question 1 to extract the review categories, because the numeric values inside the brackets are separated by commas. Then, using commas to separate columns would lead to the review number being split. Therefore, I had to extract the element inside the brackets, and then coerce it into a numeric type.

``` r
steam_games_q1 = (steam_games_q1 %>%
                    mutate(popularity = sub(").*", "",all_reviews),#Create a new column removing all of the characters after the ")" bracket in all_reviews
                           popularity = sub(".*\\(","", popularity), #Remove all of the characters before the "(" bracket of the popularity column
                           popularity = sub(",","", popularity),  #Eliminate the commas that separate the numbers so that I can coerce them to numeric
                           popularity = as.numeric(popularity)) #Coerce the values to numeric
                  ) 

#Check that the final tibble is correct
steam_games_q1 %>% 
  select(id, all_reviews, popularity) %>% 
  head() %>% 
  knitr::kable(format = "markdown")
```

<table style="width:100%;">
<colgroup>
<col width="3%" />
<col width="84%" />
<col width="11%" />
</colgroup>
<thead>
<tr class="header">
<th align="right">id</th>
<th align="left">all_reviews</th>
<th align="right">popularity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="right">1</td>
<td align="left">Very Positive,(42,550),- 92% of the 42,550 user reviews for this game are positive.</td>
<td align="right">42550</td>
</tr>
<tr class="even">
<td align="right">2</td>
<td align="left">Mixed,(836,608),- 49% of the 836,608 user reviews for this game are positive.</td>
<td align="right">836608</td>
</tr>
<tr class="odd">
<td align="right">3</td>
<td align="left">Mostly Positive,(7,030),- 71% of the 7,030 user reviews for this game are positive.</td>
<td align="right">7030</td>
</tr>
<tr class="even">
<td align="right">4</td>
<td align="left">Mixed,(167,115),- 61% of the 167,115 user reviews for this game are positive.</td>
<td align="right">167115</td>
</tr>
<tr class="odd">
<td align="right">5</td>
<td align="left">Mostly Positive,(11,481),- 74% of the 11,481 user reviews for this game are positive.</td>
<td align="right">11481</td>
</tr>
<tr class="even">
<td align="right">6</td>
<td align="left">NaN</td>
<td align="right">NaN</td>
</tr>
</tbody>
</table>

After extracting the review numbers, I computed the summary statistics of the number of reviews (popularity) per category

``` r
steam_games_q1 %>% 
  filter(!is.na(popularity), #Remove the games that have NA or NaN in the number of reviews column
         all_reviews_category %in% (steam_games_q1$all_reviews_category %>% 
                                      table() %>% 
                                      names())[1:9]) %>% #Remove the games that did not have enough reviews to have an overall review (discussed in Q1 - Extract categories)
  group_by(all_reviews_category) %>% 
  summarise(mean_pop = mean(popularity), #Compute the summary statistics
            std_pop = sd(popularity),
            median_pop = median(popularity),
            min_pop = min(popularity),
            max_pop = max(popularity)) %>% 
  knitr::kable(format = "markdown")
```

| all\_reviews\_category  |    mean\_pop|     std\_pop|  median\_pop|  min\_pop|  max\_pop|
|:------------------------|------------:|------------:|------------:|---------:|---------:|
| Overwhelmingly Positive |  12966.71340|  31030.43172|         2413|       500|    310394|
| Very Positive           |   2319.68437|  13665.27086|          279|        50|    553458|
| Positive                |     23.11771|     10.91703|           20|        10|        49|
| Mostly Positive         |   1112.16490|   8761.78135|           82|        10|    407706|
| Mixed                   |    846.76795|  13464.48441|           49|        10|    836608|
| Mostly Negative         |    232.24297|   1164.27441|           31|        10|     22589|
| Negative                |     19.33333|     10.12607|           15|        10|        49|
| Very Negative           |    148.02703|    111.80476|          107|        51|       483|
| Overwhelmingly Negative |   1426.85714|    925.46915|         1096|       509|      3057|

> Graphing task: Create a graph out of summarized variables that has at least two geom layers & Create a graph of your choosing, make one of the axes logarithmic, and format the axes labels so that they are "pretty" or easier to read.

Finally, I plotted the popularity distribution of the games per category, marking the mean as an asterisk.

``` r
steam_games_q1 %>% 
  filter(!is.na(popularity), #Remove the games that have NA in the number of reviews
         all_reviews_category %in% (steam_games_q1$all_reviews_category %>% 
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

![](mda_milestone_2_files/figure-markdown_github/unnamed-chunk-13-1.png)

With these tasks, I could see that there is a difference in the popularity distribution of the games across different categories. Games with positive and negative reviews tend to be less popular than games with overwhelmingly positive or overwhelmingly negative results. I would be interested in doing a statistical test to evaluate each category and know which ones are statistically significantly different from others. For what I can see here, probably more than 2 categories could be different to at least 2 other groups.

#### Q4: Which developer produces the most popular games (i.e games with more reviews)?

> Summarizing task: Compute the *range*, *mean*, and *two other summary statistics* of **one numerical variable** across the groups of **one categorical variable** from your data & Compute the number of observations for at least one of your categorical variables

For this research question, I needed the popularity numbers of each game, which was obtained in the previous section (Q3). Therefore, I used `steam_games_q1` again as my starting point. Then, I computed the summary statistics of the popularity measure (number of reviews) across the different developers, as well as the number of games that each developer has.

*1. Summary statistics of popularity per developer*

``` r
summary_q4 = (steam_games_q1 %>% 
  filter(!is.na(popularity), #Remove the games that have NA or NaN in the number of reviews column
         all_reviews_category %in% (steam_games_q1$all_reviews_category %>% 
                                      table() %>% 
                                      names())[1:9]) %>% #Remove the games that did not have enough reviews to have an overall review (discussed in Q1 - Extract categories)
  group_by(developer) %>% 
  summarise(n_games = n(),
            mean_pop = mean(popularity), #Compute the summary statistics
            std_pop = sd(popularity),
            median_pop = median(popularity),
            min_pop = min(popularity),
            max_pop = max(popularity)) %>%
  arrange(desc(mean_pop)))

summary_q4 %>% 
  head() %>% 
  knitr::kable(format = "markdown")
```

| developer             |  n\_games|  mean\_pop|  std\_pop|  median\_pop|  min\_pop|  max\_pop|
|:----------------------|---------:|----------:|---------:|------------:|---------:|---------:|
| Smartly Dressed Games |         1|   325675.0|        NA|       325675|    325675|    325675|
| PUBG Corporation      |         3|   279593.0|  482389.5|         1686|       485|    836608|
| Re-Logic              |         1|   201266.0|        NA|       201266|    201266|    201266|
| Rockstar North        |         3|   136585.7|  234797.1|         1165|       886|    407706|
| Team Salvato          |         1|   113354.0|        NA|       113354|    113354|    113354|
| ConcernedApe          |         1|   104530.0|        NA|       104530|    104530|    104530|

> Graphing task: Create 3 histograms out of summarized variables, with each histogram having different sized bins. Pick the ???best??? one and explain why it is the best & Create a graph of your choosing, make one of the axes logarithmic, and format the axes labels so that they are ???pretty??? or easier to read.

When looking at the head of the table, it can be observed that some developers have few games, which makes the summary statistics impossible to compute or non-informative. After ploting the histogram and watching that several games have 3 or more games, I decided to drop the rows that have less than 3.

I also explored the histograms using different bins, and picked the one with 11, since it is smooth and represents the distribution of the data accurrately. The cutoff that I used can be observed as a vertical red line in each histogram.

``` r
summary_q4 %>% 
  ggplot(aes(x = n_games))+
  geom_histogram(bins = 30, fill = "blue", alpha = 0.5)+
  xlab("Number of games")+
  ylab("Frequency")+
  scale_x_log10()+
  geom_vline(xintercept = 3, color = "red")+
  theme_classic()
```

![](mda_milestone_2_files/figure-markdown_github/unnamed-chunk-15-1.png)

``` r
summary_q4 %>% 
  ggplot(aes(x = n_games))+
  geom_histogram(bins = 20,fill = "blue", alpha = 0.5)+
  xlab("Number of games")+
  ylab("Frequency")+
  scale_x_log10()+
  geom_vline(xintercept = 3, color = "red")+
  theme_classic()
```

![](mda_milestone_2_files/figure-markdown_github/unnamed-chunk-15-2.png)

``` r
summary_q4 %>% 
  ggplot(aes(x = n_games))+
  geom_histogram(bins =11,fill = "blue", alpha = 0.5)+
  xlab("Number of games")+
  ylab("Frequency")+
  scale_x_log10()+
  theme_classic()+
  geom_vline(xintercept = 3, color = "red")
```

![](mda_milestone_2_files/figure-markdown_github/unnamed-chunk-15-3.png)

> Graphing task: Create a graph out of summarized variables that has at least two geom layers

Finaly, after observing that the cutoff still left games for my analysis, I plotted the mean popularity of the top 20 reviewed developers, with the standard deviation in the error bars to have an idea of the data dispersion.

``` r
summary_q4 %>% 
  filter(n_games >= 3 ) %>% 
  head(20) %>% 
  mutate(developer = as.factor(developer),
         developer = fct_reorder(developer, mean_pop, min)) %>% 
  ggplot(aes(x = developer,y = mean_pop, fill = developer)) +
  geom_errorbar(aes(ymin = mean_pop, ymax = mean_pop + std_pop), width=0.1)+ #I set the lower part of the standard deviation bar to the mean because it is too big, so it extends to negative numbers. I decided to hide it; just by looking at the error bar that goes up we can have an idea of the dispersion without modifying the y axis of the plot
  geom_col(alpha = 0.7)+
  scale_y_continuous(labels = scales::dollar_format(prefix = ""))+
  xlab("Developer")+
  ylab("Popularity")+
  coord_flip()+ 
  guides(fill = "none")+
  theme_classic()
```

![](mda_milestone_2_files/figure-markdown_github/unnamed-chunk-16-1.png)

These tasks were helpful for my research question because I could identify the developers that produce the top 20 most popular games. However, the dispersion in their games popularity is very high. Therefore, based on these samples, I wonder if we could say that one developer produced statistically signifcant more popular games than other developer. A statistical analysis that compares each pair of developers could help me answer that question in the future.

### 1.3 Check-point (2.5 points)

Based on the operations that I completed, I am closer to answer most of my research questions. For each of them, I produced plots that visually helped me to explore the variables that I was interested in and have an idea of what the results could look like. The next step in each of them would be to perform a statistical analysis to obtain significant conclusions, since only with plots, the statistical differences between categories remains unclear.

A more specific comment on each research question can be found at the bottom of each question section, in the previous task (1.2).

In conclussion, after observing the results of this milestone, the research questions that yielded the most interesting results were Q1, Q2 and Q3.

# Task 2: Tidy your data (12.5 points)

In this task, I was asked to reshape my data. I will work with the modified dataset that I produced throughout the task 1 (`steam_games_q1`).

By definition, tidy data has the following attributes:

-   Each row is an **observation**
-   Each column is a **variable**
-   Each cell is a **value**

### Exploring the tidyness of my data 2.1 (2.5 points)

Based on the definition provided above, I can say that my dataset is tidy for my specific questions, as it can be observed in the 8 variables I picked, according to their utility for my analysis.

In the displayed table, each row corresponds to a single observation (game), each column is a variable that I used for any of my questions, and each cell contains a single value.

``` r
task_2.1 = (steam_games_q1 %>% 
  select(id, release_date, developer, original_price, release_date_category, all_reviews_category, all_reviews_number, popularity))

task_2.1%>% 
  head(10) %>% 
  knitr::kable(format = "markdown")
```

<table style="width:100%;">
<colgroup>
<col width="3%" />
<col width="10%" />
<col width="15%" />
<col width="12%" />
<col width="17%" />
<col width="16%" />
<col width="15%" />
<col width="9%" />
</colgroup>
<thead>
<tr class="header">
<th align="right">id</th>
<th align="left">release_date</th>
<th align="left">developer</th>
<th align="right">original_price</th>
<th align="left">release_date_category</th>
<th align="left">all_reviews_category</th>
<th align="right">all_reviews_number</th>
<th align="right">popularity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="right">1</td>
<td align="left">2016-05-12</td>
<td align="left">id Software</td>
<td align="right">19.99</td>
<td align="left">New</td>
<td align="left">Very Positive</td>
<td align="right">3</td>
<td align="right">42550</td>
</tr>
<tr class="even">
<td align="right">2</td>
<td align="left">2017-12-21</td>
<td align="left">PUBG Corporation</td>
<td align="right">29.99</td>
<td align="left">New</td>
<td align="left">Mixed</td>
<td align="right">0</td>
<td align="right">836608</td>
</tr>
<tr class="odd">
<td align="right">3</td>
<td align="left">2018-04-24</td>
<td align="left">Harebrained Schemes</td>
<td align="right">39.99</td>
<td align="left">New</td>
<td align="left">Mostly Positive</td>
<td align="right">1</td>
<td align="right">7030</td>
</tr>
<tr class="even">
<td align="right">4</td>
<td align="left">2018-12-13</td>
<td align="left">Bohemia Interactive</td>
<td align="right">44.99</td>
<td align="left">New</td>
<td align="left">Mixed</td>
<td align="right">0</td>
<td align="right">167115</td>
</tr>
<tr class="odd">
<td align="right">5</td>
<td align="left">2003-05-06</td>
<td align="left">CCP</td>
<td align="right">0.00</td>
<td align="left">Intermediate</td>
<td align="left">Mostly Positive</td>
<td align="right">1</td>
<td align="right">11481</td>
</tr>
<tr class="even">
<td align="right">6</td>
<td align="left">NA</td>
<td align="left">Rockstar North</td>
<td align="right">NA</td>
<td align="left">NA</td>
<td align="left">NaN</td>
<td align="right">NA</td>
<td align="right">NaN</td>
</tr>
<tr class="odd">
<td align="right">7</td>
<td align="left">2019-03-07</td>
<td align="left">CAPCOM Co., Ltd.</td>
<td align="right">59.99</td>
<td align="left">New</td>
<td align="left">Very Positive</td>
<td align="right">3</td>
<td align="right">9645</td>
</tr>
<tr class="even">
<td align="right">8</td>
<td align="left">2016-07-22</td>
<td align="left">No Brakes Games</td>
<td align="right">14.99</td>
<td align="left">New</td>
<td align="left">Very Positive</td>
<td align="right">3</td>
<td align="right">23763</td>
</tr>
<tr class="odd">
<td align="right">9</td>
<td align="left">2017-12-12</td>
<td align="left">Numantian Games</td>
<td align="right">29.99</td>
<td align="left">New</td>
<td align="left">Very Positive</td>
<td align="right">3</td>
<td align="right">12127</td>
</tr>
<tr class="even">
<td align="right">10</td>
<td align="left">2019-05-31</td>
<td align="left">Eko Software</td>
<td align="right">49.99</td>
<td align="left">New</td>
<td align="left">Mixed</td>
<td align="right">0</td>
<td align="right">904</td>
</tr>
</tbody>
</table>

### 2.2 Untidying and tidying the data (5 points)

*Untidying the data*

After observing that my data is tidy, I proceeded to untidy it, which can be seen in the following table. I decided to make a wider dataset by making each review category an independent column, with the popularity value as their content.

As it can be observed, the dataset is untidy since not all of the columns correspond to a single variable for my research questions (specially for the research questions 2 and 3 that evaluate the category as a variable), and each cell is not a useful value.

``` r
task_2.1_untid = (task_2.1 %>% 
  pivot_wider(id_cols = -c(all_reviews_category, popularity),
              names_from = all_reviews_category,
              values_from = popularity))

task_2.1_untid %>% 
  head() %>% 
  knitr::kable(format = "markdown")
```

<table style="width:100%;">
<colgroup>
<col width="1%" />
<col width="3%" />
<col width="5%" />
<col width="4%" />
<col width="5%" />
<col width="5%" />
<col width="3%" />
<col width="2%" />
<col width="4%" />
<col width="1%" />
<col width="6%" />
<col width="4%" />
<col width="2%" />
<col width="1%" />
<col width="4%" />
<col width="4%" />
<col width="4%" />
<col width="4%" />
<col width="4%" />
<col width="4%" />
<col width="4%" />
<col width="4%" />
<col width="4%" />
<col width="2%" />
<col width="3%" />
<col width="6%" />
</colgroup>
<thead>
<tr class="header">
<th align="right">id</th>
<th align="left">release_date</th>
<th align="left">developer</th>
<th align="right">original_price</th>
<th align="left">release_date_category</th>
<th align="right">all_reviews_number</th>
<th align="right">Very Positive</th>
<th align="right">Mixed</th>
<th align="right">Mostly Positive</th>
<th align="right">NaN</th>
<th align="right">Overwhelmingly Positive</th>
<th align="right">7 user reviews</th>
<th align="right">Positive</th>
<th align="right">NA</th>
<th align="right">1 user reviews</th>
<th align="right">Mostly Negative</th>
<th align="right">5 user reviews</th>
<th align="right">3 user reviews</th>
<th align="right">2 user reviews</th>
<th align="right">4 user reviews</th>
<th align="right">9 user reviews</th>
<th align="right">6 user reviews</th>
<th align="right">8 user reviews</th>
<th align="right">Negative</th>
<th align="right">Very Negative</th>
<th align="right">Overwhelmingly Negative</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="right">1</td>
<td align="left">2016-05-12</td>
<td align="left">id Software</td>
<td align="right">19.99</td>
<td align="left">New</td>
<td align="right">3</td>
<td align="right">42550</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
</tr>
<tr class="even">
<td align="right">2</td>
<td align="left">2017-12-21</td>
<td align="left">PUBG Corporation</td>
<td align="right">29.99</td>
<td align="left">New</td>
<td align="right">0</td>
<td align="right">NA</td>
<td align="right">836608</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
</tr>
<tr class="odd">
<td align="right">3</td>
<td align="left">2018-04-24</td>
<td align="left">Harebrained Schemes</td>
<td align="right">39.99</td>
<td align="left">New</td>
<td align="right">1</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">7030</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
</tr>
<tr class="even">
<td align="right">4</td>
<td align="left">2018-12-13</td>
<td align="left">Bohemia Interactive</td>
<td align="right">44.99</td>
<td align="left">New</td>
<td align="right">0</td>
<td align="right">NA</td>
<td align="right">167115</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
</tr>
<tr class="odd">
<td align="right">5</td>
<td align="left">2003-05-06</td>
<td align="left">CCP</td>
<td align="right">0.00</td>
<td align="left">Intermediate</td>
<td align="right">1</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">11481</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
</tr>
<tr class="even">
<td align="right">6</td>
<td align="left">NA</td>
<td align="left">Rockstar North</td>
<td align="right">NA</td>
<td align="left">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NaN</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
</tr>
</tbody>
</table>

*Tidying back the data*

Then, I made it tidy again by reverting the changes I made in the previous chunk code.

``` r
task_2.1_untid %>% 
  pivot_longer(cols = -c(id, release_date, original_price, developer, original_price, release_date_category, all_reviews_number),
               names_to = "all_reviews_category", 
               values_to = "popularity") %>% 
  filter(!is.na(popularity)) %>% #Remove the NAs that were generated when making the dataset wider
  head() %>% 
  knitr::kable(format = "markdown")
```

<table style="width:100%;">
<colgroup>
<col width="3%" />
<col width="10%" />
<col width="15%" />
<col width="12%" />
<col width="17%" />
<col width="15%" />
<col width="16%" />
<col width="9%" />
</colgroup>
<thead>
<tr class="header">
<th align="right">id</th>
<th align="left">release_date</th>
<th align="left">developer</th>
<th align="right">original_price</th>
<th align="left">release_date_category</th>
<th align="right">all_reviews_number</th>
<th align="left">all_reviews_category</th>
<th align="right">popularity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="right">1</td>
<td align="left">2016-05-12</td>
<td align="left">id Software</td>
<td align="right">19.99</td>
<td align="left">New</td>
<td align="right">3</td>
<td align="left">Very Positive</td>
<td align="right">42550</td>
</tr>
<tr class="even">
<td align="right">2</td>
<td align="left">2017-12-21</td>
<td align="left">PUBG Corporation</td>
<td align="right">29.99</td>
<td align="left">New</td>
<td align="right">0</td>
<td align="left">Mixed</td>
<td align="right">836608</td>
</tr>
<tr class="odd">
<td align="right">3</td>
<td align="left">2018-04-24</td>
<td align="left">Harebrained Schemes</td>
<td align="right">39.99</td>
<td align="left">New</td>
<td align="right">1</td>
<td align="left">Mostly Positive</td>
<td align="right">7030</td>
</tr>
<tr class="even">
<td align="right">4</td>
<td align="left">2018-12-13</td>
<td align="left">Bohemia Interactive</td>
<td align="right">44.99</td>
<td align="left">New</td>
<td align="right">0</td>
<td align="left">Mixed</td>
<td align="right">167115</td>
</tr>
<tr class="odd">
<td align="right">5</td>
<td align="left">2003-05-06</td>
<td align="left">CCP</td>
<td align="right">0.00</td>
<td align="left">Intermediate</td>
<td align="right">1</td>
<td align="left">Mostly Positive</td>
<td align="right">11481</td>
</tr>
<tr class="even">
<td align="right">7</td>
<td align="left">2019-03-07</td>
<td align="left">CAPCOM Co., Ltd.</td>
<td align="right">59.99</td>
<td align="left">New</td>
<td align="right">3</td>
<td align="left">Very Positive</td>
<td align="right">9645</td>
</tr>
</tbody>
</table>

As it can be observed, the dataset is tidy again since each column is a single variable that is used in any of my research questions. It is interesting to observe that, even though it is very similar to the original dataset (table in task 2.1), it is not the same. This happened because I had to filter the NAs that were introduced in the dataset when I made it wider and, since some games already had a NA or NaN in their popularity, they were removed as well in that step. However, I made the same filters for question 2 and 3. Therefore, that can not be seen as a lost of information because for the analysis steps where I used the popularity variable, NaNs and NAs were also filtered out. However, some games that were used in the other questions might have been deleted when reshaping the data twice.

### 2.3 Choosing the final 2 research questions (5 points)

After becoming more familiar with my data and making progress in my research questions, I chose the following ones:

1.  *Do recent released games have better reviews than the older ones? (Q1)*. I chose it because I found very interesting the trend of old video games to have better reviews than new ones, at least visually (which is contrary to wha I was expecting). I would like to test that hypothesis in the future and explore its results to know if the progress that has been made lately in developing videogames is associated to actually improving the gaming experience of the users.

2.  *Are the most popular games the ones with the best overall review? (Q3)*. I chose it because I found interesting the distribution of the number of reviews per category. Just by looking at the produced plot, I would say that it is probable to find statistical differences among some categories. I would like to explore that in the future and know if the review of a game has an impact on the number of people that play it or not.

To answer those questions in milestone 3, I will use the make use of the `steam_games_q1` dataset, which was produced along the whole assignment. It would be repetitive en redundant to write all the code that I did here again, but all the steps for filtering, creating new columns, extracting embedded values, cleaning and tidying can be clearly seen in the task 1 across the 4 research question tasks (which altogether involve more than 4 functions). Here, as a final step, I will only drop the columns that have no potential use in my 2 chosen research questions.

``` r
steam_games_mda2_final = (
  steam_games_q1 %>% 
    select(id, name, release_date, release_date_category, all_reviews, all_reviews_category, all_reviews_number, popularity)
)

head(steam_games_mda2_final) %>% 
  knitr::kable(format = "markdown")
```

<table style="width:100%;">
<colgroup>
<col width="1%" />
<col width="19%" />
<col width="6%" />
<col width="10%" />
<col width="38%" />
<col width="9%" />
<col width="8%" />
<col width="5%" />
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
<td align="right">1</td>
<td align="left">DOOM</td>
<td align="left">2016-05-12</td>
<td align="left">New</td>
<td align="left">Very Positive,(42,550),- 92% of the 42,550 user reviews for this game are positive.</td>
<td align="left">Very Positive</td>
<td align="right">3</td>
<td align="right">42550</td>
</tr>
<tr class="even">
<td align="right">2</td>
<td align="left">PLAYERUNKNOWN'S BATTLEGROUNDS</td>
<td align="left">2017-12-21</td>
<td align="left">New</td>
<td align="left">Mixed,(836,608),- 49% of the 836,608 user reviews for this game are positive.</td>
<td align="left">Mixed</td>
<td align="right">0</td>
<td align="right">836608</td>
</tr>
<tr class="odd">
<td align="right">3</td>
<td align="left">BATTLETECH</td>
<td align="left">2018-04-24</td>
<td align="left">New</td>
<td align="left">Mostly Positive,(7,030),- 71% of the 7,030 user reviews for this game are positive.</td>
<td align="left">Mostly Positive</td>
<td align="right">1</td>
<td align="right">7030</td>
</tr>
<tr class="even">
<td align="right">4</td>
<td align="left">DayZ</td>
<td align="left">2018-12-13</td>
<td align="left">New</td>
<td align="left">Mixed,(167,115),- 61% of the 167,115 user reviews for this game are positive.</td>
<td align="left">Mixed</td>
<td align="right">0</td>
<td align="right">167115</td>
</tr>
<tr class="odd">
<td align="right">5</td>
<td align="left">EVE Online</td>
<td align="left">2003-05-06</td>
<td align="left">Intermediate</td>
<td align="left">Mostly Positive,(11,481),- 74% of the 11,481 user reviews for this game are positive.</td>
<td align="left">Mostly Positive</td>
<td align="right">1</td>
<td align="right">11481</td>
</tr>
<tr class="even">
<td align="right">6</td>
<td align="left">Grand Theft Auto V: Premium Online Edition</td>
<td align="left">NA</td>
<td align="left">NA</td>
<td align="left">NaN</td>
<td align="left">NaN</td>
<td align="right">NA</td>
<td align="right">NaN</td>
</tr>
</tbody>
</table>

Finally, I saved the final dataset so that it is ready for use in the following milestone.

``` r
saveRDS(steam_games_mda2_final, file = here("Milestone_2","steam_games_mda2_final"))
```

### Attribution

Thanks to Victor Yuan for mostly putting the instruction document together.
