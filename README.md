# KatieTM_MDA
## Katie Tjaden-McClement
## Mini data analysis - STAT545A 2021W
## Updated October 28, 2021

This is the repository for the Mini data analysis (MDA) project for STAT 545A in the 2021 winter term. This project consists of 3 milestones, and has the overall goal of having students conduct an independent data analysis using tidy data principles and tools being taught concurrently in class.

### Milestone 1

* completed Oct 12, 2021
* explored potential datasets from the _datateachr_ package and chose the _vancouver_trees_ dataset for this mini data analysis
* performed 4 excercises to explore the dataset further using visualizations
* proposed 4 research questions to address with this dataset

### Milestone 2

* completed Oct 18, 2021
* summarized and visualized the _vancouver_trees_ dataset to investigate 4 different research questions
* evaluated whether the dataset was "tidy" given the research questions
* chose 2 of the 4 research questions to continue pursuing and created a tidy dataset that would allow these questions to be addressed

### Milestone 3

* completed Oct 28, 2021
* manipulated factor and date type data in the _vancouver_trees_ dataset
* fit a model to test whether mean diameter differs between trees with and without root barriers and produced tidy model output
* reading and writing csv and RDS files

### Files
Each milestone is contained within a folder in the github repo, and contains the following files:

* __MDA_mX.Rmd__: 
  * R markdown file
  * Contains the code for each milestone of the project 
  * Can be ran using the knit function in RStudio
* __MDA_mX.md__:
  * Markdown file
  * Product of knitting _MDA_mX.Rmd_
* __MDA_mX_files folder__:
  * Contains the _figure-gfm_ folder which holds all the png file of the plots produced by _MDA_mX.Rmd_ to supply to the _MDA_mX.md_ document

"mX" specifies the milestone, with X being 1, 2, or 3

* __Output folder__:
  * Contains output files from writing csv and RDS data files in Milestone 3
  * _q1_summary.csv_ is a csv file containing summary statistics for trees with and without root barriers
    * can be viewed in RStudio, excel, as plain text, etc.
  * _root_barrier_ttest.RDS_ is an R data file of the t test model that tests whether trees with and without root barriers differ in their mean diameter
    * must be opened with R to view (e.g. in RStudio using `readRDS()` as demonstrated in milestone 3)

:evergreen_tree:  :deciduous_tree:  :evergreen_tree:
