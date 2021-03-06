# Statistical Models in R 

#### Load the libraries
library("ggplot2")
library("predict3d")
library("psych")
library("dplyr")
library("gtsummary")## Allows for publication ready tables
library("DescTools") ## Needed for normality testing
library("nortest") ## Needed for normality testing
library("lmtest") ## Need for heteroskedasticity testing
library(corrr)

### Introduction

- Linear regression models (also known as "Ordinary Least Squares" model) allow us to determine if changing the values on a variable is associated with the values of another variable.

- In other words, if I make a 1-unit change in $X$, how much does Y change? 
 
- We use linear regression models to test the association between two or more variables where the outcome is a continuous data type. 

- linear regression model is called the "Ordinary Least Squares" or OLS model because it minimizes the squared errors (e.g., distance from the best-fit line). 

1. Assumptions about the form of the model: 
   - The linearity assumption. examining the scatter plot of Y versus X. 
2. Assumptions about the errors: $e \sim Niid(0,\sigma^2)$.
   - normality assumption.
   - The errors have mean zero.
   - the constant variance assumption. 
   - The independent-errors assumption. the autocorrelation problem. 
3. Assumptions about the predictors: 
   - nonrandom (assumed fixed). 
   - measured without error. 
   - linearly independent of each other; collinearity problem. 
4.	Assumptions about the observations: 
     - equally reliable and have approximately equal role in determining the regression results and in influencing conclusions. 

- diagnostic methods to check the violation of regression assumption are based on the study of model residuals with the help of various types of graphics. 

### Simple linear regression

- The structural form of a linear regression model:
$$\large Y_i = \beta_0 + \beta_1 X_{1i} + \epsilon$$

- Typical notations of the linear regression include:

* $Y_i$ denotes the outcome (or dependent) variable for subject $i$
* $X_{1i}$ denotes the predictor of interest $(X_1)$ for subject $i$
* $\beta_0$ denotes the Y-intercept when $X$ is zero
* $\beta_1$ denotes the slope or the change in $Y$ with a 1-unit change in $X$
* $\epsilon$ denotes the error or residuals

#### Diabets data

```{r,message=FALSE, warning=FALSE}
library(readr)
diabetes1 <- read_csv("diabetes.data.csv")
```

- The following variables are included in the data

* Pregnancies: Number of times pregnant
* Glucose: Plasma glucose concentration a 2 hours in an oral glucose tolerance test
* BloodPressure: Diastolic blood pressure (mm Hg)
* SkinThickness: Triceps skin fold thickness (mm)
* Insulin: 2-Hour serum insulin (mu U/ml)
* BMI: Body mass index (weight in kg/(height in m)^2)
* DiabetesPedigreeFunction: Diabetes pedigree function
* Age: Age (years)
* Outcome: Class variable (0 = no diabetes or 1 = diabetes)

### Correlation
-	Correlation analysis is concerned with measuring whether two variables are associated with each other. 
   -	If two variables tend to change together in the same direction, they are said to be positively correlated. 
   -	If they tend to change together in opposite directions, they are said to be negatively correlated. 


```{r, message=FALSE, warning=FALSE}
 diabetes1 %>%
  select(- ...1,- Outcome,-Pregnancies,-DiabetesPedigreeFunction) %>%
  correlate() %>% shave(upper = TRUE) %>%
  fashion(decimals = 2, na_print = ".") 
```

### Visualize the association using scatter plot
- Let's look at how Age is related to Glucose level by plotting their relationship.  
- As Age increases, the Glucose level also increases. There appears to be a positive relationship between Age and Glucose level. 

```{r, warning = FALSE, message = FALSE,fig.align='center', fig.height=4}
ggplot(diabetes1, aes(x = Age, y = Glucose)) +
  geom_point() +  stat_smooth()
```
- Then, we update our linear regression model's structural form:

$$Glucose_{i} = \beta_0 + \beta_1 Age_{i} + \epsilon,$$

- where $Glucose_{i}$ denotes the expected Glucose level for subject $i$ given the Age of subject $i$. 
 
- We will use the `lm()` function with Glucose level as the $Y$ variable and Age as the $X$ variable. 
- By using the `lm()` function, we can construct the linear regression model: `lm(Glucose ~ Age, data = diabetes.data)`. 

Here is how we put all of this together in R:

```{r, warning = FALSE, message = FALSE}
linear.model1 <- lm(Glucose ~ Age, data = diabetes1)
summary(linear.model1)
```

- $\beta_{1}$ coefficient is in the linear regression output as `Age`, which is `0.71642`.

