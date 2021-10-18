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
    -   [1.3 Progress on research
        questions](#13-progress-on-research-questions)
-   [**Task 2**](#task-2)
    -   [2.1 Is my data tidy?](#21-is-my-data-tidy)
    -   [2.2 Untidying and tidying](#22-untidying-and-tidying)
    -   [2.3 Narrowing down research questions and prepping
        data](#23-narrowing-down-research-questions-and-prepping-data)

### Introduction

This is the R markdown document for milestone 2 of the mini data
analysis project for STAT 545A. This milestone of the project builds on
the work done in the first milestone (*MDA\_m1.Rmd*) to gain experience
working with *dplyr* and *tidyr* to handle datasets and address research
questions. This milestone focuses on exploring datasets for the purpose
of addressing a research question and the principle of tidy data and how
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
trees_neighbourhood <- trees %>% 
  group_by(neighbourhood_name) %>% 
  summarise(n = n())
print(trees_neighbourhood, n = 22) # prints all rows of tibble for markdown doc
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
    ## 11 MARPOLE                   6345
    ## 12 MOUNT PLEASANT            6291
    ## 13 OAKRIDGE                  4786
    ## 14 RENFREW-COLLINGWOOD      11380
    ## 15 RILEY PARK                6868
    ## 16 SHAUGHNESSY               6995
    ## 17 SOUTH CAMBIE              3342
    ## 18 STRATHCONA                2724
    ## 19 SUNSET                    8364
    ## 20 VICTORIA-FRASERVIEW       7777
    ## 21 WEST END                  3507
    ## 22 WEST POINT GREY           4935

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
             colour = neighbourhood_count), # colour with a continuous scale according to # of trees in the neighbourhood
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

First I need to subset the dataset to just cherry trees based on whether
they have “Cherry” in the common name.

``` r
cherry <- trees %>% 
     filter(grepl("CHERRY", common_name), # the grepl function evaluates whether the string of characters "CHERRY" appears in each common name and returns a TRUE/FALSE that can be filtered by
            !is.na(longitude)) # removes entries missing longitude (and also latitude) values needed to answer the question of spatial distribution

nrow(cherry)
```

    ## [1] 15989

There are 15 989 cherry trees in the dataset.

To summarise the cherry tree data I will create a categorical variable
based on the latitude and longitude of the trees to determine which
quadrant of the city they are in (NW, NE, SW, SE). For this exercise I
will set the center of Vancouver as (49.24388553259791,
-123.13716749250246).

``` r
cherry <- cherry %>% 
  mutate(quadrant = case_when((latitude > 49.24388553259791 & 
                               longitude < -123.13716749250246) ~ "NW",
                              (latitude > 49.24388553259791 & 
                               longitude > -123.13716749250246) ~ "NE",
                              (latitude < 49.24388553259791 & 
                               longitude < -123.13716749250246) ~ "SW",
                              (latitude < 49.24388553259791 & 
                               longitude > -123.13716749250246) ~ "SE")) %>% 
  mutate(quadrant = as.factor(quadrant))

summary(cherry$quadrant)
```

    ##   NE   NW   SE   SW 
    ## 5885 2451 5377 2276

Now let’s map the locations of all the cherry trees in Vancouver using
the leaflet package. This creates an interactive map that you can
navigate by zooming in and moving around. I’ve increased the
transparency of the points and made them quite small to better show the
hotspots of density across the city.

``` r
leaflet(cherry) %>% 
  addProviderTiles(providers$Esri.WorldStreetMap) %>% 
  addCircleMarkers(lng = cherry$longitude, lat = cherry$latitude,
                   radius = 0.1, opacity = 0.1) # opacity is the same as alpha transparency
```

![](MDA_m2_files/figure-gfm/cherry_map-1.png)<!-- -->

Note: this map is interactive when created in RStudio, but only a static
version is available in the markdown document.

#### 1.2.4 Tree area vs. size

To investigate whether the area a tree is planted in affects its size, I
first need to tidy up the plant area column:

``` r
class(trees$plant_area) # this is currently a character vector
```

    ## [1] "character"

``` r
unique(trees$plant_area) # and is a mix of numbers and letters
```

    ##  [1] "N"  "4"  "B"  "6"  "3"  "5"  "2"  NA   "10" "C"  "7"  "8"  "12" "25" "40"
    ## [16] "9"  "17" "1"  "24" "11" "20" "13" "15" "16" "G"  "18" "b"  "14" "30" "c" 
    ## [31] "L"  "P"  "50" "34" "60" "M"  "21" "35" "n"  "75" "45" "19" "0"  "g"  "22"
    ## [46] "y"  "27" "32" "26"

From the [dataset
information](opendata.vancouver.ca/explore/dataset/street-trees/information/)
we know that:

-   B = behind sidewalk
-   G = in tree grate
-   N = no sidewalk
-   C = cutout
-   a number indicates boulevard width in feet

I will assume that lowercase letters are typos and should be uppercase,
but I will remove entries with letters for plant area not explained in
the dataset documentation: y, L, P, M. *Credit for the code to replace
the numbers in the column with a character goes to TA Yulia Egorova who
answered this question for another student on Slack.*

``` r
trees_area <- trees %>% 
  filter(!is.na(plant_area), # filter out any trees missing plant area information
         !(plant_area %in% c("y", "P", "L", "M"))) %>% # filter out plant area entries not in documentation
  mutate(area_factor = gsub("[^0-9.-]", "", plant_area), # replaces characters with blanks
         area_factor = ifelse(area_factor == "", plant_area,"BL"), # replaces blanks from last step with their original value, and replaces everything else (= the numbers) with "BL" for boulevard
         area_factor = toupper(area_factor), # make all letters uppercase
         area_factor = as.factor(area_factor))
```

Let’s look at the number of observations across our newly wrangled
categories for the area planted:

``` r
summary(trees_area$area_factor)
```

    ##      B     BL      C      G      N 
    ##   9043 110739   8505   1979  14046

Now let’s plot tree diameter across these categories using boxplots to
see the means and quantiles of the data as well as adding the jittered
data points behind with high transparency to show the spread of the data
and give the viewer a rough, relative idea of the number of observations
in each category. As for question 1, I made the axis with diameter
logarithmic to better see the spread of data.

``` r
ggplot(trees_area, aes(x = area_factor, y = diameter)) + 
  geom_jitter(width = 0.4, alpha = 0.1, colour = "grey") +
  geom_boxplot(width = 0.6, alpha = 0.7) +
  scale_y_log10() + # makes the y axis logarithmic
  labs(x = "Area tree planted",
       y = "Tree diameter (inches)") +
  scale_x_discrete(limits = c("B", "BL", "C", "G", "N"),
                   labels = c("Behind sidewalk", "Boulevard", 
                              "Cutout", "Tree grate", "No sidewalk")) # renames the factor codes to something meaningful for a viewer
```

![](MDA_m2_files/figure-gfm/area_diameter_boxplot-1.png)<!-- -->

There doesn’t seem to be variation in tree diameter between the
different areas the trees are planted.

### 1.3 Progress on research questions

After completing the above summarizing and graphing tasks, I am closer
to answering all of the 4 research questions I proposed for the
*vancouver\_trees* dataset. Particularly through graphing summary
statistics and distributions of my variables of interest, diameter and
height, against some categorical variables of interest I’ve started to
get an idea of whether my hypotheses for my research questions will be
supported by the data. However, visualizing data to see if there is a
difference in tree diameter, for example, between trees with and without
barriers is not the same as modelling this relationship. This next step
of modeling in milestone 3 will reveal whether the trends (or lack there
of) visible in the plots I’ve created are statistically significant. The
only research question that seems to be yeilding any interesting results
is questions 1, where trees with root barriers appear to have smaller
diameters than those without.

## **Task 2**

### 2.1 Is my data tidy?

Tidy data:

-   each row is an observation
-   each column is a variable
-   each cell is a value

To determine if the *trees* dataset is tidy, let’s have another look at
it:

``` r
glimpse(trees)
```

    ## Rows: 146,484
    ## Columns: 20
    ## $ tree_id            <dbl> 149556, 149563, 149579, 149590, 149604, 149616, 149…
    ## $ civic_number       <dbl> 494, 450, 4994, 858, 5032, 585, 4909, 4925, 4969, 7…
    ## $ std_street         <chr> "W 58TH AV", "W 58TH AV", "WINDSOR ST", "E 39TH AV"…
    ## $ genus_name         <chr> "ULMUS", "ZELKOVA", "STYRAX", "FRAXINUS", "ACER", "…
    ## $ species_name       <chr> "AMERICANA", "SERRATA", "JAPONICA", "AMERICANA", "C…
    ## $ cultivar_name      <chr> "BRANDON", NA, NA, "AUTUMN APPLAUSE", NA, "CHANTICL…
    ## $ common_name        <chr> "BRANDON ELM", "JAPANESE ZELKOVA", "JAPANESE SNOWBE…
    ## $ assigned           <chr> "N", "N", "N", "Y", "N", "N", "N", "N", "N", "N", "…
    ## $ root_barrier       <chr> "N", "N", "N", "N", "N", "N", "N", "N", "N", "N", "…
    ## $ plant_area         <chr> "N", "N", "4", "4", "4", "B", "6", "6", "3", "3", "…
    ## $ on_street_block    <dbl> 400, 400, 4900, 800, 5000, 500, 4900, 4900, 4900, 7…
    ## $ on_street          <chr> "W 58TH AV", "W 58TH AV", "WINDSOR ST", "E 39TH AV"…
    ## $ neighbourhood_name <chr> "MARPOLE", "MARPOLE", "KENSINGTON-CEDAR COTTAGE", "…
    ## $ street_side_name   <chr> "EVEN", "EVEN", "EVEN", "EVEN", "EVEN", "ODD", "ODD…
    ## $ height_range_id    <dbl> 2, 4, 3, 4, 2, 2, 3, 3, 2, 2, 2, 5, 3, 2, 2, 2, 2, …
    ## $ diameter           <dbl> 10.00, 10.00, 4.00, 18.00, 9.00, 5.00, 15.00, 14.00…
    ## $ curb               <chr> "N", "N", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "…
    ## $ date_planted       <date> 1999-01-13, 1996-05-31, 1993-11-22, 1996-04-29, 19…
    ## $ longitude          <dbl> -123.1161, -123.1147, -123.0846, -123.0870, -123.08…
    ## $ latitude           <dbl> 49.21776, 49.21776, 49.23938, 49.23469, 49.23894, 4…

Each row in this dataset corresponds to an individual tree, so the first
criteria of having each row be an observation is met. As it relates to
my research questions, each column is a variable (although a handful are
not relevant to any of my questions) so that condition is met as well.
Lastly, each cell in this dataset does represent a value. Another
indicator of whether a dataset is tidy or not is whether you could put
the column names on the axes of a graph visualizing the data for your
research question. This test is met as well, as I used the dataset
(mostly) as is to create all the above plots.

So, yes - this dataset is tidy!

### 2.2 Untidying and tidying

Now to demonstrate what untidy data would look like, I’ll untidy the
*trees* dataset. I’m going to make the dataset longer by lumping all
street info into a single column with another column for the values of
those variables:

``` r
trees_untidy <- trees %>%
  mutate(on_street_block = as.character(on_street_block)) %>% # needs to be a chr vector to combine it with other chr columns in the pivot function
  pivot_longer(cols = c(std_street, on_street, 
                        on_street_block, street_side_name),
              names_to = "street_element",
              values_to = "street_info")
glimpse(trees_untidy)
```

    ## Rows: 585,936
    ## Columns: 18
    ## $ tree_id            <dbl> 149556, 149556, 149556, 149556, 149563, 149563, 149…
    ## $ civic_number       <dbl> 494, 494, 494, 494, 450, 450, 450, 450, 4994, 4994,…
    ## $ genus_name         <chr> "ULMUS", "ULMUS", "ULMUS", "ULMUS", "ZELKOVA", "ZEL…
    ## $ species_name       <chr> "AMERICANA", "AMERICANA", "AMERICANA", "AMERICANA",…
    ## $ cultivar_name      <chr> "BRANDON", "BRANDON", "BRANDON", "BRANDON", NA, NA,…
    ## $ common_name        <chr> "BRANDON ELM", "BRANDON ELM", "BRANDON ELM", "BRAND…
    ## $ assigned           <chr> "N", "N", "N", "N", "N", "N", "N", "N", "N", "N", "…
    ## $ root_barrier       <chr> "N", "N", "N", "N", "N", "N", "N", "N", "N", "N", "…
    ## $ plant_area         <chr> "N", "N", "N", "N", "N", "N", "N", "N", "4", "4", "…
    ## $ neighbourhood_name <chr> "MARPOLE", "MARPOLE", "MARPOLE", "MARPOLE", "MARPOL…
    ## $ height_range_id    <dbl> 2, 2, 2, 2, 4, 4, 4, 4, 3, 3, 3, 3, 4, 4, 4, 4, 2, …
    ## $ diameter           <dbl> 10, 10, 10, 10, 10, 10, 10, 10, 4, 4, 4, 4, 18, 18,…
    ## $ curb               <chr> "N", "N", "N", "N", "N", "N", "N", "N", "Y", "Y", "…
    ## $ date_planted       <date> 1999-01-13, 1999-01-13, 1999-01-13, 1999-01-13, 19…
    ## $ longitude          <dbl> -123.1161, -123.1161, -123.1161, -123.1161, -123.11…
    ## $ latitude           <dbl> 49.21776, 49.21776, 49.21776, 49.21776, 49.21776, 4…
    ## $ street_element     <chr> "std_street", "on_street", "on_street_block", "stre…
    ## $ street_info        <chr> "W 58TH AV", "W 58TH AV", "400", "EVEN", "W 58TH AV…

Now let’s tidy it back up

``` r
trees_tidy <- trees_untidy %>% 
  pivot_wider(id_cols = c(-street_element, -street_info),
               names_from = street_element,
               values_from = street_info)
glimpse(trees_tidy)
```

    ## Rows: 146,484
    ## Columns: 20
    ## $ tree_id            <dbl> 149556, 149563, 149579, 149590, 149604, 149616, 149…
    ## $ civic_number       <dbl> 494, 450, 4994, 858, 5032, 585, 4909, 4925, 4969, 7…
    ## $ genus_name         <chr> "ULMUS", "ZELKOVA", "STYRAX", "FRAXINUS", "ACER", "…
    ## $ species_name       <chr> "AMERICANA", "SERRATA", "JAPONICA", "AMERICANA", "C…
    ## $ cultivar_name      <chr> "BRANDON", NA, NA, "AUTUMN APPLAUSE", NA, "CHANTICL…
    ## $ common_name        <chr> "BRANDON ELM", "JAPANESE ZELKOVA", "JAPANESE SNOWBE…
    ## $ assigned           <chr> "N", "N", "N", "Y", "N", "N", "N", "N", "N", "N", "…
    ## $ root_barrier       <chr> "N", "N", "N", "N", "N", "N", "N", "N", "N", "N", "…
    ## $ plant_area         <chr> "N", "N", "4", "4", "4", "B", "6", "6", "3", "3", "…
    ## $ neighbourhood_name <chr> "MARPOLE", "MARPOLE", "KENSINGTON-CEDAR COTTAGE", "…
    ## $ height_range_id    <dbl> 2, 4, 3, 4, 2, 2, 3, 3, 2, 2, 2, 5, 3, 2, 2, 2, 2, …
    ## $ diameter           <dbl> 10.00, 10.00, 4.00, 18.00, 9.00, 5.00, 15.00, 14.00…
    ## $ curb               <chr> "N", "N", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "…
    ## $ date_planted       <date> 1999-01-13, 1996-05-31, 1993-11-22, 1996-04-29, 19…
    ## $ longitude          <dbl> -123.1161, -123.1147, -123.0846, -123.0870, -123.08…
    ## $ latitude           <dbl> 49.21776, 49.21776, 49.23938, 49.23469, 49.23894, 4…
    ## $ std_street         <chr> "W 58TH AV", "W 58TH AV", "WINDSOR ST", "E 39TH AV"…
    ## $ on_street          <chr> "W 58TH AV", "W 58TH AV", "WINDSOR ST", "E 39TH AV"…
    ## $ on_street_block    <chr> "400", "400", "4900", "800", "5000", "500", "4900",…
    ## $ street_side_name   <chr> "EVEN", "EVEN", "EVEN", "EVEN", "EVEN", "ODD", "ODD…

As we can see from the before and after, the tidy form of the data is
much more usable and easier to read and navigate in when viewing the raw
dataset.

### 2.3 Narrowing down research questions and prepping data

Now that I have explored the *trees* data further and gotten an idea of
which of my research questions seem to be giving interesting results,
I’m narrowing down tot he following 2 research questions to continue
with of Milestone 3:

1.  How does having a root barrier relate to tree size as measured by
    diameter and height?
    -   This question is the only one that seems to have a difference in
        a response variable (diameter) across my proposed predictor
        variable (root barrier). I also haven’t had a chance to explore
        the relationship between height and root barriers yet, which I
        would like to do.
2.  How does tree biomass (measured as a combination of number of trees
    and their size) vary across neighbourhoods?
    -   While my graphing exersise for this questions seemed to suggest
        that there is no difference in tree height across
        neightbourhoods, there is a difference in number of trees per
        neighbourhood, which still likely leads to differences in
        overall tree biomass between neighbourhoods. I am also
        interested in adding tree age to this question, both to get a
        chance to practice working with date type data and to see which
        neighbourhoods have the oldest and youngest trees, on average
        which i think could yeild interesting results. I will revise
        this question to the following:
    -   **How does tree age and tree biomass vary across Vancouver
        neighbourhoods?**

Given these 2 questions, I’ll create a version of the *trees* dataset
for Milestone 3 that is tidy and only keeps the relevant information:

``` r
glimpse(vancouver_trees) #starting with the raw vancouver_trees dataset because I've made some changes to the trees version I've been working with throughout this milestone
```

    ## Rows: 146,611
    ## Columns: 20
    ## $ tree_id            <dbl> 149556, 149563, 149579, 149590, 149604, 149616, 149…
    ## $ civic_number       <dbl> 494, 450, 4994, 858, 5032, 585, 4909, 4925, 4969, 7…
    ## $ std_street         <chr> "W 58TH AV", "W 58TH AV", "WINDSOR ST", "E 39TH AV"…
    ## $ genus_name         <chr> "ULMUS", "ZELKOVA", "STYRAX", "FRAXINUS", "ACER", "…
    ## $ species_name       <chr> "AMERICANA", "SERRATA", "JAPONICA", "AMERICANA", "C…
    ## $ cultivar_name      <chr> "BRANDON", NA, NA, "AUTUMN APPLAUSE", NA, "CHANTICL…
    ## $ common_name        <chr> "BRANDON ELM", "JAPANESE ZELKOVA", "JAPANESE SNOWBE…
    ## $ assigned           <chr> "N", "N", "N", "Y", "N", "N", "N", "N", "N", "N", "…
    ## $ root_barrier       <chr> "N", "N", "N", "N", "N", "N", "N", "N", "N", "N", "…
    ## $ plant_area         <chr> "N", "N", "4", "4", "4", "B", "6", "6", "3", "3", "…
    ## $ on_street_block    <dbl> 400, 400, 4900, 800, 5000, 500, 4900, 4900, 4900, 7…
    ## $ on_street          <chr> "W 58TH AV", "W 58TH AV", "WINDSOR ST", "E 39TH AV"…
    ## $ neighbourhood_name <chr> "MARPOLE", "MARPOLE", "KENSINGTON-CEDAR COTTAGE", "…
    ## $ street_side_name   <chr> "EVEN", "EVEN", "EVEN", "EVEN", "EVEN", "ODD", "ODD…
    ## $ height_range_id    <dbl> 2, 4, 3, 4, 2, 2, 3, 3, 2, 2, 2, 5, 3, 2, 2, 2, 2, …
    ## $ diameter           <dbl> 10.00, 10.00, 4.00, 18.00, 9.00, 5.00, 15.00, 14.00…
    ## $ curb               <chr> "N", "N", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "…
    ## $ date_planted       <date> 1999-01-13, 1996-05-31, 1993-11-22, 1996-04-29, 19…
    ## $ longitude          <dbl> -123.1161, -123.1147, -123.0846, -123.0870, -123.08…
    ## $ latitude           <dbl> 49.21776, 49.21776, 49.23938, 49.23469, 49.23894, 4…

``` r
trees_m3 <- vancouver_trees %>% 
  select(tree_id, genus_name, species_name, common_name, # keeping tree species info in case I want to look at species specific size differences or neighbourhood distributions
         root_barrier, neighbourhood_name, height_range_id,
         diameter, date_planted) %>% # keep only columns that might be relevant
  filter(diameter != 0 & diameter <= 75) %>% # remove outliers and incorrect values for diameter 
  mutate(log_diameter = log(diameter), # create column that's log of diameter for modelling (has more normal distribution, see milestone 1)
         neighbourhood_name = fct_rev(fct_infreq(neighbourhood_name))) %>% #reorder the factor by increasing frequency of neighbourhood
  group_by(neighbourhood_name) %>% 
  mutate(neighbourhood_count = n()) %>% # add column for neighbourhood count
  ungroup()

# I will also likely drop columns missing data for date planted for question 2, but will keep that data in for now as those observations can still be used when answering question 1

glimpse(trees_m3)
```

    ## Rows: 146,484
    ## Columns: 11
    ## $ tree_id             <dbl> 149556, 149563, 149579, 149590, 149604, 149616, 14…
    ## $ genus_name          <chr> "ULMUS", "ZELKOVA", "STYRAX", "FRAXINUS", "ACER", …
    ## $ species_name        <chr> "AMERICANA", "SERRATA", "JAPONICA", "AMERICANA", "…
    ## $ common_name         <chr> "BRANDON ELM", "JAPANESE ZELKOVA", "JAPANESE SNOWB…
    ## $ root_barrier        <chr> "N", "N", "N", "N", "N", "N", "N", "N", "N", "N", …
    ## $ neighbourhood_name  <fct> MARPOLE, MARPOLE, KENSINGTON-CEDAR COTTAGE, KENSIN…
    ## $ height_range_id     <dbl> 2, 4, 3, 4, 2, 2, 3, 3, 2, 2, 2, 5, 3, 2, 2, 2, 2,…
    ## $ diameter            <dbl> 10.00, 10.00, 4.00, 18.00, 9.00, 5.00, 15.00, 14.0…
    ## $ date_planted        <date> 1999-01-13, 1996-05-31, 1993-11-22, 1996-04-29, 1…
    ## $ log_diameter        <dbl> 2.302585, 2.302585, 1.386294, 2.890372, 2.197225, …
    ## $ neighbourhood_count <int> 6345, 6345, 11033, 11033, 11033, 6345, 11033, 1103…
