---
title: "MSBA OPIM 602 Project 1 - Capital Bikeshare Multiple Linear Regression"
linktitle: "MSBA OPIM 602 Project 1 - Capital Bikeshare Multiple Linear Regression"
author: "Jed Raynes"
date: 2021-11-09
type:
- post 
- posts
weight: 10
# series:
# - Hugo 101
aliases:
- /posts/OPIM-602-Project-1-Capital-Bikeshare-Multiple-Linear-Regression
---

# Capital Bikeshare Multiple Linear Regression

### # Introduction

---

**Created:** November 9, 2021

This webpage is my project submission for the Project 1 of OPIM 602 (Machine Learning I) in the MSBA program at the Georgetown University McDonough School of Business. 

### # Introduction

---

Capital Bikeshare (the “Company”) is a micro-transit, bicycle-sharing system that services the Washington, DC, Virginia, and Maryland areas. The Company offers single trip, day trip, and annual memberships. This paper analyzes the effects of collected predictors from 2011 and 2012 to determine the most impactful drivers of bike rental demand. Through the analysis detailed below, it is evident that bike rental demand is primarily driven by temperature, humidity, and certain hours of the day.

### # Pre-Modeling Approach

---

#### *Process and Assumptions*

To begin, we'll load in the data used in our analysis.

```r
# load data
bs_data <- read_excel("Capital Bike Sharing Dataset.xlsx")
```
For the purposes of the analysis, a multiple linear regression model was used and an alpha value of 0.05 was used as the benchmark to determine statistical significance. Though some predictors (e.g., hour of the day, season, etc.) were provided as numbers, I assumed that these are intended to be treated as factored levels as they are limited in range (e.g., 0 to 23, 1 to 4, etc.), categorical in nature, and will not take decimal places during predictions. The categorical predictors were converted to dummy variables (i.e., a column for each category and a 1 or 0 to specify if it matches that category) to identify specific predictors within categories. The model predictions are used to determine aggregate demand. In other words, this is demand for both casual and registered users. Data was separated into a training set, consisting approximately 70% of data, and a testing set, consisting of the remaining data points. The constructed model is based on the training set whereas the reported model metrics are based on the testing set.

My exploration of the data types is outline in the code block below.

```r
# understanding data types

# categorical
## season, mnth, hr, holiday, weekday, workingday, weathersit

# quantitative
## temp, atemp, hum, windspeed, cnt

# ignored variables
## instant, dteday, yr, casual, resgistered
```

I then separated the variables into a training and testing set using simple random sampling with a 70% / 30% split.

```r
# split into training and testing sets (70/30)

# define the index
index <- sample(1:nrow(bs_data),
                round(nrow(bs_data) * 0.70))

# split into train / test
bs_data_train <- bs_data[index, ]
bs_data_test <- bs_data[-index, ]
```

I then explored for missing values which showed that the data was not missing data and thus imputation was not required prior to the model fitting.

```r
# inspect for missing values
colSums(is.na(bs_data_train))
```

Given my exploration of the data types, I converted some variables to factor variables, as they were categorical in nature, and removed the unnecessary columns.

```r
# subset for relevant columns and convert categorical columns to factors

# training set
bs_data_train <- bs_data_train %>%
  mutate(season = factor(season),
         mnth = factor(mnth),
         hr = factor(hr),
         holiday = factor(holiday),
         weekday = factor(weekday),
         workingday = factor(workingday),
         weathersit = factor(weathersit)) %>%
  dplyr::select(season, mnth, hr, holiday, weekday, workingday, weathersit, temp, atemp, hum, windspeed, cnt)
```

I then converted factor variables to dummies to assist in the modeling process.

```r
# convert factor variables to dummies

# training set
bs_data_train_0 <- dummy_cols(bs_data_train,
                              remove_first_dummy = FALSE,
                              remove_selected_columns =  TRUE)
```



#### *Excluded Predictors*

