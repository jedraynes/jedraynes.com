---
title: "Developing Data Products Week 2 Project 1 - Minimum Wage: 1968 to 2019 Actual vs. Inflation Adjusted"
linktitle: "Developing Data Products Week 2 Project 1 - Minimum Wage: 1968 to 2019 Actual vs. Inflation Adjusted"
author: "Jed Raynes"
date: 2021-04-01
type:
- post 
- posts
weight: 10
# series:
# - Hugo 101
aliases:
- /posts/Minimum Wage-1968-to-2019-Actual-vs.-Inflation-Adjusted
---

# Minimum Wage: 1968 to 2019 Actual vs. Inflation Adjusted

### # Overview and Sources

---

**Created:** April 1, 2021

The dataset used in this project was initially scraped from the US Department of Labor. Note, this dataset uses the minimum wage (and takes the lower value in years where there are multiple) which represents the phase-in amounts when the law was enacted to include new populations.


### # Data Wrangling

---

I'll start by loading in the required packages. For this project, I'm using 'dplyr' to clean the data, 'plotly' to layer in the interactivity, 'ggplot2' to actually create the plot, 'data.table' to use the copy function, 'reshape2' to melt the dataframe, and 'htmlwidgets' and 'htmltools' to save the plot down for the website.


```r
# load libraries
suppressPackageStartupMessages(pacman::p_load(dplyr,
                                              plotly,
                                              ggplot2,
                                              data.table, 
                                              reshape2, 
                                              htmlwidgets, 
                                              htmltools))
```

Next, I'll load and inspect the data.


```r
# set the working directory
setwd('C:\\Users\\Jed\\iCloudDrive\\Documents\\Learn\\R\\Johns Hopkins Data Science Specialization\\9 Developing Data Products\\Week 3')

# load data
inflage <- read.csv('inflage.csv')[, c(-1, -3)]
inflage_copy <- copy(inflage)

# inspect
head(inflage)
```

```
##   year min_wage adjusted_min_wage
## 1 1968     1.15          8.869974
## 2 1969     1.15          8.496239
## 3 1970     1.30          9.045455
## 4 1971     1.30          8.590910
## 5 1972     1.60         10.238988
## 6 1973     1.60          9.878460
```


To get the data organized for plotting, I'll rename the columns and unpivot the wage columns.


```r
# rename columns
names(inflage) <- c('Year', 'Minimum Wage, Actual', 'Minimum Wage, Inflation Adjusted')

# unpivot
inflage <- inflage %>% 
  melt(id.var = 'Year')
```

### # Plot

---

<iframe seamless src="/plotly/inflage_plot.html" width="100%" height="500"></iframe>

