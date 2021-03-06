Assignment B-1
================

## 1. Introduction

This mini-project is part of the assignments in the STAT545B course. It
involves the following exercises:

1.  Making a function and fortifying it
2.  Documenting the function
3.  Demonstrating the usefulness of the function with examples
4.  Testing the function
5.  Making a README file for the project repository

Ultimately, a well written, reproducible Rmd file containing all the
code used in the exercises and relevant descriptions will be knitted to
a markdown file and pushed to the new Github repository created as part
of the course.

### 1.1. Load required packages

The packages below will be used to complete the exercises in this
project. Also, the `palmerpenguins::penguins`, `gapminder::gapminder`,
`datateachr::apt_buildings` datasets will be used. Therefore, I will
load them before I continue with the exercises.

However, if any of the packages is not installed, the function
`install.packages()` should be used for that purpose before loading.

``` r
suppressMessages(library(tidyverse))
suppressMessages(library(palmerpenguins))
suppressMessages(library(gapminder))
suppressMessages(library(testthat))
suppressMessages(library(datateachr))
suppressMessages(library(roxygen2))
```

## 2. Exercises 1 and 2: Make a function and document it

Here, I will make, fortify and document a function that takes a tibble
or data frame, computes summary statistics on a numeric variable grouped
by a categorical variable, and outputs a tibble or data frame and a
boxplot.

``` r
# Document function using the roxygen2 package

#' Compute summary statistics, produce a tibble or data frame and a boxplot
#' 
#' This function computes summary statistics on a numeric variable 
#' grouped by a categorical variable. The summary statistics include 
#' mean, median, minimum, maximum, and count. It also 
#' produces a boxplot.
#' 
#' @param df Tibble or data frame containing variables for computing summary statistics. 
#' Name is easily recognizable.
#' @param x A categorical variable by which rows are grouped. 
#' It is named x because it is more generally used and easy to identify.
#' @param y A numeric variable for computing summary statistics. 
#' It is named y because it is more generally used and easy to identify.
#' @return A tibble or data frame containing summary statistics and a boxplot.


summarize_data <- function(df, x, y, na.rm = TRUE) {
    val1 <- eval(substitute(x), df)
    val2 <- eval(substitute(y), df)

    if (!(is.character(val1) || is.factor(val1))) {
      stop('This function only works with a categorical input as x.\n',
           'You have provided an object of class: ', class(val1))
    }

    if (!is.numeric(val2)) {
      stop("This function only works with a numeric input as y.\n",
           "You have provided a variable of class: ", class(val2))
    }
  
    summary_df <- df %>%
    group_by({{ x }}) %>%
    summarise(mean = mean({{ y }}, na.rm = na.rm),
              median = median({{ y }}, na.rm = na.rm),
              min = min({{ y }}, na.rm = na.rm),
              max = max({{ y }}, na.rm = na.rm),
              n = n())
  
    summary_plot <- df %>% ggplot(aes({{ x }}, {{ y }})) +
    geom_boxplot()
  
    return(list(summary_df, summary_plot))
}
```

The new function takes a tibble or data frame, removes all missing
values by default (user can specify otherwise), groups the data by a
categorical variable, and calculates mean, median, minimum, maximum, and
count of a numeric variable. It returns a tibble or data frame and a
boxplot.

To fortify the function, I generalized it, included the option to
intentionally allow missing values by changing the default from
`na.rm = TRUE` to `na.rm = FALSE`. I also documented the function.

With the code established above, I can easily pass the new function
`summarize_data` to other datasets that meet the criteria set in the
code.

## 3. Exercise 3: Include examples

I will use the following examples to demonstrate the usage of the
`summarize_data` function.

``` r
# Example 1: Compute summary statistics using the palmerpenguins::penguins data

summarize_data(penguins, species, bill_length_mm)
```

    ## [[1]]
    ## # A tibble: 3 ?? 6
    ##   species    mean median   min   max     n
    ##   <fct>     <dbl>  <dbl> <dbl> <dbl> <int>
    ## 1 Adelie     38.8   38.8  32.1  46     152
    ## 2 Chinstrap  48.8   49.6  40.9  58      68
    ## 3 Gentoo     47.5   47.3  40.9  59.6   124
    ## 
    ## [[2]]

    ## Warning: Removed 2 rows containing non-finite values (stat_boxplot).

