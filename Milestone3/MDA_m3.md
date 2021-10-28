Mini Data Analysis: Milestone 3
================
Katie Tjaden-McClement
October 28, 2021

-   [Introduction](#introduction)
-   [Setup](#setup)
-   [Exercise 1: Special Data Types](#exercise-1-special-data-types)
-   [Exercise 2: Modelling](#exercise-2-modelling)
    -   [2.0](#20)
    -   [2.1](#21)
    -   [2.2](#22)
-   [Exercise 3: Reading and writing
    data](#exercise-3-reading-and-writing-data)
    -   [3.1](#31)
    -   [3.2](#32)

### Introduction

This is the 3rd and final milestone of the Mini Data Analysis project
for STAT 545A. Milestone 1 focused on exploring datasets and coming up
with research questions, milestone 2 used data wrangling and
visualizations to begin to answer research questions. This milestone
will continue the work from the first two by:

-   Exploring special data types
-   Fitting models to answer research questions
-   Reading and writing data in a robust manner

*Note: this Rmd was copied directly from the stat545 website as
instructed, but instructions and other materials have been removed for
conciseness and to maintain consistency of style within the MDA*

### Setup

Begin by loading your data and the tidyverse package below:

``` r
library(datateachr) # <- might contain the data you picked!
library(tidyverse)
library(lubridate) # for working with date data
library(broom) # produces tidy outputs of model objects

theme_set(theme_classic()) #sets theme for all plots as classic, removing backgrounds and gridlines
```

From Milestone 2, I chose two research questions:

1.  How does having a root barrier relate to tree size as measured by
    diameter and height?
2.  How does tree biomass (measured as a combination of number of trees
    and their size) vary across neighbourhoods?

## Exercise 1: Special Data Types

For this exercise, you’ll be choosing two of the three tasks below.

But first, tasks 1 and 2 below ask you to modify a plot you made in a
previous milestone. The plot you choose should involve plotting across
at least three groups (whether by facetting, or using an aesthetic like
colour). Place this plot below.

``` r
# Prepare trees dataset as in milestones 1 and 2:
trees <- vancouver_trees %>% 
  filter(diameter <= 75,
         diameter != 0)

trees %>% 
  mutate(neighbourhood_name = fct_rev(fct_infreq(neighbourhood_name))) %>% #reorder the factor by increasing frequency of neighbourhood
  group_by(neighbourhood_name) %>% 
  mutate(neighbourhood_count = n()) %>% # add a column to the tibble with tree count for each neighbouhood
  ggplot(aes(x = neighbourhood_name, y = height_range_id,
             colour = neighbourhood_count), # colour with a continuous scale according to # of trees in the neighbourhood
         alpha = 0.1) +
  geom_boxplot() +
  coord_flip() + # flips the x and y axes so that the neighbourhood names fit
  labs(y = "Tree height range", x = "Neighbourhood name",
       colour = "# of trees")
```

![](MDA_m3_files/figure-gfm/M2%20Tree_height_neighbourhood-1.png)<!-- -->

I have chosen to complete tasks 1 and 3.

**Task 1.** Produce a new plot that reorders a factor in your original
plot, using the `forcats` package. Then, in a sentence or two, briefly
explain why you chose this ordering.

I am reordering this plot to arrange the neighbourhoods by their mean
tree height. This will allow a viewer to easily see which
neightbourhoods have the tallest and shortest trees on average, because
they will be at the top and the bottom of the plot, respectively.

``` r
trees %>% 
  mutate(neighbourhood_name = fct_reorder(neighbourhood_name, height_range_id, mean)) %>% #reorder the factor by mean tree height
  group_by(neighbourhood_name) %>% 
  mutate(neighbourhood_count = n()) %>% # add a column to the tibble with tree count for each neighbouhood
  ggplot(aes(x = neighbourhood_name, y = height_range_id,
             colour = neighbourhood_count), # colour with a continuous scale according to # of trees in the neighbourhood
         alpha = 0.1) +
  geom_boxplot() +
  coord_flip() + # flips the x and y axes so that the neighbourhood names fit
  labs(y = "Tree height range", x = "Neighbourhood name",
       colour = "# of trees")
```

![](MDA_m3_files/figure-gfm/Tree%20height%20neighbourhood%20reordered-1.png)<!-- -->

This updated plot shows that Kitsilano and Shaugnessy have the tallest
trees on average, while Renfrew-Collingwood and Oakridge have the
shortest trees. Presenting the neighbourhoods in this order also
highlights that there isn’t a clear relationship between number of trees
in a neighbourhood and their height, as some of the neighbourhoods with
the most trees show up in the bottom half of this plot.

**Task 3.** If your data has some sort of time-based column like a date
(but something more granular than just a year): 1. Make a new column
that uses a function from the `lubridate` or `tsibble` package to modify
your original time-based column. 2. Then, in a sentence or two, explain
how your new column might be useful in exploring a research question.

``` r
# First let's check out the class of the date_planted column and get a look at what the data looks like:
str(trees$date_planted) # "Date" class with YYYY-MM-DD date format
```

    ##  Date[1:146484], format: "1999-01-13" "1996-05-31" "1993-11-22" "1996-04-29" "1993-12-17" NA ...

``` r
trees_month <- trees %>% 
  mutate(month_planted = month(date_planted, #extracts the month from the date planted data
                               label = TRUE), #labels month as ordered factor of written out month names, rather than numbers
                               abbr = TRUE) #abbreviates the month names
head(trees_month$month_planted)
```

    ## [1] Jan  May  Nov  Apr  Dec  <NA>
    ## 12 Levels: Jan < Feb < Mar < Apr < May < Jun < Jul < Aug < Sep < ... < Dec

This new month\_planted variable could be used to investigate whether
trees planted in certain months tend to have better outcomes e.g. larger
diameters 15 years after planting, and if this is a species specific
response. This line of questioning could lead to recommendations for
best practices for what time of year to plant which species.

## Exercise 2: Modelling

### 2.0

Pick a research question, and pick a variable of interest that’s
relevant to the research question. Indicate these.

**Research Question**: How does having a root barrier relate to tree
size as measured by diameter?

**Variable of interest**: Tree diameter

### 2.1

Fit a model or run a hypothesis test that provides insight on this
variable with respect to the research question. Store the model object
as a variable, and print its output to screen.

I will run a t-test to determine if the mean diameter of trees with root
barriers is different from those without:

``` r
root_barrier_ttest <- t.test(formula = diameter ~ root_barrier, data = trees)
root_barrier_ttest #print model object
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  diameter by root_barrier
    ## t = 200.83, df = 26208, p-value < 2.2e-16
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  7.485686 7.633245
    ## sample estimates:
    ## mean in group N mean in group Y 
    ##       11.942017        4.382552

### 2.2

Produce something relevant from your fitted model: either predictions on
Y, or a single value like a regression coefficient or a p-value.

The *tidy* function from the broom package converts a model output into
a tibble that is easy to read and interpret. The value I want to
highlight in this tibble is the t statistic, which is in the “statistic”
column.

``` r
tidy(root_barrier_ttest)
```

    ## # A tibble: 1 × 10
    ##   estimate estimate1 estimate2 statistic p.value parameter conf.low conf.high
    ##      <dbl>     <dbl>     <dbl>     <dbl>   <dbl>     <dbl>    <dbl>     <dbl>
    ## 1     7.56      11.9      4.38      201.       0    26208.     7.49      7.63
    ## # … with 2 more variables: method <chr>, alternative <chr>

# Exercise 3: Reading and writing data

Get set up for this exercise by making a folder called `output` in the
top level of your project folder / repository. You’ll be saving things
there.

First let’s make sure the *here* function is correctly identifying my
project location:

``` r
here::here() # yes, this is correct!
```

    ## [1] "/Users/Katie/Library/Mobile Documents/com~apple~CloudDocs/Documents/UBC/MSc/Coursework/STAT 545/KatieTM_MDA"

Now let’s create an “Output” folder:

``` r
dir.create(here::here("Output"))
```

### 3.1

Take a summary table that you made from Milestone 2 (Exercise 1.2), and
write it as a csv file in your `output` folder. Use the `here::here()`
function.

Table of summary statistics for trees with and without root barriers
from milestone 2:

``` r
q1_summary <- trees %>% 
  group_by(root_barrier) %>% 
  summarise(mean = mean(diameter, na.rm = T),
            range_lower = min(diameter),
            range_upper = max(diameter),
            median = median(diameter, na.rm = T),
            sd = sd(diameter),
            n = n())
q1_summary
```

    ## # A tibble: 2 × 7
    ##   root_barrier  mean range_lower range_upper median    sd      n
    ##   <chr>        <dbl>       <dbl>       <dbl>  <dbl> <dbl>  <int>
    ## 1 N            11.9         0.25        75       10  9.03 137330
    ## 2 Y             4.38        0.5         54.5      3  2.75   9154

Write as csv file in the Output folder:

``` r
write_csv(q1_summary, here::here("Output", "q1_summary.csv"))
```

### 3.2

Write your model object from Exercise 2 to an R binary file (an RDS),
and load it again. Be sure to save the binary file in your `output`
folder. Use the functions `saveRDS()` and `readRDS()`.

Write to RDS file in the output folder:

``` r
saveRDS(root_barrier_ttest, here::here("Output", "root_barrier_ttest.RDS"))
```

Load back in:

``` r
readRDS(here::here("Output", "root_barrier_ttest.RDS"))
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  diameter by root_barrier
    ## t = 200.83, df = 26208, p-value < 2.2e-16
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  7.485686 7.633245
    ## sample estimates:
    ## mean in group N mean in group Y 
    ##       11.942017        4.382552
