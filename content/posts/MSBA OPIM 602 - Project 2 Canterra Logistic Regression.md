---
title: "MSBA OPIM 602 Project 2 - Canterra Logistic Regression"
linktitle: "MSBA OPIM 602 Project 2 - Canterra Logistic Regression"
author: "Jed Raynes"
date: 2021-11-22
type:
- post 
- posts
weight: 10
# series:
# - Hugo 101
aliases:
- /posts/OPIM-602-Project-2-Canterra-Logistic-Regression
---

# Canterra Logistic Regression

---

**Created:** November 22, 2021

This webpage is for Project 2 of the OPIM 602 course, Machine Learning I, in the [MSBA program at the Georgetown University McDonough School of Business](https://msb.georgetown.edu/msba/). 

### # Introduction

---

Canterra is a large organization with around 4,000 to 5,000 employees and experiences attrition of approximately 15% per annum. High level of employee turnover leads to delayed projects which have impacts on customer deliverables. Additionally, to replace former employees, the company must invest time and capital to train newer employees to cover the responsibilities of former employees. The company has engaged me to analyze an employee dataset which includes various job characteristics, employee demographics, and whether the employee left the organization within the past year. My findings indicate that job satisfaction and the employee’s total years in the workforce are the leading indicators of attrition or lack thereof.

We'll begin by loading in the data.

```r
# load data
employee_data <- read_excel("Employee_Data_Project.xlsx")

# convert dependent variable to binary instead of yes/no
employee_data <- employee_data %>%
  mutate(Attrition = case_when(
    Attrition == "Yes" ~ 1,
    Attrition == "No" ~ 0))

# reorder data
col_order <- c("Age", "BusinessTravel", "DistanceFromHome", "Education", "EmployeeID", "Gender", "JobLevel", "MaritalStatus", "Income", "NumCompaniesWorked", "StandardHours", "TotalWorkingYears", "TrainingTimesLastYear", "YearsAtCompany", "YearsWithCurrManager", "EnvironmentSatisfaction", "JobSatisfaction", "Attrition")
employee_data <- employee_data[, col_order]
```

### # Scope, Approach, and Assumptions

---

The dataset provided consists of 4,410 employees, 17 predictor variables, and one dependent variable. I was engaged to analyze the employee data and provide insights based on the analysis. Given the binomial outcome variable, I used a logistic regression model to predict attrition. A threshold of 0.05, or 5%, was used to determine statistical significance. Management hypothesized that job satisfaction, years at Canterra, and total working years influenced attrition. It was also asked that I consider employee demographics such as gender, the highest level of education achieved, and age. I focused my analysis on these six predictors given they were the focus of management. When predicting attrition, employees with predicted probabilities of 50% or greater are categorized as attrition. 

### # Data Splitting

---
I split the dataset into a training and testing set with a 70% / 30% split. Given the data provided is for only the past year and to maximize the data available for model training, I did not use a validation set in my analysis. Across the entire dataset, and in line with management’s estimate, 16.12% of employees left the company in the past year. Given the imbalance skewed towards employees that stay with the company, a stratified random sampling method was used to maintain a similar weighting of attrition in the testing and training sets.

```r
# train test split (stratified random sampling)

# set the index using the caret splitting
index <- createDataPartition(y = employee_data$Attrition,
                             p = 0.70,
                             list = FALSE)

# training set
employee_data_train <- employee_data[index, ]

# testing set
employee_data_test <- employee_data[-index, ]
```

We'll do an initial edit of the training set.

```r
# edit data types
## EDIT TEST SET
employee_data_train <- employee_data_train %>%
  mutate(TotalWorkingYears = as.numeric(TotalWorkingYears),
         NumCompaniesWorked = as.numeric(NumCompaniesWorked),
         Education = as.numeric(Education),
         EnvironmentSatisfaction = as.numeric(EnvironmentSatisfaction),
         JobSatisfaction = as.numeric(JobSatisfaction)) %>%
  mutate(Attrition = as.factor(Attrition),
         BusinessTravel = as.factor(BusinessTravel),
         Gender = as.factor(Gender),
         JobLevel = as.factor(JobLevel),
         MaritalStatus = as.factor(MaritalStatus))

# inspect data types
summary(employee_data_train)
```

### # Missing Data

---

Through the exploration of the training set, I noticed that the following predictors were missing data: number of companies worked (12 missing entries), total working years (6), environment satisfaction (18), and job satisfaction (11). The image below plots the distribution of the four predictors with missing data points.

```r
# exploration of missing data

# ~~ ggplot ~~

# total working years
total_working_years_distribution <- employee_data_train %>%
  ggplot() +
  aes(x = TotalWorkingYears) + 
  geom_histogram(fill = "cadetblue",
                 color = "black",
                 aes(y = ..density..)) +
  geom_density(color = "coral",
               size = 1) +
  labs(x = "Total Working Years",
       y = "Density") +
  ggplot_theme

num_companies_worked_distribution <- employee_data_train %>%
  ggplot() +
  aes(x = NumCompaniesWorked) + 
  geom_histogram(fill = "cadetblue",
                 color = "black",
                 aes(y = ..density..),
                 binwidth = 1) +
  geom_density(color = "coral",
               size = 1) +
  labs(x = "Number of Companies Worked",
       y = "Density") +
  ggplot_theme

# environment satisfaction
environment_satisfaction_distribution <- employee_data_train %>%
  ggplot() +
  aes(x = EnvironmentSatisfaction) + 
  geom_bar(fill = "cadetblue",
                 color = "black") + 
  labs(x = "Environment Satisfaction",
       y = "Count") +
  ggplot_theme

# job satisfaction
job_satisfaction_distribution <- employee_data_train %>%
  ggplot() +
  aes(x = JobSatisfaction) + 
  geom_bar(fill = "cadetblue",
                 color = "black") + 
  labs(x = "Job Satisfaction",
       y = "Count") +
  ggplot_theme



# ~~ cowplot ~~

# title
distribution_title <- ggdraw() + 
  draw_label("Distributions of Predictors with Missing Data",
             fontface = "bold",
             x = 0,
             hjust = -0.3) + 
  theme(plot.margin = margin (0, 0, 0, 7))

# caption
distribution_source <- ggdraw() + 
  draw_label("Source: Canterra",
             x = 0,
             hjust = 0,
             size = 10) + 
  theme(plot.margin = margin (0, 0, 0, 7))

# distribution plots
dist_plots <- plot_grid(total_working_years_distribution,
                        num_companies_worked_distribution,
                        environment_satisfaction_distribution,
                        job_satisfaction_distribution,
                        ncol = 2)

# final cowplot
plot_grid(distribution_title,
          dist_plots,
          distribution_source,
          ncol = 1,
          rel_heights = c(0.1, 1))
```

<div style="text-align:center"><img src="/images/OPIM602_Project2/01_missing_data.png" /></div>

The total working years and number of companies worked variables appear to be right-skewed and represent numerical data types. Given the skewness, I opted to impute the median value of each predictor for missing data points. The environment and job satisfaction are technically categorical as it is a score between one and four. For the purposes of the analysis, these variables were treated as numerical. For the latter two predictors, I used the mean of each predictor as to not skew the satisfactory ratings of the environment / job given there is no true midpoint of a four-level factor.

The edits were made to the training set in the code below.

```r
# handle missing data

# total working years: impute median since it's positively skewed
# number of companies worked: impute the median since it's positively skewed
# environment satisfaction: put it at the mean
# job satisfaction: put it at the mean
## EDIT TEST SET
employee_data_train <- employee_data_train %>%
  # mutate total working years
  mutate_at(vars(TotalWorkingYears),
            ~ifelse(is.na(.), median(., na.rm = TRUE), .)) %>%
  # mutate number of companies worked at
  mutate_at(vars(NumCompaniesWorked),
            ~ifelse(is.na(.), median(., na.rm = TRUE), .)) %>%
  # mutate job satisfaction
  mutate_at(vars(JobSatisfaction),
            ~ifelse(is.na(.), mean(., na.rm = TRUE), .)) %>%
  # muatte environment satisfaction
  mutate_at(vars(EnvironmentSatisfaction),
            ~ifelse(is.na(.), mean(., na.rm = TRUE), .)) %>%
  # convert total working years and number of companies worked to numeric
  mutate(TotalWorkingYears = as.numeric(TotalWorkingYears),
         NumCompaniesWorked = as.numeric(NumCompaniesWorked)) %>%
  # ensure categories are factors
  mutate(Attrition = as.factor(Attrition),
         BusinessTravel = as.factor(BusinessTravel),
         Gender = as.factor(Gender),
         JobLevel = as.factor(JobLevel),
         MaritalStatus = as.factor(MaritalStatus)) %>%
  
  dplyr::select(-c(EmployeeID, StandardHours))

# inspect to ensure the data was changed and there are no remaining missing values
summary(employee_data_train)
```


### # Exploratory Data Analysis

---

After the imputation of missing data, a robust exploration of data was performed to assess relationships between predictors and employee attrition. First, I explored management’s assumed predictors: job satisfaction, years at the company, and total working years. Job satisfaction responses were primarily three or four, however, while attrition occurred across all satisfaction levels, there tends to be more attrition at the one and three level. Most of the employees have been with the company between one and 10 years and attrition tends to occur with employees with less than five years of tenure. Employees tend to have between five and 10 years of total years of work experience with attrition typically occurring with those between one to two years, five to six years, and ten years of experience. The image below displays the density of each predictor separated by attrition and no attrition.

```r
# exploration of management's assumptions

# ~~  plots ~~

# job satisfaction
job_satisfaction_attrition <- employee_data_train %>%
  ggplot() +
  aes(x = JobSatisfaction ,
      y = Attrition) + 
  geom_jitter(color = "cadetblue",
              alpha = 1/3) + 
  labs(x = "Job Satisfaction",
       y = "Attrition") +
  scale_y_discrete(labels = c("No", "Yes")) + 
  ggplot_theme

# years at the company
years_at_company_attrition <- employee_data_train %>%
  ggplot() +
  aes(x = YearsAtCompany ,
      y = Attrition) + 
  geom_jitter(color = "cadetblue",
              alpha = 1/3) + 
  labs(x = "Years at Canterra",
       y = "Attrition") +
  scale_y_discrete(labels = c("No", "Yes")) + 
  ggplot_theme

# total working years
total_working_years_attrition <- employee_data_train %>%
  ggplot() +
  aes(x = TotalWorkingYears ,
      y = Attrition) + 
  geom_jitter(color = "cadetblue",
              alpha = 1/3) + 
  labs(x = "Total Working Years",
       y = "Attrition") +
  scale_y_discrete(labels = c("No", "Yes")) + 
  ggplot_theme



# ~~ cowplot ~~

# title
attrition_title <- ggdraw() + 
  draw_label("Attrition by Management's Assumed Predictors",
             fontface = "bold",
             x = 0,
             hjust = -0.3) + 
  theme(plot.margin = margin (0, 0, 0, 7))

# caption
attrition_source <- ggdraw() + 
  draw_label("Source: Canterra",
             x = 0,
             hjust = 0,
             size = 10) + 
  theme(plot.margin = margin (0, 0, 0, 7))

# distribution plots
attr_plots_0_p1 <- plot_grid(job_satisfaction_attrition,
                        years_at_company_attrition,
                        ncol = 2)

# layer in total working years
attr_plots_0_p2 <- plot_grid(attr_plots_0_p1,
                             total_working_years_attrition,
                             ncol = 1)


# final cowplot
plot_grid(attrition_title,
          attr_plots_0_p2,
          attrition_source,
          ncol = 1,
          rel_heights = c(0.1, 1))
```

<div style="text-align:center"><img src="/images/OPIM602_Project2/02_eda_management.png" /></div>

In addition to management’s hypothesized predictors, I explored the employee demographics of gender, highest level of education achieved, and age. The company’s employee base tends to be slightly weighted towards more males, employees primarily have a bachelor’s and master’s degrees with few having doctorates, and employees tend to be approximately 25 to 45 years old. The image above displays the employee demographics by attrition.

```r
# exploration of employee demographics

# ~~  plots ~~

# job satisfaction
gender_attrition <- employee_data_train %>%
  ggplot() +
  aes(x = Gender ,
      y = Attrition) + 
  geom_jitter(color = "cadetblue",
              alpha = 1/3) + 
  labs(x = "Gender",
       y = "Attrition") +
  scale_y_discrete(labels = c("No", "Yes")) + 
  ggplot_theme

# years at the company
education_attrition <- employee_data_train %>%
  ggplot() +
  aes(x = Education ,
      y = Attrition) + 
  geom_jitter(color = "cadetblue",
              alpha = 1/3) + 
  labs(x = "Education",
       y = "Attrition") +
  scale_y_discrete(labels = c("No", "Yes")) + 
  ggplot_theme

# total working years
age_attrition <- employee_data_train %>%
  ggplot() +
  aes(x = Age ,
      y = Attrition) + 
  geom_jitter(color = "cadetblue",
              alpha = 1/3) + 
  labs(x = "Age",
       y = "Attrition") +
  scale_y_discrete(labels = c("No", "Yes")) + 
  ggplot_theme



# ~~ cowplot ~~

# title
attrition_title_1 <- ggdraw() + 
  draw_label("Attrition by Employee Demographics",
             fontface = "bold",
             x = 0,
             hjust = -0.5) + 
  theme(plot.margin = margin (0, 0, 0, 7))

# caption
attrition_source <- ggdraw() + 
  draw_label("Source: Canterra",
             x = 0,
             hjust = 0,
             size = 10) + 
  theme(plot.margin = margin (0, 0, 0, 7))

# distribution plots
attr_plots_1_p1 <- plot_grid(gender_attrition,
                             education_attrition,
                             ncol = 2)

# layer in age
attr_plots_1_p2 <- plot_grid(attr_plots_1_p1,
                             age_attrition,
                             ncol = 1)

# final cowplot
plot_grid(attrition_title_1,
          attr_plots_1_p2,
          attrition_source,
          ncol = 1,
          rel_heights = c(0.1, 1))
```

<div style="text-align:center"><img src="/images/OPIM602_Project2/03_eda_employee.png" /></div>

### # Data Sampling

---

As mentioned previously, I used stratified random sampling when splitting my data given the imbalance in the dependent variable. To better predict the probability of attrition, I explored sampling methods to offset the imbalanced classes. Employee attrition, which is the minority class, consisted of 16.2% of the training data before sampling. Refer to Appendix #1 for each sampling methodology and related impacts on the training set.

To compare the sampling methods, the Akaike information criterion (“AIC”) scores, which estimates prediction error, of a full logistic model (i.e., inclusive of all 17 predictors) was compared. The sampling method with the lowest AIC, the under-sampling method, is used as the basis for training the logistic model.

```r
# compare splits
employee_data %>%
  group_by(Attrition) %>%
  summarize(count = n()) %>%
  mutate(proportion = count / sum(count, na.rm = TRUE))
  
employee_data_train %>%
  group_by(Attrition) %>%
  summarize(count = n()) %>%
  mutate(proportion = count / sum(count, na.rm = TRUE))
  
# random over sample the attrition (1) and undersample the lack of attrition (0)
employee_data_train_overunder <- ovun.sample(formula = Attrition ~ .,
                                            data = employee_data_train,
                                            method = "both",
                                            N = nrow(employee_data_train),
                                            p = 0.5,
                                            seed = 1337)$data

# random over sample the attrition (1)
employee_data_train_over <- ovun.sample(formula = Attrition ~ .,
                                        data = employee_data_train,
                                        method = "over",
                                        p = 0.5,
                                        seed = 1337)$data

# random under sample the lack of attrition (0)
employee_data_train_under <- ovun.sample(formula = Attrition ~ .,
                                        data = employee_data_train,
                                        method = "under",
                                        p = 0.5,
                                        seed = 1337)$data

# ROSE sample of synthetic data
employee_data_train_rose <- ROSE(formula = Attrition ~.,
                                 data = employee_data_train,
                                 p = 0.5,
                                 seed = 1337)$data


# explore distribution of the dependent variable
employee_data_train %>%
  group_by(Attrition) %>%
  summarize(count = n()) %>%
  mutate(proportion = count / sum(count, na.rm = TRUE))
  
employee_data_train_overunder %>%
  group_by(Attrition) %>%
  summarize(count = n()) %>%
  mutate(proportion = count / sum(count, na.rm = TRUE))
  
employee_data_train_over %>%
  group_by(Attrition) %>%
  summarize(count = n()) %>%
  mutate(proportion = count / sum(count, na.rm = TRUE))
  
employee_data_train_under %>%
  group_by(Attrition) %>%
  summarize(count = n()) %>%
  mutate(proportion = count / sum(count, na.rm = TRUE))
  
employee_data_train_rose %>%
  group_by(Attrition) %>%
  summarize(count = n()) %>%
  mutate(proportion = count / sum(count, na.rm = TRUE))

# choosing the best method for weighting samples
# under-weighting the lack of attrition is the lowest AIC and thus will be used
model_all <- glm(Attrition ~ ., family = "binomial", employee_data_train)
model_all_overunder <- glm(Attrition ~ ., family = "binomial", employee_data_train_overunder)
model_all_over <- glm(Attrition ~ ., family = "binomial", employee_data_train_over)
model_all_under <- glm(Attrition ~ ., family = "binomial", employee_data_train_under)
model_all_rose <- glm(Attrition ~ ., family = "binomial", employee_data_train_rose)

# stargazer to compare up/down sampling methods
stargazer(model_all,
          model_all_overunder,
          model_all_over,
          model_all_under,
          model_all_rose,
          type = "text",
          title = "Regression Results",
          align = TRUE,
          single.row = TRUE)
```

The table below is the AIC by split.

|         | No Sampling | Over and Under | Over    | Under   | ROSE    |
| ------- | ----------- | -------------- | ------- | ------- | ------- |
| **AIC** | 2,374.8     | 3,680.6        | 6,072.5 | 1,180.6 | 3,774.5 |

### # Modeling

---

Four logistic regression models were fit to the under-sampled training data to predict attrition.

*Model 1: Management’s Hypothesis*

```r
# model 0: management's hypothesis
model_0 <- glm(Attrition ~ JobSatisfaction + TotalWorkingYears + YearsAtCompany, family = "binomial", employee_data_train_under)

# make predictions
model_0_train_preds <-predict(model_0, newdata = employee_data_train, type = "response")
employee_data_train_model_0 <- employee_data_train
employee_data_train_model_0$preds <- ifelse(model_0_train_preds >= 0.5, 1, 0)

# confusion matrix
confusionMatrix(data = as.factor(employee_data_train_model_0$preds),
                reference = as.factor(employee_data_train_model_0$Attrition),
                positive = "1")
                
# ROC curve
roc.curve(employee_data_train_model_0$Attrition,
          employee_data_train_model_0$preds)
          
# VIF
vif(model_0)

# summary
summary(model_0)
```

In the first model, job satisfaction and total working years are statistically significant predictors which indicates that they impact likelihood of attrition. Given the negative coefficients for both, as job satisfaction increases by one or total years of working experience increases by one, holding all-else equal, the log-odds of attrition decreases by 0.22 and 0.06, respectively. 

*Model 2: Employee Demographics*

```r
# model 1: employee demographics
model_1 <- glm(Attrition ~ Gender + Education + Age, family = "binomial", employee_data_train_under)

# make predictions
model_1_train_preds <-predict(model_1, newdata = employee_data_train, type = "response")
employee_data_train_model_1 <- employee_data_train
employee_data_train_model_1$preds <- ifelse(model_1_train_preds >= 0.5, 1, 0)

# confusion matrix
confusionMatrix(data = as.factor(employee_data_train_model_1$preds),
                reference = as.factor(employee_data_train_model_1$Attrition),
                positive = "1")
                
# ROC curve
roc.curve(employee_data_train_model_1$Attrition,
          employee_data_train_model_1$preds)
          
# VIF
vif(model_1)

# summary
summary(model_1)
```

In the second model, age is the only statistically significant predictor with a negative coefficient. This indicates that as an employee’s age increases by one year, holding all-else equal, the log-odds of attrition decreases by 0.04.

*Model 3: Management’s Hypothesis and Employee Demographics*

```r
# model 2: management's hypothesis + employee demographics
model_2 <- glm(Attrition ~ JobSatisfaction + TotalWorkingYears + YearsAtCompany + Gender + Education + Age, family = "binomial", employee_data_train_under)

# make predictions
model_2_train_preds <-predict(model_2, newdata = employee_data_train, type = "response")
employee_data_train_model_2 <- employee_data_train
employee_data_train_model_2$preds <- ifelse(model_2_train_preds >= 0.5, 1, 0)

# confusion matrix
confusionMatrix(data = as.factor(employee_data_train_model_2$preds),
                reference = as.factor(employee_data_train_model_2$Attrition),
                positive = "1")
                
# ROC curve
roc.curve(employee_data_train_model_2$Attrition,
          employee_data_train_model_2$preds)
          
# VIF
vif(model_2)

# summary
summary(model_2)
```

In the third model, job satisfaction and total working years are statistically significant predictors, both with negative coefficients. Like Model 1, as job satisfaction increases by one or total working years increases by one, holding all else equal, the log-odds of attrition decreases by 0.22 and 0.05, respectively. Despite being statistically significant in Model 2, when adding additional predictors, age, along with the other employee demographic predictors, are not statistically significant. This indicates that while there may be a relationship between the employee demographics and attrition, it is not statistically significant and should not be used when assessing the likelihood of an employee leaving the company. Refer to Appendix #2 for a summary of the first three models.

The accuracy, balanced accuracy, area under the curve (“AUC”), AIC, and residual deviance of each model was assessed to determine the starting point for my final model. Accuracy is defined as the total correct predictions (of both attrition and no attrition) over the total predictions. Balanced accuracy is the calculated as the average of the sensitivity (the true positive rate) and specificity (the true negative rate) and was used given the imbalance in the original training and testing data. Area under the curve measures ranking of predictions and actuals. Finally, AIC and residual deviance are measures of model fit. Refer to Appendix #3 for the models’ metrics.

While Model 1 displayed the best performance in four of the five metrics, given the same two predictors were statistically significant and the differences in metrics are generally immaterial (within 1-2%), I used Model 3 as the initial model when determining the top two predictors.

*Final Model: Top Two Predictors*

```r
# subset for the relevant columns
employee_data_train_under <- employee_data_train_under %>%
  dplyr::select(JobSatisfaction, TotalWorkingYears, YearsAtCompany, Gender, Education, Age, Attrition)


# bestglm backward
bestglm_output <- bestglm(employee_data_train_under,
                          family = binomial,
                          IC = "AIC",
                          method = "exhaustive",
                          nvmax = 2)
                          
# view the best model by AIC
bestglm_output$BestModels

# final model
final_model <- glm(Attrition ~ JobSatisfaction + TotalWorkingYears, family = "binomial", data = employee_data_train_under)

# ~~ train ~~

# make predictions: train
final_model_train_preds <-predict(final_model, newdata = employee_data_train, type = "response")
employee_data_train_final_model <- employee_data_train
employee_data_train_final_model$preds <- ifelse(final_model_train_preds >= 0.5, 1, 0)

# confusion matrix
confusionMatrix(data = as.factor(employee_data_train_final_model$preds),
                reference = as.factor(employee_data_train_final_model$Attrition),
                positive = "1")
                
# ROC curve
roc.curve(employee_data_train_final_model$Attrition,
          employee_data_train_final_model$preds)
          
# VIF
vif(final_model)

# summary
summary(final_model)
````

Using Model 3 as the starting point, I performed an “exhaustive” stepwise regression to determine the model of best fit. AIC was set as the metric to minimize when analyzing potential models and the under-sampled training set was used for model training. The model with the lowest AIC is predicted by job satisfaction and total working years, the same two statistically significant predictors in both Model 1 and Model 3. Once again, given the negative coefficients, as job satisfaction increases by one or total working years increases by one, holding all-else equal, the log-odds of attrition decreases by 0.23 and 0.07, respectively. The final model equation is as follows:

The final model was used to predict on the training set (without sampling changes) and the testing set. The table below highlights the metrics from the training and testing sets.

The results of the testing set closely mirror that of the training set. Our final model predicted employee attrition and lack of attrition 57.45% of the time. Refer to Appendix #4 for a summary of the final model.

```r
# test data mutations to match
employee_data_test_final_model <- employee_data_test %>%
  mutate(TotalWorkingYears = as.numeric(TotalWorkingYears),
         NumCompaniesWorked = as.numeric(NumCompaniesWorked),
         Education = as.numeric(Education),
         EnvironmentSatisfaction = as.numeric(EnvironmentSatisfaction),
         JobSatisfaction = as.numeric(JobSatisfaction)) %>%
  mutate(Attrition = as.factor(Attrition),
         BusinessTravel = as.factor(BusinessTravel),
         Gender = as.factor(Gender),
         JobLevel = as.factor(JobLevel),
         MaritalStatus = as.factor(MaritalStatus)) %>%
  # mutate total working years
  mutate_at(vars(TotalWorkingYears),
            ~ifelse(is.na(.), median(., na.rm = TRUE), .)) %>%
  # mutate number of companies worked at
  mutate_at(vars(NumCompaniesWorked),
            ~ifelse(is.na(.), median(., na.rm = TRUE), .)) %>%
  # mutate job satisfaction
  mutate_at(vars(JobSatisfaction),
            ~ifelse(is.na(.), mean(., na.rm = TRUE), .)) %>%
  # muatte environment satisfaction
  mutate_at(vars(EnvironmentSatisfaction),
            ~ifelse(is.na(.), mean(., na.rm = TRUE), .)) %>%
  # convert total working years and number of companies worked to numeric
  mutate(TotalWorkingYears = as.numeric(TotalWorkingYears),
         NumCompaniesWorked = as.numeric(NumCompaniesWorked)) %>%
  # ensure categories are factors
  mutate(Attrition = as.factor(Attrition),
         BusinessTravel = as.factor(BusinessTravel),
         Gender = as.factor(Gender),
         JobLevel = as.factor(JobLevel),
         MaritalStatus = as.factor(MaritalStatus)) %>%
  dplyr::select(JobSatisfaction, TotalWorkingYears, Attrition)
  
# ~~ test ~~

# make predictions: test
final_model_test_preds <-predict(final_model, newdata = employee_data_test_final_model, type = "response")
employee_data_test_final_model$preds <- ifelse(final_model_test_preds >= 0.5, 1, 0)

# confusion matrix
confusionMatrix(data = as.factor(employee_data_test_final_model$preds),
                reference = as.factor(employee_data_test_final_model$Attrition),
                positive = "1")
                
# ROC curve
roc.curve(employee_data_test_final_model$Attrition,
          employee_data_test_final_model$preds)
```

|                            | Training      | Testing       |
|----------------------------|---------------|---------------|
| **Accuracy**               |     0.5799    |     0.5745    |
| **Balanced   Accuracy**    |     0.6330    |     0.6359    |
| **AUC**                    |     0.6330    |     0.6360    |

### # Recommendations

---

Given the analysis performed, it is evident that management’s initial hypothesis was largely proven true; the top two predictors of attrition or lack of attrition are the employee’s job satisfaction score and his/her total years of work experience. The employee demographics of gender, education, and age are not statistically significant predictors in determining attrition. As a result, I recommend a two-pronged approach to reduce the high levels of employee turnover:

1. **Boost employee morale:** Job satisfaction is a subjective metric as one person’s definition of satisfaction may differ from another’s definition. However, enterprise-wide initiatives should be implemented to improve employee morale. Employees already typically score job satisfaction as a three or four, indicating generally positive levels of satisfaction. However, additional initiatives such as spot bonuses for work ethic, real-time recognition to show employee appreciation, and employee events to boost camaraderie will improve employee morale and boost job satisfaction in employees. Given the subjectivity, the company should have employees’ managers hold one-on-one meetings to gather feedback on what the employee wants and how they define job satisfaction so initiatives can be focused for each person.
2. **Hire experienced employees:** The current employee base has mostly worked less than 10 years and has worked at one additional company. Rather than focusing hiring initiatives on entry-level employees, the company should focus efforts on experienced hires. Using external headhunters may prove useful in finding candidates with more years of experience. In turn, this should reduce levels of future turnover.

While two potential solutions are proposed above, I recommend focusing initial efforts on retaining staff to prevent current burnout and turnover before focusing on external hires. Keeping talent within the organization and showing lower levels of turnover makes the company more attractive to experienced candidates looking for a job / company to settle with long-term. Aside from these two predictors, the company needs to ensure that controllable metrics such as total compensation remain competitive with other companies. Candidates then can overlook the compensation aspect and focus on the culture within the company (i.e., employees’ job satisfaction).

### # Appendix

---

For referenced appendices, refer to the final project submission in my GitHub.

---