![](assignment_b_1_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

The function call above groups the penguins data by `species`, computes
summary statistics on `bill_length_mm`, produces a tibble and a boxplot.

``` r
# Example 2: Compute summary statistics using the gapminder::gapminder data

summarize_data(gapminder, continent, lifeExp)
```

    ## [[1]]
    ## # A tibble: 5 ?? 6
    ##   continent  mean median   min   max     n
    ##   <fct>     <dbl>  <dbl> <dbl> <dbl> <int>
    ## 1 Africa     48.9   47.8  23.6  76.4   624
    ## 2 Americas   64.7   67.0  37.6  80.7   300
    ## 3 Asia       60.1   61.8  28.8  82.6   396
    ## 4 Europe     71.9   72.2  43.6  81.8   360
    ## 5 Oceania    74.3   73.7  69.1  81.2    24
    ## 
    ## [[2]]

![](assignment_b_1_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

The function call above groups the `gapminder` data by `continent`,
computes summary statistics on `lifeExp`, produces a tibble and a
boxplot.

**Now, let me use a numeric variable in place of a categorical variable
for x and see what the outcome would be.**

``` r
# Example 3: Use a numeric variable in place of a categorical variable for x

summarize_data(gapminder, gdpPercap, lifeExp)
```

    ## Error in summarize_data(gapminder, gdpPercap, lifeExp): This function only works with a categorical input as x.
    ## You have provided an object of class: numeric

The function throws an error, since a numeric variable was used in place
of a categorical variable.

**I will also use a categorical variable where a numeric variable is
expected and see if the function would throw an error**

``` r
# Example 4: Use a categorical variable in place of a numeric variable for y

summarize_data(gapminder, continent, country)
```

    ## Error in summarize_data(gapminder, continent, country): This function only works with a numeric input as y.
    ## You have provided a variable of class: factor

## 4. Exercise 4: Test the function

I will use the `expect_()` function in the `test_that` package to test
the newly created `summarize_data()` function

``` r
# First, compute summary statistics manually
summary1 <- penguins %>% 
  group_by(species) %>%
  summarize(mean = mean(body_mass_g, na.rm = TRUE))
# Extract the mean column
  summary1$mean
```

    ## [1] 3700.662 3733.088 5076.016

``` r
# Compute summary statistics using the summarize_data function
summary2 <- summarize_data(penguins, species, body_mass_g)
# Extract the tibble, then the mean column
  summary2[[1]]$mean
```

    ## [1] 3700.662 3733.088 5076.016

``` r
test_that("the two means are identical", {
  expect_identical(summary1$mean, summary2[[1]]$mean)})
```

    ## Test passed ????

``` r
test_that("the function returns a list", {
  expect_type(summarize_data(apt_buildings, window_type, no_of_storeys), "list")})
```

    ## Test passed ????

``` r
test_that("the use of a numeric variable for x returns an error", {
  expect_error(summarize_data(apt_buildings, year_built, no_of_storeys))})
```

    ## Test passed ????

``` r
test_that("the two mean vectors have NAs", {
  na_present_1 <- penguins %>% 
                  group_by(species) %>% 
                  summarise(mean = mean(body_mass_g))
  na_present_2 <- summarize_data(penguins, species, body_mass_g, na.rm = FALSE)
  expect_equal(na_present_1$mean, na_present_2[[1]]$mean)})
```

    ## Test passed ????

``` r
test_that("silent when function successfully creates a boxplot", {
  plot <- summarize_data(gapminder, continent, lifeExp)
  expect_silent(print(plot[[2]]))})
```

![](assignment_b_1_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

    ## Test passed ????

**I used 5 `expect_()` functions to test the `summarize_data` function
and the tests passed.**

**First,** I computed summary statistics manually, extracted the mean. I
also computed summary statistics using the `summarize_data` function and
extracted the mean. I then used `expect_identical()` to test if the two
means are identical.

**Second,** I used `expect_type()` to test if the `summarize_data`
function returns a `list`.

**Third,** I used `expect_error()` to test if using a numeric variable
instead of a categorical variable in the `summarize_data` function would
pass. The test passed because `expect_error()` is expecting an error.

**Fourth,** I computed summary statistics manually, extracted the mean
vector, this time, not removing `NAs`. I also used the `summarize_data`
function to compute summary statistics, with `na.rm = FALSE`, and then
extracted the mean vector. I used `expect_equal()` to test if the two
mean vectors have `NAs`.

**Fifth,** I used `expect_silent()` to test if the `summarize_data`
function successfully creates a boxplot.

**References**

1.  <https://www.youtube.com/watch?v=3nDgR7l5Tps>
2.  <http://adv-r.had.co.nz/Computing-on-the-language.html>
3.  <https://stackoverflow.com/questions/53761025/>
    testthatexpect-silent-does-not-seem-to-notice-ggplot2-errors
