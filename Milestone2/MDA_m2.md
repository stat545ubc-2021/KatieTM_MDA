Mini Data Analysis: Milestone 2
================
Katie Tjaden-McClement
October 19, 2021

-   [Introduction](#introduction)
-   [**Task 1**](#task-1)
    -   [1.1 Research Questions](#11-research-questions)
    -   [1.2 Summarizing & Graphing](#12-summarizing--graphing)
        -   [1.2.1 Root barriers vs. tree
            size](#121-root-barriers-vs-tree-size)
        -   [1.2.2 Tree biomass across
            neighbourhoods](#122-tree-biomass-across-neighbourhoods)
        -   [1.2.3 Distribution of cherry
            trees](#123-distribution-of-cherry-trees)
        -   [1.2.4 Tree area vs. size](#124-tree-area-vs-size)
    -   [1.3](#13)
-   [**Task 2**](#task-2)
    -   [2.1 Is my data tidy?](#21-is-my-data-tidy)

### Introduction

This is the R markdown document for milestone 2 of the mini data
analysis project for STAT 545A. This milestone of the project builds on
the work done in the first milestone (*MDA\_m1.Rmd*) to gain experience
working with *dplyr* and *tidyr* to handle datasets and adress research
questions. This milestone focuses on the principle of tidy data and how
datasets can be converted to tidy formats for a given research question.

In Milestone 1 I chose the *vancouver\_trees* dataset from the
*datateachr* package to work with for the Mini Data Analysis, and
explored it through various visualizations.

## **Task 1**

### 1.1 Research Questions

These are the 4 research questions I proposed for the *vancouver\_trees*
dataset in milestone 1:

1.  How does having a root barrier relate to tree size as measured by
    diameter and height?
2.  How does tree biomass (measured as a combination of number of trees
    and their size) vary across neighbourhoods?
3.  What is the spatial distribution of cherry trees in Vancouver?
4.  What is the relationship between plant\_area (whether the tree is in
    a sidewalk cutout, gate, behind sidewalk, or in boulevards or
    varying widths) and tree size as measured by diameter and height?

Before getting into processing and summarizing my data, I will load it
into my R environment using the more concise name “trees”. I will also
trim outliers for diameter as discuessed in milestone 1.

``` r
trees <- vancouver_trees
trees <- filter(trees, diameter <= 75)
```

### 1.2 Summarizing & Graphing

#### 1.2.1 Root barriers vs. tree size

To summarize the data for question 1, I will compute the range, mean,
median, and standard deviation of tree diameter for trees with and
without root barriers

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
    ## 1 N            11.9          0          75       10  9.03 137422
    ## 2 Y             4.38         0.5        54.5      3  2.75   9154

Hmm we can see from the minimum that there are trees with a diameter of
0, which can’t be right. Let’s see how many there are and remove them
from the dataset

``` r
diameter0 <- filter(trees, diameter == 0)
nrow(diameter0) # there are 92!
```

    ## [1] 92

``` r
trees <- filter(trees, diameter != 0)
```

Now lets repeat the summary statistics:

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

Let’s visualize these differences in diameter between trees with and
without root barriers using a boxplot overlayed with density plots to
get a better sense of the full spread of the data

``` r
ggplot(trees) +
  geom_density(aes(x = diameter, fill = root_barrier), 
              alpha = 0.4) +
  geom_boxplot(aes(x = diameter, y = root_barrier, fill = root_barrier), 
               width = 0.5, alpha = 0.9) +
  scale_x_log10() + # changes the x axis to a logarithmic scale which more clearly shows the distribution
  scale_fill_brewer(palette = "Set1") + # specifies a colour palette from the rcolourbrewer package (in tidyverse)
  labs(x = "Tree diameter",
       fill = "Root barrier",
       y = "") + # removes y axis label because it is redundant to fill label
  theme(legend.position = c(0.9, 0.8)) # positions the legend in the top right corner of the graph that was blank space anyway
```

![](MDA_m2_files/figure-gfm/diameter_barrier_boxplot_density-1.png)<!-- -->

#### 1.2.2 Tree biomass across neighbourhoods

To look at how tree biomass varies across neighbourhoods, I will find
out how many observations (= number of trees, since each row of the data
frame is a different tree) there are across the different
neighbourhoods. Since I want to measure biomass, which is a combination
of tree sizes and number of trees, this will give me an idea of how many
trees are in each neighbourhood, but won’t get at the total biomass yet.

``` r
(trees_neighbourhood <- trees %>% 
  group_by(neighbourhood_name) %>% 
  summarise(n = n()))
```

    ## # A tibble: 22 × 2
    ##    neighbourhood_name           n
    ##    <chr>                    <int>
    ##  1 ARBUTUS-RIDGE             5166
    ##  2 DOWNTOWN                  5157
    ##  3 DUNBAR-SOUTHLANDS         9392
    ##  4 FAIRVIEW                  4001
    ##  5 GRANDVIEW-WOODLAND        6699
    ##  6 HASTINGS-SUNRISE         10544
    ##  7 KENSINGTON-CEDAR COTTAGE 11033
    ##  8 KERRISDALE                6931
    ##  9 KILLARNEY                 6142
    ## 10 KITSILANO                 8105
    ## # … with 12 more rows

``` r
mean(trees_neighbourhood$n)
```

    ## [1] 6658.364

``` r
max(trees_neighbourhood$n)
```

    ## [1] 11380

Of the 22 neighborhoods in this dataset, all have at least 2500 trees,
with an average of 6658 trees and a maximum of just over 11 000 trees in
Renfrew-Collingwood.

Let’s look at the average heights of trees across neighbourhoods, first
sorting the neighbourhood name factor from least to most trees to better
see any trends in tree height and number of trees in neighbourhood

``` r
trees %>% 
  mutate(neighbourhood_name = fct_rev(fct_infreq(neighbourhood_name))) %>% #reorder the factor by increasing frequency of neighbourhood
  group_by(neighbourhood_name) %>% 
  mutate(neighbourhood_count = n()) %>% # add a column to the tibble with tree count for each neighbouhood
  ggplot(aes(x = neighbourhood_name, y = height_range_id,
             colour = neighbourhood_count), #
         alpha = 0.1) +
  geom_boxplot() +
  coord_flip() + # flips the x and y axes so that the neighbourhood names fit
  labs(y = "Tree height range", x = "Neighbourhood name",
       colour = "# of trees")
```

![](MDA_m2_files/figure-gfm/Tree_height_neighbourhood-1.png)<!-- -->
There does not seem to be a relationship between number of trees and
tree height in a neighbourhood.

#### 1.2.3 Distribution of cherry trees

#### 1.2.4 Tree area vs. size

### 1.3

## **Task 2**

### 2.1 Is my data tidy?

Tidy data: \* each row is an observation \* each column is a variable \*
each cell is a value

To determine if the *vancouver\_trees* dataset is tidy, let’s take a
