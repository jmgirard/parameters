
# parameters <img src='man/figures/logo.png' align="right" height="139" />

[![CRAN](http://www.r-pkg.org/badges/version/parameters)](https://cran.r-project.org/package=parameters)
[![downloads](http://cranlogs.r-pkg.org/badges/parameters)](https://cran.r-project.org/package=parameters)
[![Build
Status](https://travis-ci.org/easystats/parameters.svg?branch=master)](https://travis-ci.org/easystats/parameters)
[![codecov](https://codecov.io/gh/easystats/parameters/branch/master/graph/badge.svg)](https://codecov.io/gh/easystats/parameters)

***Describe and understand your model’s parameters\!***

`parameters`’s primary goal is to provide utilities for processing the
parameters of various statistical models. Beyond computing *p* values,
**CIs**, and other indices for a wide variety of models, this package
implements features like **standardization** or **bootstrapping** of
parameters and models, **feature reduction** (feature extraction and
variable selection) as well as conversion between indices of **effect
size**.

## Installation

Run the following:

``` r
install.packages("devtools")
devtools::install_github("easystats/parameters")
```

``` r
library("parameters")
```

## Documentation

[![Documentation](https://img.shields.io/badge/documentation-parameters-orange.svg?colorB=E91E63)](https://easystats.github.io/parameters/)
[![Blog](https://img.shields.io/badge/blog-easystats-orange.svg?colorB=FF9800)](https://easystats.github.io/blog/posts/)
[![Features](https://img.shields.io/badge/features-parameters-orange.svg?colorB=2196F3)](https://easystats.github.io/parameters/reference/index.html)

Click on the buttons above to access the package
[**documentation**](https://easystats.github.io/parameters/) and the
[**easystats blog**](https://easystats.github.io/blog/posts/), and
check-out these vignettes:

#### Parameters Engineering

  - [Bootstrapped
    parameters](https://easystats.github.io/parameters/articles/bootstrapping.html)
  - [Standardized
    parameters](https://easystats.github.io/parameters/articles/standardization.html)

#### Variable/Feature Reduction

  - [How many factors to retain in Factor Analysis
    (FA)](https://easystats.github.io/parameters/articles/n_factors.html)
  - [Variable extraction (PCA, FA,
    …)](https://easystats.github.io/parameters/articles/variable_extraction.html)
  - [Variable selection (stepwise, projpred,
    …)](https://easystats.github.io/parameters/articles/variable_selection.html)

# Features

## Model’s parameters description

<img src='man/figures/figure1.png' align="center" />

The `model_parameters` function allows you to extract the parameters and
their characteristics from various models in a consistent way. It can be
considered as a lightweight alternative to
[`broom::tidy()`](https://github.com/tidymodels/broom), with some
notable differences:

  - The names of the returned dataframe are **specific** to their
    content. For instance, the column containing the statistic is named
    following the statistic name, *i.e.*, *t*, *z*, etc., instead of a
    generic name such as *statistic*.
  - It is able to compute or extract indices not available by default,
    such as ***p* values**, **CIs**, etc.
  - It includes **feature engineering** capabilities, including
    [**bootstrapping**](https://easystats.github.io/parameters/articles/bootstrapping.html)
    and
    [**standardization**](https://easystats.github.io/parameters/articles/standardization.html)
    of parameters.

### Correlations

#### Frequentist

``` r
model <- cor.test(iris$Sepal.Length, iris$Sepal.Width)
model_parameters(model)
```

| Parameter1        | Parameter2       | r      | t      | DoF | p    | 95% CI          | Method  |
| :---------------- | :--------------- | :----- | :----- | --: | :--- | :-------------- | :------ |
| iris$Sepal.Length | iris$Sepal.Width | \-0.12 | \-1.44 | 148 | .152 | \[-0.27, 0.04\] | Pearson |

#### Bayesian

``` r
library(BayesFactor)

model <- BayesFactor::correlationBF(iris$Sepal.Length, iris$Sepal.Width)
model_parameters(model)
```

| Parameter | Median | 89% CI          | pd     | % in ROPE | Prior              | BF   |
| :-------- | :----- | :-------------- | :----- | :-------- | :----------------- | :--- |
| rho       | \-0.11 | \[-0.25, 0.01\] | 91.75% | 43.36%    | Cauchy (0 +- 0.33) | 0.51 |

### t-tests

#### Frequentist

``` r
df <- iris
df$Sepal.Big <- ifelse(df$Sepal.Width >= 3, "Yes", "No")

model <- t.test(Sepal.Length ~ Sepal.Big, data=df)
model_parameters(model)
```

| Parameter    | Group     | Mean\_Group1 | Mean\_Group2 | Difference | t    | DoF    | p    | 95% CI          | Method                  |
| :----------- | :-------- | :----------- | :----------- | :--------- | :--- | :----- | :--- | :-------------- | :---------------------- |
| Sepal.Length | Sepal.Big | 5.95         | 5.78         | \-0.18     | 1.36 | 142.15 | .177 | \[-0.08, 0.43\] | Welch Two Sample t-test |

#### Bayesian

``` r
model <- BayesFactor::ttestBF(formula = Sepal.Length ~ Sepal.Big, data=df)
model_parameters(model)
```

| Parameter  | Median | 89% CI         | pd   | % in ROPE | Prior              | BF   |
| :--------- | :----- | :------------- | :--- | :-------- | :----------------- | :--- |
| Difference | 5.86   | \[5.75, 5.97\] | 100% | 0%        | Cauchy (0 +- 0.71) | 0.38 |

### ANOVAs

#### Simple

``` r
model <- aov(Sepal.Length ~ Sepal.Big, data = df)
model_parameters(model, omega_squared = TRUE)
```

| Parameter | Sum\_Squares | DoF | Mean\_Square | F    | p    | Omega\_Sq |
| :-------- | :----------- | --: | :----------- | :--- | :--- | --------: |
| Sepal.Big | 1.10         |   1 | 1.10         | 1.61 | .207 |         0 |
| Residuals | 101.07       | 148 | 0.68         |      |      |           |

#### Repeated measures

``` r
model <- aov(Sepal.Length ~ Sepal.Big + Error(Species), data = df)
model_parameters(model)
```

| Group   | Parameter | Sum\_Squares | DoF | Mean\_Square | F     | p       |
| :------ | :-------- | :----------- | --: | :----------- | :---- | :------ |
| Species | Sepal.Big | 28.27        |   1 | 28.27        | 0.81  | .534    |
| Species | Residuals | 34.94        |   1 | 34.94        |       |         |
| Within  | Sepal.Big | 4.74         |   1 | 4.74         | 20.24 | \< .001 |
| Within  | Residuals | 34.21        | 146 | 0.23         |       |         |

``` r
library(lme4)
model <- anova(lmer(Sepal.Length ~ Sepal.Big + (1 | Species), data = df))
model_parameters(model)
```

| Parameter | Sum\_Squares | DoF | Mean\_Square | F     |
| :-------- | :----------- | --: | :----------- | :---- |
| Sepal.Big | 4.64         |   1 | 4.64         | 19.82 |

### General Linear Models (GLM)

``` r
model <- glm(vs ~ wt + cyl, data = mtcars, family = "binomial")
model_parameters(model, standardize = "refit")
```

| Parameter   | Coefficient | SE   | 95% CI           | z      | DoF (residual) | p    | Coefficient (std.) |
| :---------- | :---------- | :--- | :--------------- | :----- | -------------: | :--- | :----------------- |
| (Intercept) | 10.62       | 4.17 | \[4.79, 22.66\]  | 2.55   |             29 | .011 | \-0.76             |
| wt          | 2.10        | 1.55 | \[-0.53, 6.24\]  | 1.36   |             29 | .174 | 2.05               |
| cyl         | \-2.93      | 1.38 | \[-6.92, -1.07\] | \-2.12 |             29 | .034 | \-5.23             |

### Bootstrapped models

``` r
model <- lm(mpg ~ drat * cyl, data = mtcars)
model_parameters(model, bootstrap = TRUE)
```

|   | Parameter   | Coefficient | 95% CI            | p    |
| - | :---------- | :---------- | :---------------- | :--- |
| 1 | (Intercept) | 0.93        | \[-43.74, 37.46\] | .941 |
| 3 | drat        | 9.31        | \[-0.49, 20.92\]  | .066 |
| 2 | cyl         | 1.95        | \[-3.37, 8.29\]   | .408 |
| 4 | drat \* cyl | \-1.20      | \[-2.95, 0.19\]   | .074 |

### Mixed models

``` r
library(lme4)

model <- lmer(Sepal.Width ~ Petal.Length + (1|Species), data = iris)
model_parameters(model, standardize = "refit")
```

| Parameter    | Coefficient | SE   | 95% CI          | t    | p       | Coefficient (std.) |
| :----------- | :---------- | :--- | :-------------- | :--- | :------ | :----------------- |
| (Intercept)  | 2.00        | 0.56 | \[-1.92, 5.92\] | 3.56 | \< .001 | 0.00               |
| Petal.Length | 0.28        | 0.06 | \[-0.27, 0.83\] | 4.75 | \< .001 | 1.14               |

### Bayesian models

``` r
library(rstanarm)

model <- stan_glm(mpg ~ wt + cyl, data = mtcars)
model_parameters(model)
```

|   | Parameter   | Median | 89% CI           | pd     | % in ROPE |  ESS | Rhat  | Prior               |
| - | :---------- | :----- | :--------------- | :----- | :-------- | ---: | :---- | :------------------ |
| 1 | (Intercept) | 39.64  | \[36.69, 42.52\] | 100%   | 0%        | 5250 | 0.999 | Normal (0 +- 60.27) |
| 3 | wt          | \-3.19 | \[-4.38, -1.92\] | 100%   | 0.05%     | 1963 | 1.000 | Normal (0 +- 15.40) |
| 2 | cyl         | \-1.51 | \[-2.21, -0.84\] | 99.98% | 1.88%     | 1997 | 1.000 | Normal (0 +- 8.44)  |

### Exploratory Factor Analysis (EFA) and Principal Component Analysis (PCA)

``` r
library(psych)

model <- psych::fa(attitude, nfactors = 3)
model_parameters(model)
```

| Variable   |   MR1 |   MR2 |   MR3 | Complexity | Uniqueness |
| :--------- | ----: | ----: | ----: | ---------: | ---------: |
| rating     |   0.9 | \-0.1 |   0.0 |          1 |        0.2 |
| complaints |   1.0 | \-0.1 |   0.0 |          1 |        0.1 |
| privileges |   0.4 |   0.3 | \-0.1 |          2 |        0.6 |
| learning   |   0.5 |   0.5 | \-0.3 |          2 |        0.2 |
| raises     |   0.5 |   0.4 |   0.3 |          2 |        0.2 |
| critical   |   0.2 |   0.2 |   0.5 |          2 |        0.7 |
| advance    | \-0.1 |   0.9 |   0.1 |          1 |        0.2 |

### Confirmatory Factor Analysis (CFA) and Structural Equation Models (SEM)

``` r
library(lavaan)

model <- lavaan::cfa(' visual  =~ x1 + x2 + x3
                       textual =~ x4 + x5 + x6
                       speed   =~ x7 + x8 + x9 ', 
                     data=HolzingerSwineford1939)
model_parameters(model)
```

|    | Link                | Coefficient | SE   | 95% CI         | p       | Type        |
| -- | :------------------ | :---------- | :--- | :------------- | :------ | :---------- |
| 1  | visual =\~ x1       | 1.00        | 0.00 | \[1.00, 1.00\] | \< .001 | Loading     |
| 2  | visual =\~ x2       | 0.55        | 0.10 | \[0.36, 0.75\] | \< .001 | Loading     |
| 3  | visual =\~ x3       | 0.73        | 0.11 | \[0.52, 0.94\] | \< .001 | Loading     |
| 4  | textual =\~ x4      | 1.00        | 0.00 | \[1.00, 1.00\] | \< .001 | Loading     |
| 5  | textual =\~ x5      | 1.11        | 0.07 | \[0.98, 1.24\] | \< .001 | Loading     |
| 6  | textual =\~ x6      | 0.93        | 0.06 | \[0.82, 1.03\] | \< .001 | Loading     |
| 7  | speed =\~ x7        | 1.00        | 0.00 | \[1.00, 1.00\] | \< .001 | Loading     |
| 8  | speed =\~ x8        | 1.18        | 0.16 | \[0.86, 1.50\] | \< .001 | Loading     |
| 9  | speed =\~ x9        | 1.08        | 0.15 | \[0.79, 1.38\] | \< .001 | Loading     |
| 22 | visual \~\~ textual | 0.41        | 0.07 | \[0.26, 0.55\] | \< .001 | Correlation |
| 23 | visual \~\~ speed   | 0.26        | 0.06 | \[0.15, 0.37\] | \< .001 | Correlation |
| 24 | textual \~\~ speed  | 0.17        | 0.05 | \[0.08, 0.27\] | \< .001 | Correlation |

## Variable and parameters selection

<img src='man/figures/figure2.png' align="center" />

### General Linear Models (GLM)

``` r
library(dplyr)

lm(disp ~ ., data = mtcars) %>% 
  parameters_selection() %>% 
  model_parameters()
```

| Parameter   | Coefficient |    SE | CI\_low | CI\_high |   t | DoF\_residual |   p | Std\_Coefficient |
| :---------- | ----------: | ----: | ------: | -------: | --: | ------------: | --: | ---------------: |
| (Intercept) |       141.7 | 125.7 | \-116.6 |      400 |   1 |            26 | 0.3 |              0.0 |
| cyl         |        13.1 |   7.9 |   \-3.1 |       29 |   2 |            26 | 0.1 |              0.2 |
| hp          |         0.6 |   0.2 |     0.2 |        1 |   3 |            26 | 0.0 |              0.3 |
| wt          |        80.5 |  12.2 |    55.3 |      106 |   7 |            26 | 0.0 |              0.6 |
| qsec        |      \-14.7 |   6.1 |  \-27.3 |      \-2 | \-2 |            26 | 0.0 |            \-0.2 |
| carb        |      \-28.8 |   5.6 |  \-40.3 |     \-17 | \-5 |            26 | 0.0 |            \-0.4 |

### Mixed models

``` r
library(lme4)

lmer(Sepal.Length ~ Sepal.Width * Petal.Length * Petal.Width + (1|Species), data = iris)  %>%
  parameters_selection() %>%
  model_parameters()
```

| Parameter                            | Coefficient |  SE | CI\_low | CI\_high |     t |   p | Std\_Coefficient |
| :----------------------------------- | ----------: | --: | ------: | -------: | ----: | --: | ---------------: |
| (Intercept)                          |         1.8 | 0.9 |   \-1.7 |      5.3 |   1.9 | 0.1 |            \-0.2 |
| Petal.Length                         |         0.9 | 0.5 |   \-0.8 |      2.6 |   1.8 | 0.1 |              0.3 |
| Petal.Length:Petal.Width             |         0.3 | 0.2 |   \-0.3 |      0.8 |   1.2 | 0.2 |              1.6 |
| Petal.Width                          |       \-2.0 | 1.5 |     2.0 |    \-6.0 | \-1.3 | 0.2 |            \-0.4 |
| Sepal.Width                          |         0.8 | 0.3 |   \-0.7 |      2.2 |   2.8 | 0.0 |            \-0.1 |
| Sepal.Width:Petal.Length             |       \-0.1 | 0.2 |     0.1 |    \-0.3 | \-0.6 | 0.5 |              0.1 |
| Sepal.Width:Petal.Length:Petal.Width |         0.0 | 0.1 |     0.0 |    \-0.1 | \-0.6 | 0.5 |              0.2 |
| Sepal.Width:Petal.Width              |         0.3 | 0.5 |   \-0.3 |      1.0 |   0.7 | 0.5 |              0.0 |

### Bayesian models

``` r
library(rstanarm)

model <- stan_glm(mpg ~ ., data = mtcars) %>% 
  parameters_selection() %>% 
  model_parameters()
```

|   | Parameter   | Median | CI\_low | CI\_high |  pd | ROPE\_Percentage |  ESS | Rhat | Prior\_Distribution | Prior\_Location | Prior\_Scale |
| - | :---------- | -----: | ------: | -------: | --: | ---------------: | ---: | ---: | :------------------ | --------------: | -----------: |
| 1 | (Intercept) |   20.2 |   \-1.9 |     42.1 | 0.9 |              0.0 | 2483 |    1 | normal              |               0 |         60.3 |
| 7 | wt          |  \-3.9 |   \-5.9 |    \-1.9 | 1.0 |              0.0 | 2860 |    1 | normal              |               0 |         15.4 |
| 3 | cyl         |  \-0.5 |   \-1.8 |      0.8 | 0.7 |              0.5 | 2781 |    1 | normal              |               0 |          8.4 |
| 5 | hp          |    0.0 |     0.0 |      0.0 | 0.9 |              1.0 | 2907 |    1 | normal              |               0 |          0.2 |
| 2 | am          |    2.9 |     0.2 |      5.9 | 1.0 |              0.1 | 3082 |    1 | normal              |               0 |         15.1 |
| 6 | qsec        |    0.8 |   \-0.2 |      1.7 | 0.9 |              0.4 | 2449 |    1 | normal              |               0 |          8.4 |
| 4 | disp        |    0.0 |     0.0 |      0.0 | 0.9 |              1.0 | 2750 |    1 | normal              |               0 |          0.1 |

## Variable and features extraction

### How many factors to retain in Factor Analysis (FA)

``` r
n_factors(mtcars)
```

## Miscellaneous

### Describe a Distribution

``` r
x <- rnorm(300)
describe_distribution(x)
>     Mean SD Min Max Skewness Kurtosis   n n_Missing
> 1 -0.007  1  -3   3      0.1     -0.1 300         0
```

### Standardization and normalization

``` r
df <- standardize(iris)
summary(df$Sepal.Length)
>    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
>    -1.9    -0.9    -0.1     0.0     0.7     2.5

df <- normalize(iris)
summary(df$Sepal.Length)
>    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
>     0.0     0.2     0.4     0.4     0.6     1.0
```