- The `lm()` function does not generate 95% CI, so you will need to use the `confint()` function. 

```{r, warning = FALSE, message = FALSE}
### Generate the 95% CI
confint(linear.model1)
```

### Interpret the linear regression output

- We are interested in the coefficients. To make interpreting the output easier, we can create a table to visualize the critical elements using `gtsummary` package.

```{r, warning = FALSE, message = FALSE}
model1 <- tbl_regression(linear.model1, intercept = TRUE)
as_gt(model1) %>% 
  gt::tab_header("Table 2. Linear regression model") %>%
  gt::tab_options(table.align='center')
```

- The `Intercept` denotes the $Y$ intercept when $X$ is equal to zero. In this case, it would be where Glucose level would be on the linear plot when Age is equal to zero, which is 97.08-units of Glucose.

- The `Age` coefficient denotes the change in Glucose level for a one-unit increase in Age. In other words, a 1-year increase in Age is associated with a 0.72-unit increase in Glucose level. 

- Since the 95% CI is between 0.53-units and 0.90-units of Glucose, it does not include zero, so this association is statistically significant. We can also look a the p-value of the `Age` coefficient to determine whether this is statistically significant (<0.0001).
   - However, it is preferable to present the 95% CI when describing the association between $X$ and $Y$. 

- The `Adjusted R squared` denotes the amount of data that are explained by the linear regression model. 

- In other words, the current linear regression model explains 6.8% of the data.

### Visualize predicted model

- We can plot the linear form of the model against the actual data using the `ggPredict()` function. 


```{r, warning = FALSE, message = FALSE, fig.dim=c(8,5), fig.align='center'}
ggPredict(linear.model1, digits = 1, show.point = TRUE,
          se = TRUE, xpos = 0.5)
```

- Here is a version without the scatter plot.
```{r, warning = FALSE, message = FALSE, fig.align='center', fig.dim=c(7,5)}
ggPredict(linear.model1, digits = 1, show.point = FALSE,
          se = TRUE, xpos = 0.5)
```


### Multiple regression: Adding a confounder

```{r, warning = FALSE, message = FALSE}
# Generate groups based on pregnancy history
diabetes1$pregnancy.history <- ifelse(diabetes1$Pregnancies == 0, 0,1)
table(diabetes1$pregnancy.history)
```

- We see that there are 657 women who a history of pregnancy and 111 women with no history of pregnancy. 

- We need to include a confounder `pregnancy.history` in our linear regression model:

$$Glucose_{i} = \beta_0 + \beta_1 Age_{i} + \beta_2 Pregnancy History_{i} + \epsilon,$$

where $Glucose_{i}$ denotes the expected Glucose level for subject $i$ given the Age of subject $i$ controlling for Pregnancy History of subject $i$. 

```{r, warning = FALSE, message = FALSE}
linear.model2 <- lm(Glucose ~ Age + pregnancy.history, data = diabetes1)
summary(linear.model2)
```

- We can present the model output into a table.

```{r, warning = FALSE, message = FALSE}
#### Present the output in a table
model2 <- tbl_regression(linear.model2, intercept = TRUE)
as_gt(model2) %>% 
  gt::tab_header("Table 3. Multiple regression model") %>% 
  gt::tab_options(table.align='center')
```

- You can see that the `Age` coefficient is slightly different from our first model. It is 0.76 with a 95% CI of 0.57, 0.95. Compare this to the previous model's result, which was 0.72; 95% CI: 0.53, 0.90. 

```{r, warning = FALSE, message = FALSE}
#### Merge the two linear regression model's outputs
model1 <- tbl_regression(linear.model1, intercept = TRUE)
model2 <- tbl_regression(linear.model2, intercept = TRUE)
table1 <- tbl_merge(tbls = list(model1, model2),
          tab_spanner = c("**Model 1**", "**Model 2**"))
as_gt(table1) %>% 
  gt::tab_header("Table 4. Comparison between regression models")%>%
  gt::tab_options(table.align='center')
```

- Model 1 is considered the crude model or the unadjusted model. 
- Model 2 is the adjusted model because it is adjusting based on the Pregnancy History confounder. 

- Notice that the $\beta_{1, unadjusted}$ is 0.72 which is lower than the $\beta_{1, adjusted}$ result which is 0.76. 

- Additionally, the `Adjusted R squared` is higher in model 2 (7.35%) compared to model 1, which was 6.82%. 
- This means that Model 2 does a better job of explaining the data than Model 1. 
- Let's plot Model 2's results.