Certain predictors were intentionally excluded from the analysis. These include the following: “instant” (removed as it is an identifier for each measurement), “dteday” (as this field is specific to the day, month, and year which would differ in future predictions, “yr” (as this field is a binary classifier signifying either 2011 or 2012, which would not apply to future predictions), and “casual” / “registered” (as we are predicting aggregate demand). Throughout the rest of the analysis, any reference to the predictors means the original set of predictors less those identified above, and any incremental exclusions as identified below.

#### *Collinearity*

Certain predictors were intentionally excluded from the analysis. These include the following: “instant” (removed as it is an identifier for each measurement), “dteday” (as this field is specific to the day, month, and year which would differ in future predictions, “yr” (as this field is a binary classifier signifying either 2011 or 2012, which would not apply to future predictions), and “casual” / “registered” (as we are predicting aggregate demand). Throughout the rest of the analysis, any reference to the predictors means the original set of predictors less those identified above, and any incremental exclusions as identified below.

```r
# ~~ ggplot method if using all predictors ~~
# inspect co-linearity

# subset for only numeric values
bs_data_train_0_numeric <- bs_data_train_0 %>%
  dplyr::select(where(is.numeric))

# calculate the correlations and melt the data frame
bs_data_train_numeric_cor <- melt(cor(bs_data_train_0_numeric))

# plot the heat
bs_data_train_numeric_cor %>%
  ggplot() + 
  geom_tile() +
  aes(x = Var1, y = Var2, fill = value) +
  labs(title = "Correlation Matrix",
       subtitle = "Numeric Predictors | 2011-2012",
       caption = "Source: Capital Bikeshare Data",
       x = NULL,
       y = NULL,
       fill = "Correlation") +
  #ggplot_theme +
  scale_fill_distiller(palette = "RdBu", direction = 1) +
  theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust = 1))
```

<div style="text-align:center"><img src="/images/OPIM602_Project1/correlation_matrix.png" /></div>


```r
# ~~ ggpairs method if using actual numeric predictors ~~
# inspect co-linearity

# subset for only numeric values
bs_data_train_numeric <- bs_data_train %>%
  dplyr::select(where(is.numeric))

# plot the heat
ggpairs(bs_data_train_numeric) + 
  labs(title = "Correlation Matrix",
       subtitle = "Numeric Predictors | 2011-2012",
       caption = "Source: Capital Bikeshare Data",
       x = NULL,
       y = NULL) +
  ggplot_theme + 
  theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust = 1))
```
<div style="text-align:center"><img src="/images/OPIM602_Project1/correlation_matrix_2.png" /></div>

As shown in the chart above, there is a strong, positive correlation between the “temp” and “atemp” predictors which also have a moderate, positive relationship with the dependent variable, “cnt” (with “temp” having a higher correlation). There are also moderate, negative correlations between “windspeed” and “hum” and “hum” and “cnt”. The diagonal of the plot displays the distribution density of the respective variable. The distribution of “cnt”, the dependent variable, is positively skewed which gives rise to the need for potential feature engineering. Additionally, the “workingday” predictor is a higher-level grouping of the “weekday” predictor. As a result, “atemp” and “workingday” were removed from our data set for model training.

```r
# remove atemp since temp is better correlated with the dependent variable
# remove workingday_0 and workingday_1 since these are highly correlated with dayof the week

# training set
bs_data_train_0 <- bs_data_train_0 %>%
  dplyr::select(-c(atemp, workingday_0, workingday_1))
```

### # Initial Model

---

#### *Predictors*

An initial model was constructed using all predictors before any feature engineering. A backward stepwise approach was used to derive the top three predictors of bike rentals. This resulted in a model with demand predicted by “temp” (normalized recorded temperature), “hr_17” (5:00 PM), and “hr_18” (6:00 PM).

```r
# ~~ initial model ~~

# backward stepwise
model_0_train <- train(cnt ~ .,
                       data = bs_data_train_0,
                       method = "leapBackward",
                       tuneGrid = data.frame(nvmax = 3))
```



```r
# inspect the summary of the backward stepwise
summary(model_0_train$finalModel)

model_0 <- lm(formula = cnt ~ temp + hr_17 + hr_18,
           data = bs_data_train_0)
```

This resulted in a three-variable model, given we are focusing on the top three factors, being the temperature, hour 17, and hour 18. 

#### *Linear Regression Assumptions and Influential Data Points – Initial Model*

There are four linear regression assumptions I tested: linearity, normality, heteroscedasticity, and multicollinearity. For the initial model, normality, and heteroscedasticity were not satisfied. The linearity portion was tested during our collinearity matrix analysis which suggested that there were moderate linear relationships that existed between the predictors and dependent variable.

<div style="text-align:center"><img src="/images/OPIM602_Project1/initial_model_assumptions.png" /></div>

The model did not satisfy the linear regression assumptions of normality and homoscedasticity. To test normality, the Anderson-Darling test was used on the residuals. To test homoscedasitcity / constant error variance, the Breusch-Pagan test was used on the residuals. There were no issues with multicollinearity when assessing VIF scores for the initial model.

```r
# ~~ test assumptions ~~

# Normality: Residuals are NOT normal
ad.test(model_0$residuals)

# Heteroscedasticity: HETEROSCEDASTIC, error variance is NOT constant
bptest(model_0)

# Multicollinearity: Vales are less than 10, there is NOT multicollinearity
vif(model_0)
```

In addition to testing the regression assumptions, I inspected the model for influential data points. Given provided data is assumed to be correct, I opted not to remove any influential observations. Refer to Appendix #1 for the initial model’s diagnostic plots.

### # Final Model

---

#### *Feature Engineering – Dependent Variable*

As done previously, the starting data set for the final model also excluded atemp and working day variables as they were deemed to have high levels of collinearity with other predictors.

```r
# remove atemp and working day fields
bs_data_train_1 <- bs_data_train %>%
  dplyr::select(-c(atemp, workingday))
```

The chart below displays the bike rentals as reported (top-left), natural log transformation (top-right), square root transformation (bottom-left), and cube root transformation (bottom-right).

```r
# explore the distribution of our data: cnt

# ~~ dependent variable visuals ~~

# pre-feature engineering
cnt_reported <- bs_data_train_0 %>%
  ggplot() + 
  aes(x = cnt) + 
  geom_histogram(color = "black",
                 fill = "steelblue") + 
  annotate("text", 
           x = 250, 
           y = 2000, 
           label = paste("Skewness:", round(skewness(bs_data_train_0$cnt), 2)),
           size = 3) + 
  labs(x = "Bike Rentals",
       y = "Count",
       subtitle = "Pre-Feature Engineering") +
  ggplot_theme

# log transformation
cnt_log <- bs_data_train_0 %>%
  ggplot() + 
  aes(x = log(cnt)) + 
  geom_histogram(color = "black",
                 fill = "steelblue") + 
  annotate("text",
           x = 2,
           y = 1000,
           label = paste("Skewness:", round(skewness(log(bs_data_train_0$cnt)), 2)),
           size = 3) +
  labs(x = "Bike Rentals (Log)",
       y = "Count",
       subtitle = "Log Transformation") +
  ggplot_theme

# square root transformation
cnt_sqrt <- bs_data_train_0 %>%
  ggplot() + 
  aes(x = sqrt(cnt)) + 
  geom_histogram(color = "black",
                 fill = "steelblue") + 
  annotate("text",
           x = 25,
           y = 750,
           label = paste("Skewness:", round(skewness(sqrt(bs_data_train_0$cnt)), 2)),
           size = 3) +
  labs(x = "Bike Rentals (Square Root)",
       y = "Count",
       subtitle = "Square Root Transformation") +
  ggplot_theme

# cube root transformation
cnt_cbrt <- bs_data_train_0 %>%
  ggplot() + 
  aes(x = nthroot(cnt, 3)) + 
  geom_histogram(color = "black",
                 fill = "steelblue") + 
  annotate("text",
           x = 3,
           y = 1000,
           label = paste("Skewness:", round(skewness(nthroot(bs_data_train_0$cnt, 3)), 2)),
           size = 3) +
  labs(x = "Bike Rentals (Cube Root)",
       y = "Count",
       subtitle = "Cube Root Transformation") +
  ggplot_theme

# ~~ cowplot ~~

# title
cnt_title <- ggdraw() + 
  draw_label("Bike Rental Feature Engineering Distributions",
             fontface = "bold",
             x = 0,
             hjust = -0.25) + 
  theme(plot.margin = margin (0, 0, 0, 7))

# caption
cnt_source <- ggdraw() + 
  draw_label("Source: Capital Bikeshare data",
             x = 0,
             hjust = 0,
             size = 10) + 
  theme(plot.margin = margin (0, 0, 0, 7))

# distribution plot grid
cnt_distributions <- plot_grid(cnt_reported,
                               cnt_log,
                               cnt_sqrt,
                               cnt_cbrt)

# final plot
plot_grid(cnt_title,
          cnt_distributions,
          cnt_source,
          ncol = 1,
          rel_heights = c(0.1, 1))
          
# ~~ data changes ~~

# convert cnt to cube root
bs_data_train_1 <-  bs_data_train_1 %>%
  mutate(cnt = nthroot(cnt, 3))
```

<div style="text-align:center"><img src="/images/OPIM602_Project1/dependent_variable_feature_engineering.png" /></div>

As reported, the bike rentals had a skewness factor of 1.28 showing the dependent variable is positively skewed. The log transformation shifted the skewness from 1.28 to -0.93, or negatively skewed. The square root and cube root transformations shifted the skewness from 1.28 to 0.29 and -0.08, respectively. Given the near-normal skewness of the cube root transformation, I transformed the dependent variable by taking its cube root.

#### *Feature Engineering – “Hr” Predictor*

I then visualized the bike rental demand by hour of the day (refer to Appendix #2). Based on the visual, there is an uptick in demand in certain hours (mainly commute times) and lower demand in other hours (such as early morning / late night). To better simplify the data, I binned the “hr” predictor into the three distinct bins of nearly equal rental demand: working hours, other hours, and commuting hours. The plot below displays the rental demand by hour bin along with a label noting which bin covers which hours.

```r
# explore the distribution of our data: hr

# ~~ hour variable visuals ~~

# distribution of the data
bs_data_train %>%
  ggplot() + 
  aes(x = hr) + 
  geom_histogram(color = "black",
                 fill = "steelblue",
                 stat = "count") + 
  labs(x = "Hour of the Day",
       y = "Count",
       title = "Distribution of Hour of the Day",
       subtitle = "2011-2012") +
  ggplot_theme
```

<div style="text-align:center"><img src="/images/OPIM602_Project1/hour_distribution_by_day.png" /></div>

```r  
# count by hour
bs_data_train %>%
  ggplot(aes(x = hr,
             y = cnt)) + 
  geom_bar(stat = "identity",
           fill = "steelblue") +
  labs(x = "Hour of the Day",
       y = "Count",
       title = "Bike Rental by Hour of the Day",
       subtitle = "2011-2012",
       caption = "Source: Capital Bikeshare Data") +
  scale_y_continuous(labels = function(x) format(x, big.mark = ",", scientific = FALSE)) + 
  ggplot_theme
```
<div style="text-align:center"><img src="/images/OPIM602_Project1/bike_rental_by_hour_of_the_day.png" /></div>

```r  
# feature engineering: hr

# convert hour to the distinct categories
bs_data_train_1 <- bs_data_train_1 %>%
  mutate(hr_category = case_when(
    hr %in% c(7, 8, 16, 17, 18, 19) ~ "commuting_hours",
    hr %in% c(9, 10, 11, 12, 13, 14, 15) ~ "working_hours",
    hr %in% c(0, 1, 2, 3, 4, 5, 6, 20, 21, 22, 23) ~ "other_hours"
  ))


# count by hour
bs_data_train_1 %>%
  mutate(hr_category = case_when(
    hr_category == "commuting_hours" ~ "Commuting Hours\n(7:00 AM to 8:59 AM and\n4:00 PM to 7:59 PM)",
    hr_category == "working_hours" ~ "Working Hours\n(9:00 AM to 3:59 PM)",
    hr_category == "other_hours" ~ "Other Hours\n(8:00 PM to 6:59 AM)",
  )) %>%
  ggplot(aes(x = hr_category,
             y = cnt)) + 
  geom_bar(stat = "identity",
           fill = "steelblue")+
  labs(x = "Hour Bin",
       y = "Count",
       title = "Bike Rental by Hour Bin",
       subtitle = "2011-2012",
       caption = "Source: Capital Bikeshare Data") +
  scale_y_continuous(labels = function(x) format(x, big.mark = ",", scientific = FALSE)) + 
  ggplot_theme + 
  coord_flip()
  
# ~~ data changes ~~

# remove the hour category
bs_data_train_1 <-  bs_data_train_1 %>%
  dplyr::select(-hr)

# make dummies
bs_data_train_1 <- dummy_cols(bs_data_train_1,
                              remove_first_dummy = FALSE,
                              remove_selected_columns =  TRUE)
```

<div style="text-align:center"><img src="/images/OPIM602_Project1/bike_rental_by_hour_bin.png" /></div>

#### *Final Model*

Based on my modeling procedures, the top three predictors of aggregate demand are hourly normalized temperature (“temp”), hourly normalized humidity (“hum”), and the binned other hour category. The regression equation can be seen below.

```r
# ~~ final model ~~

# backward stepwise
model_1_train <- train(cnt ~ .,
                       data = bs_data_train_1,
                       method = "leapBackward",
                       tuneGrid = data.frame(nvmax = 3))

# inspect the summary of the backward stepwise
summary(model_1_train$finalModel)

# three variable model based on the above
model_1 <- lm(formula = cnt ~ temp + hum + hr_category_other_hours,
           data = bs_data_train_1)
summary(model_1)
```

$$
\operatorname{cnt} = \alpha + \beta_{1}(\operatorname{temp}) + \beta_{2}(\operatorname{hum}) + \beta_{3}(\operatorname{hr\_category\_other\_hours}) + \epsilon
$$

The model has an adjusted R2 of 0.541, which means that 54.1% of the variation in aggregate demand is driven by the three variables in my final model. Additionally, the RMSE of the model is 1.381, which is very low relative to the bike rental amounts each day. Given the cube root transformation of the dependent variable, the inverse should be calculated (i.e., raised to the third power) to reverse the effects of the transformation when interpreting predictions. Refer to Appendix #3 for the summary results of the final model and Appendix #5 for the model metrics.

#### *Linear Regression Assumptions – Final Model*

As part of the auditing process of creating my multiple linear regression model, I tested my model for the four regression assumptions, which are detailed below.

<div style="text-align:center"><img src="/images/OPIM602_Project1/final_model_assumptions.png" /></div>

```r
# ~~ test assumptions ~~
# 1. Linearity: Refer to the scatterplot matrix

# 2. Normality: Residuals are NOT normal
ad.test(model_1$residuals)

# 3. Heteroscedasticity: HETEROSCEDASTIC, error variance is NOT constant
bptest(model_1)

# 4. Multicollinearity: Vales are less than 10, there is NOT multicollinearity
vif(model_1)

# ~~ plot test assumptions ~~

# explore influential observations
cooks_distance_cutoff_1 <- 4 / nrow(bs_data_train_1)

# plot residuals, QQ, and leverage
par(mfrow = c(2, 2))
lapply(c(1, 2, 4, 5),
       function (x) plot(model_1,
                         which = x,
                         cook.levels = c(cooks_distance_cutoff_1))) %>%
  invisible()
```

1.	Linearity: In Appendix #4 (top-right), the standard residuals closely follow the Q-Q plot and this show that there is a linear relationship.
2.	Normality: In Appendix #4 (top-left), the residuals and fitted values appear to be randomly scattered around the zero reference line in red initially suggesting normality. However, the results of the Anderson-Darling test for normality of the residuals returns a p-value of near-zero, which indicates the normality assumption is not satisfied.
3.	Heteroscedasticity: In Appendix #4 (top-left), there does not appear to be a bell-shaped effect of the residuals initially suggesting homoscedasticity and constant error variance. However, the results of the Breusch-Pagan test for constant error variance returns a p-value of near-zero, which indicates that the error terms have non-constant variance and are heteroscedastic.
4.	Independent Errors / Multicollinearity: In Appendix #6, I calculated the variance inflation factor (VIF) of my final model to inspect of independent predictor multicollinearity. Given the VIF values are low, near the 1.0 to 1.2 range, the error terms appear to be independent and there is no multicollinearity.

#### *Predictions on Test Data*

```r
# test data cleanup
bs_data_test_1 <- bs_data_test %>%
  
  # factor and select relevant columns
  mutate(season = factor(season),
         mnth = factor(mnth),
         hr = factor(hr),
         holiday = factor(holiday),
         weekday = factor(weekday),
         workingday = factor(workingday),
         weathersit = factor(weathersit)) %>%
  dplyr::select(season, mnth, hr, holiday, weekday, workingday, weathersit, temp, atemp, hum, windspeed, cnt) %>%
  
  # remove working day and atemp
  dplyr::select(-c(workingday, atemp)) %>%
  
  # cube root cnt
  mutate(cnt = nthroot(cnt, 3)) %>%
  
  # bin the hr
  mutate(hr_category = case_when(
    hr %in% c(7, 8, 16, 17, 18, 19) ~ "commuting_hours",
    hr %in% c(9, 10, 11, 12, 13, 14, 15) ~ "working_hours",
    hr %in% c(0, 1, 2, 3, 4, 5, 6, 20, 21, 22, 23) ~ "other_hours"
  )) %>%
  
  # remove hr
  dplyr::select(-c(hr))

# make dummies
bs_data_test_1 <- dummy_cols(bs_data_test_1,
                             remove_first_dummy = FALSE,
                             remove_selected_columns =  TRUE)

# prediction df
bs_data_test_1 <- bs_data_test_1 %>%
  dplyr::select(temp, hum, hr_category_other_hours, cnt)

# make predictions and report results
metrics <- function(y_pred, y_true){
  mse <- MSE(y_pred, y_true)
  rmse <- RMSE(y_pred, y_true)
  mae <- MAE(y_pred, y_true)
  print(paste0("MSE: ", round(mse, 6)))
  print(paste0("RMSE: ", round(rmse, 6)))
  print(paste0("MAE: ", round(mae, 6)))
  corPredAct <- cor(y_pred, y_true)
  print(paste0("Correlation: ", round(corPredAct, 6)))
  print(paste0("R^2 between y_pred & y_true: ", round(corPredAct^2, 6)))
  }


# performance on training data
cat("Performance using train dataset:\n")
metrics(y_pred = model_1$fitted.values, y_true = bs_data_train_1$cnt)

# performance on testing data
y_test_preds <- predict(model_1, newdata = bs_data_test_1)
cat("\nPerformances using test dataset:\n")
metrics(y_pred = y_test_preds, y_true = bs_data_test_1$cnt)
```

The performance of the final model on the training set and testing set is displayed in the table below.

|             | Training Set | Testing Set |
|-------------|--------------|-------------|
| MSE         | 1.897385     | 1.906189    |
| RMSE        | 1.377456     | 1.380648    |
| MAE         | 1.142663     | 1.143901    |
| Correlation | 0.734301     | 0.735362    |
| Adj. R^2    | 0.539197     | 0.540758    |


### # Recommendation

---

Based on my analysis, the top three predictors of bike rental demand are “other” hours of the day, temperature, and humidity. While my model is statistically significant overall, and each of the three predictors are statistically significant, the normality and homoscedasticity linear regression assumptions were not satisfied. As such, I recommend Capital Bikeshare management explore the usage of other machine learning models that can better predict the data as a multiple linear regression model may not be appropriate. However, while additional modeling is performed, Capital Bikeshare can further explore the relationship between temperature, humidity, and time of the day as they are statistically significant, and intuitively, practically significant. Specifically, management should target marketing campaigns on favorable (i.e., hotter) days, days with low humidity levels, and hours of the day not within off-peak times (i.e., 10:00 PM to 6:59 AM). This, in turn, will help increase ridership and thus drive top-line performance.

---

