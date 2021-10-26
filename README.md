# Mini Data Analysis Project
This project is part of the [STAT545A course](https://stat545.stat.ubc.ca) from UBC; its goal is to provide some practical experience in a self-guided mini data analysis, using an almost real dataset.

It is divided in three stages (called milestones); each one of them is intended to guide the students through the exploration of the dataset, formulation and refinement of potential research questions, as well as answering them using statistical methods. 

## Content
There are 4 folders in the repository; 3 of them (the ones corresponding to a milestone) contain mainly a .Rmd file that can be run in Rstudio, and a .md file, which is intended to be viewed here in github; each of them correspond to one stage in this project. The last folder contains files produced in the third milestone. 

### Folder Description
1. [Milestone 1](https://github.com/stat545ubc-2021/mda-ErickNavarroD/tree/main/Milestone_1). This is the starting stage  of the project, and comprises mainly the exploration of the available datasets for the assignment, their variables and their relationships. At the end, I chose 1 dataset for the following stages (`steam_games()`), for which I formulated 4 potential research questions that could be solved with the variables. The .md version (the knited file) can be found [here](https://github.com/stat545ubc-2021/mda-ErickNavarroD/blob/main/Milestone_1/mda_milestone1.md). The research questions formulated were: 
   - Q1. Do recent released games have better reviews than the older ones?
   - Q2. Are games with better reviews the most expensive ones?
   - Q3. Do games with a better review score tend to be the most popular? 
   - Q4. Which developer produces the most popular games (i.e games with more reviews) or the ones with the best reviews?
2. [Milestone 2](https://github.com/stat545ubc-2021/mda-ErickNavarroD/tree/main/Milestone_2). In this stage, I manipulated, summarized and made plots of the variables in the `steam_games()` dataset to answer the questions that I formulated in the previous step. After observing the results and, I narrowed my research questions to only 2 research questions that were the most interesting to me, and finished with a tidy dataset that can be used to solve them in the next stage. The .md version can be found [here](https://github.com/stat545ubc-2021/mda-ErickNavarroD/blob/main/Milestone_2/mda_milestone_2.md). The chosen research questions were: 
    - Q1: Do recent released games have better reviews than the older ones?
    - Q2: Do games with a better review score tend to be the most popular? 
3. [Milestone 3](https://github.com/stat545ubc-2021/mda-ErickNavarroD/tree/main/Milestone_3). In this stage, I manipulated special data types in R, such as factors and dates and times to produce new plots that sharpened my results. Also, I fitted a model object to my data to get statistical results for the question *Do games with a better review score tend to be the most popular?*, and extracted the result. This model, and a summarized table were saved in the output file. The .md version can be found [here](https://github.com/stat545ubc-2021/mda-ErickNavarroD/blob/main/Milestone_3/mda_milestone_3.md)
4. [Output](). It contains two files: the model object produced in Milestone 3 to answer the question *Do games with a better review score tend to be the most popular?*, and a summarized table with variables of interest for it. 

> Note: the milestone folders contain figures that were produced in the .Rmd files, which are used for displaying the .md file.