```{r, warning = FALSE, message = FALSE, fig.align='center', fig.height=4}
ggPredict(linear.model2, digits = 1, show.point = FALSE,
          se = TRUE, xpos = 0.5)
```
- You can derive the difference between the groups with a history of pregnancy and without a history of pregnancy by substracting 102 - 94.5, which is 7.5, the $\beta_{2}$ for `pregnancy.history`. 


- We can see that the group that had a history of pregnancy is lower than the group that did not have a history of pregnancy. 

- This makes sense when you look at the `pregnancy.history` coefficient. 

- It is -7.5, which means that a subject with a history of pregnancy is associated with a 7.5 decrease in Glucose level (95% CI: -13.80, 
-1.15) compared to a subject without a history of pregnancy controlling for age.

- Therefore, for all ranges of Age, the group with a history of pregnancy will have Glucose levels that are 7.5-unit lower than a group without a history of pregnancy. 

- You can visualize this on the plot; the linear lines do not cross and remain constant across all ranges of Age.

- But there is a positive correlation betwteen Age and Glucose level. 

### Evaluate residual plots

- It is good practice to look at the residuals of the regression model to make sure that the assumptions hold. 
- We need to check whether the residual are correlated with fitted or predicted values of Glucose. 
- If there is an association, then we have heteroscedasticity, which is a violation of the linear regression model assumption. 

```{r, warning = FALSE, message = FALSE, fig.dim=c(6,4), fig.align='center'}
plot(linear.model1$res ~ linear.model1$fitted)
```

- Upon visual inspection,there doesn't appear to be any evidence of heteroskedasticity. we can see that the residuals are uniform across the "fitted" values. 

- We can verify this visual inspection by performing the Breusch-Pagan test of heteroskedasticity. 
- We will need to install and load the `lmtest` package and use the `bptest()` function. 

```{r, warning = FALSE, message = TRUE}
bptest(linear.model1)
```

- The p-value is 0.1169, we fail to reject the null that the variance of the residuals are constant. 
- If, there was heteroskedasticity, we could address this by estimating robust standard errors. 


- We can also evaluate if the residuals are normally distributed. 
- We can generate a histogram and a Q-Q plot. 
- The historgram has a slight left skew and the Q-Q plot has its tails deviate from the neutral line. 

```{r, warning = FALSE, message = FALSE, fig.dim=c(6,4), fig.align='center'}
par(mfrow = c(1, 2))
hist(linear.model1$res); qqnorm(linear.model1$res); 
qqline(linear.model1$res, col = "2", lwd = 1, lty = 1)
```
- You only need to use one of these tests. They will generally give the same results. 

- We can also test for the normality of the residuals. 
- Common tests of normality include the Shapiro-Wilk's test, and the Kolmogorov-Smirnov test. 

```{r, warning = FALSE, message = FALSE}
shapiro.test(linear.model1$res)
lillie.test(linear.model1$res)
```
- Despite not being normally distributed, the linear regression model is pretty robust to violations of this assumption. 

## Logistic regression model
The structural form of the logistic regression model:

$$logit(p_i) = ln(\frac{p_i}{1 - p_i}) = \beta_0 + \beta_1 X_{1i}$$

* $Y_i$ denotes the outcome (or dependent) variable for subject $i$; this is a binary variable
* $X_{1i}$ denotes the predictor of interest or the independent variable ($X_1$) for subject $i$
* $\beta_0$ denotes the Y-intercept when $X$ is zero; this is not informative for logistic regression models
* $\beta_1$ denotes the slope or the change in $Y$ with a 1-unit change in $X$
* $p_{i}$ denotes the probability of the event occurring

- The logistic regression model is a predictive model for binary data.

- Hence, the logistic regression model can generate probabilities that a sample will have the discrete outcome given an input variable(s). 

- The logistic regression model uses maximum likelihood estimation (MLE) which is a conditional probability that classifies the outcome if a certain threshold is met (e.g,. > 0.50). 

- Hence, the probability range of a logistic regression model is between 0 and 1. 

- Additionally, the logistic regression can include multiple predictors which can be controlled or adjusted in a multivariable logistic regression model. 

```{r, message = FALSE, warning = FALSE, fig.align='center', fig.dim=c(6,4)}
library(LaplacesDemon)
x <- -10:10
prob <- invlogit(x)
plot(x, prob, type = "l", 
     main = "Logistic regression plot", 
     ylab = "Probability", xlab = "Values of X")
```

#### Example - Logistic regression 

- We'll use the `mtcar` data to build our logistic regression model.

```{r, message = FALSE, warning = FALSE} 
#### Load the libraries
library("ggplot2"); library("gmodels")
library("epitools"); library("tidyverse")
### Create factors
data1 <- within(mtcars, {
           vs <- factor(vs, labels = c("V", "S"))
           am <- factor(am, labels = c("automatic", "manual"))
})
# head(data1)
```

### Odds ratio calculation

```{r, message = FALSE, warning = FALSE}
oddsratio(data1$vs, data1$am, conf.level = 0.95, method = "wald")
```

- Using the odds ratio, vehicles with a "V" engine had a 2 times higher odds of having an automatic transmission (95% CI: 0.47, 8.40) compared to vehicles with a straight engine. 

### Logistic regression in R

- We can create a crude logistic regression model to estimate the odds ratio.

- We set the transmission type `am` as the dependent variable and the engine type `vs` as the independent variable. 

- The `glm()` command generates coefficients that are interpreted as the log odds of the event occuring. 

- We need to exponentiate this to get the odds ratio using the `exp()` command.

```{r, message = FALSE, warning = FALSE}
logit1<- glm(formula = am ~ vs, data = data1, 
             family = "binomial"(link = "logit"))
summary(logit1)
```

- According to the logistic regression model, vehicles with an "V" engine had a 2.0 times higher odds of having an automatic transmission (95% CI: 0.48, 8.76) compared to vehicles with a straight engine; this is not statistically significant since the odds ratio crosses the null or OR = 1. 

```{r, message = FALSE, warning = FALSE}
confint(logit1)### Generate the 95% CI
exp(coef(logit1))     ### Odds ratio
exp(confint(logit1))  ### 95% CI (odds ratio)
```

### Multivariable logistic regression model

- The structural form of the multivariable logistic regression model (this example uses two `X` variables):

$$logit(p_i) = ln(\frac{p_i}{1 - p_i}) = \beta_0 + \beta_1 X_{1i} + \beta_2 X_{2i}$$

- Since the logistic regression model can include both continuous and categorical predictors, we can add the engine type `vs` (V versus straight engine) and vehicle weight `wt`.

$$logit(p_i) = ln(\frac{p_i}{1 - p_i}) = \beta_0 + \beta_1 (vs)_{i} + \beta_2 (wt)_{i}$$
where `vs` is the engine type and `wt` is the vehicle weight. 

```{r, message = FALSE, warning = FALSE}
logit2 <- glm(formula = am ~ vs + wt, data = data1, 
              family = "binomial"(link = "logit"))
summary(logit2)
```

- In this multivariable logistic regression model, the association between engine type `vs` and transmission type `am` is much lower (OR = 0.01; 95% CI: 0.000005, 0.048) controlling for vehicle weight `wt` .

- Controlling for the vehicle's weight reduced odds of the association between engine type `vs` and transmission type `am`.

```{r, message = FALSE, warning = FALSE}
### Exponentiate the coefficients
exp(coef(logit2))     ### Odds ratio
exp(confint(logit2))  ### 95% CI (odds ratio)
```

### Comparison between models

- We compare the odds ratio between the crude and adjusted logistic regression models. 

#### example -- diabetes dataset

```{r, message = FALSE, warning = FALSE, echo=FALSE}
library("psych")
## Import file
diabetes1 <- read.csv("diabetes.data.csv", header = TRUE)
diabetes1$pregnancy.history <- ifelse(diabetes1$Pregnancies == 0, 0,1)
```

- There was a total of 657 (85.5%) subjects with a history of pregnancies and 111 (14.5%) subjects with no history of pregnancy.

```{r, message = FALSE, warning = FALSE}
### Create factors
data2 <- within(diabetes1, {
           pregnancy.history <- factor(pregnancy.history,
    labels = c("No history of pregnancies", "History of pregnancies"))
})
```

### Crude Logistic Regression Model

- We create a crude logistic regression model to evaluate the association of the subject's age `Age` on history of pregnancy `pregnancy.history`. 

```{r, message = FALSE, warning = FALSE}
logit3 <- glm(pregnancy.history ~ Age, data = data2, 
              family = "binomial"(link = "logit"))
summary(logit3)
```

### Multivariable logistic regression model
```{r, message = FALSE, warning = FALSE}
logit3 <- glm(pregnancy.history ~ Age + BMI + Glucose + SkinThickness,
              data = data2, family = "binomial"(link = "logit"))
summary(logit3)
```

### Models comparisons

- Based on the crude logistic regression model, a 1-unit increase in age was associated with a 7% increase in the odds of having a history of pregnancy (95% CI: 1.05, 1.10), which is statistically significant.

- The odds ratio describing the association between Age and History of pregnancy did not change much between the crude and adjusted logistic regression model.
