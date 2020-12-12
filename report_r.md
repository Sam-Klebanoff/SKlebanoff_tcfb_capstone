Analyzing data used to predict glaucoma diagnosis
================
Sam Klebanoff
12/10/2020

In this project, I will be analyzing the data from the paper entitled
“Development of machine learning models for diagnosis of glaucoma”
(Kim SJ et al. PLOS One, 2017). The goal of the paper was to train and
test different models of machine learning, using clinical data from eye
doctors on various tests which have been known to correlate with
development of glaucoma, as well as the information of which data came
from patients with glaucoma. The questions I’ll ask in my project mostly
center on the question of how robust the separation of glaucoma positive
and negative patients really is (based on the variables reported in the
paper).

## About the data

The data published along with this paper had seven distinct clinical
measurements associated with glaucoma: Age, ocular pressure, MD - mean
deviation (an overall value of the total amount of visual field loss),
PSD - pattern standard deviation (the amount of visual field loss for
patients with a localized visual field defect), GHT - glaucoma hemifield
test score (an indicator of the difference between superior and inferior
hemifield vision), cornea thickness, and a variable they call RNFL4.mean
(which represents the mean of three of the four common measurements of
the retinal nerve fiber layer). For each observation, the authors also
included whether the measurement came from the left eye (OS) or the
right eye (OD), as well as whether the patient had glaucoma.

Overall, this dataset was formatted very well, making it easy to use.
However, there are a couple of small changes the authors could have made
to improve the understandability of the raw data:

Firstly, they include a column showing whether each observation came
from a left eye or a right eye, however they never use this data in any
of their analyses in the paper. For this project, I am assuming that
this left/right eye data has no correlation with any of the other
markers, and I am therefore excluding it from my own analysis. It would
have been better if the authors had included either in the data or in
the text of the paper why they kept this measurement - if it truly
doesn’t correlated with any other measurements, why include it at all?

Secondly, the data published by the authors is already trimmed to
exclude a number of other clinical measurements - gender, VFI, as well
as the individual measurements of RNFL SUP, NAS, INF, and TMP (which
they combine into the variable RNFL4.mean). Although in the paper they
indicate that the seven variables in the published dataset are the most
important for building their model, the exclusion of the rest of the
data prevents any follow-up analysis to confirm the reproducibility of
this choice. Therefore, in this project, I have no choice but to assume
that the authors are correct to exclude these variables from the
mathematical analyses.

## Are the random subsets of data used for training/testing both truly representative?

In this paper, the authors randomly select a subset of their data to use
to train their machine learning algorithms, and then they use rest of
the data as a test set to confirm the accuracy of their model. It’s
important to know that the training data and the test data have similar
means and variances for all the recorded variables, otherwise the model
might not be truly representative, and the measured accuracy would be
artificially lowered.

``` r
# First, import the three .csv files for the whole dataset, the training subset, and the testing subset. I use 
# select(-RL) to remove the RL column from each table (which is the useless left vs right eye data)
data_whole <- read_csv("data/ds_whole.csv") %>%
  select(-RL)
```

    ## 
    ## -- Column specification --------------------------------------------------------
    ## cols(
    ##   RL = col_character(),
    ##   glaucoma = col_double(),
    ##   age = col_double(),
    ##   ocular_pressure = col_double(),
    ##   MD = col_double(),
    ##   PSD = col_double(),
    ##   GHT = col_double(),
    ##   cornea_thickness = col_double(),
    ##   RNFL4.mean = col_double()
    ## )

``` r
data_train <- read_csv("data/ds_train.csv") %>%
  select(-RL)
```

    ## 
    ## -- Column specification --------------------------------------------------------
    ## cols(
    ##   RL = col_character(),
    ##   glaucoma = col_double(),
    ##   age = col_double(),
    ##   ocular_pressure = col_double(),
    ##   MD = col_double(),
    ##   PSD = col_double(),
    ##   GHT = col_double(),
    ##   cornea_thickness = col_double(),
    ##   RNFL4.mean = col_double()
    ## )

``` r
data_test <- read_csv("data/ds_test.csv") %>%
  select(-RL)
```

    ## 
    ## -- Column specification --------------------------------------------------------
    ## cols(
    ##   RL = col_character(),
    ##   glaucoma = col_double(),
    ##   age = col_double(),
    ##   ocular_pressure = col_double(),
    ##   MD = col_double(),
    ##   PSD = col_double(),
    ##   GHT = col_double(),
    ##   cornea_thickness = col_double(),
    ##   RNFL4.mean = col_double()
    ## )

``` r
# Filter out the glaucoma positive and negative rows from each data frame, to be able to compare those individually
# between the training, test, and whole conditions.
# I use 'g' for glaucoma positive (where 'glaucoma'  =  1), 'h' for healthy (where 'glaucoma' = 0)
data_whole_g <- data_whole %>% filter(glaucoma == 1)
data_train_g <- data_train %>% filter(glaucoma == 1)
data_test_g <- data_test %>% filter(glaucoma == 1)
data_whole_h <- data_whole %>% filter(glaucoma == 0)
data_train_h <- data_train %>% filter(glaucoma == 0)
data_test_h <- data_test %>% filter(glaucoma == 0)

# Create two blank summary data frames in which to store the mean and standard deviation of each variable from each
# dataset. Make separate summaries for the glaucoma positive and healthy patients
summary_df_g <- data.frame(dataset=character(), mean_age=double(), std_age=double(), mean_OP=double(), 
                           std_OP=double(), mean_MD=double(), std_MD=double(), mean_PSD=double(), 
                           std_PSD=double(), mean_GHT=double(), std_GHT=double(), mean_CT=double(), 
                           std_CT=double(), mean_RNFL=double(), std_RNFL=double())
summary_df_h <- data.frame(dataset=character(), mean_age=double(), std_age=double(), mean_OP=double(), 
                           std_OP=double(), mean_MD=double(), std_MD=double(), mean_PSD=double(), 
                           std_PSD=double(), mean_GHT=double(), std_GHT=double(), mean_CT=double(), 
                           std_CT=double(), mean_RNFL=double(), std_RNFL=double())

# Create a list of the data frames, and a list of the corresponding set names, to use over a for loop
df_g_list <- list(data_test_g, data_train_g, data_whole_g)
df_h_list <- list(data_test_h, data_train_h, data_whole_h)
name_list <- c("Test", "Training", "Whole")

#Populate the summary data frames
for (i in 1:3) {
  summary_df_g[i,1] = name_list[i]
  summary_df_g[i,2] = mean(df_g_list[[i]]$age)
  summary_df_g[i,3] = sd(df_g_list[[i]]$age)
  summary_df_g[i,4] = mean(df_g_list[[i]]$ocular_pressure)
  summary_df_g[i,5] = sd(df_g_list[[i]]$ocular_pressure)
  summary_df_g[i,6] = mean(df_g_list[[i]]$MD)
  summary_df_g[i,7] = sd(df_g_list[[i]]$MD)
  summary_df_g[i,8] = mean(df_g_list[[i]]$PSD)
  summary_df_g[i,9] = sd(df_g_list[[i]]$PSD)
  summary_df_g[i,10] = mean(df_g_list[[i]]$GHT)
  summary_df_g[i,11] = sd(df_g_list[[i]]$GHT)
  summary_df_g[i,12] = mean(df_g_list[[i]]$cornea_thickness)
  summary_df_g[i,13] = sd(df_g_list[[i]]$cornea_thickness)
  summary_df_g[i,14] = mean(df_g_list[[i]]$RNFL4.mean)
  summary_df_g[i,15] = sd(df_g_list[[i]]$RNFL4.mean)

  summary_df_h[i,1] = name_list[i]
  summary_df_h[i,2] = mean(df_h_list[[i]]$age)
  summary_df_h[i,3] = sd(df_h_list[[i]]$age)
  summary_df_h[i,4] = mean(df_h_list[[i]]$ocular_pressure)
  summary_df_h[i,5] = sd(df_h_list[[i]]$ocular_pressure)
  summary_df_h[i,6] = mean(df_h_list[[i]]$MD)
  summary_df_h[i,7] = sd(df_h_list[[i]]$MD)
  summary_df_h[i,8] = mean(df_h_list[[i]]$PSD)
  summary_df_h[i,9] = sd(df_h_list[[i]]$PSD)
  summary_df_h[i,10] = mean(df_h_list[[i]]$GHT)
  summary_df_h[i,11] = sd(df_h_list[[i]]$GHT)
  summary_df_h[i,12] = mean(df_h_list[[i]]$cornea_thickness)
  summary_df_h[i,13] = sd(df_h_list[[i]]$cornea_thickness)
  summary_df_h[i,14] = mean(df_h_list[[i]]$RNFL4.mean)
  summary_df_h[i,15] = sd(df_h_list[[i]]$RNFL4.mean)
} 

print(summary_df_g)
```

    ##    dataset mean_age  std_age  mean_OP    std_OP   mean_MD    std_MD mean_PSD
    ## 1     Test 63.71667 13.33416 24.41667 10.120112 -14.66317  9.259565 8.128000
    ## 2 Training 59.64557 13.40254 24.02110  9.386001 -12.81949 11.477491 7.700506
    ## 3    Whole 60.46801 13.46617 24.10101  9.522550 -13.19195 11.075649 7.786869
    ##    std_PSD mean_GHT   std_GHT  mean_CT   std_CT mean_RNFL std_RNFL
    ## 1 3.773875 1.866667 0.4681977 533.5000 33.29707  65.56111 19.53851
    ## 2 4.241327 1.751055 0.6321615 535.9620 31.64090  67.55977 20.08262
    ## 3 4.148593 1.774411 0.6037202 535.4646 31.94029  67.15600 19.95741

``` r
print(summary_df_h)
```

    ##    dataset mean_age  std_age  mean_OP   std_OP   mean_MD   std_MD mean_PSD
    ## 1     Test 52.20000 15.26215 16.95000 3.529727 -2.125250 3.813241 2.150500
    ## 2 Training 51.35185 16.87052 16.20370 3.356904 -1.969198 2.315397 2.171667
    ## 3    Whole 51.51980 16.53135 16.35149 3.395962 -2.000099 2.668224 2.167475
    ##     std_PSD  mean_GHT   std_GHT  mean_CT   std_CT mean_RNFL std_RNFL
    ## 1 1.0027193 0.3750000 0.7403222 552.3500 35.98401  103.6417 13.73879
    ## 2 0.8578358 0.5000000 0.7821772 546.2160 34.45687  105.5700 14.03002
    ## 3 0.8857743 0.4752475 0.7738777 547.4307 34.75984  105.1881 13.96018

For this question I imported the test, training, and whole datasets into
separate data frames. I split the data from each table into two separate
data frames by ‘glaucoma positive’ vs ‘healthy’, with the idea that
difference between the test, training, and whole datasets might masked
by the differences between glaucoma positive and healthy patients. I
then created summary data frames which contain the mean and standard
deviation of each variable, for each dataset. This should allow easy
visual comparison.

``` r
# I chose to use grid and gridExtra in this visualization, so that I could take advantage of grid.arrange
# This allowed me to make separate graphs for each variable, grouped by +/- glaucoma, showing the relationship
# between test, training, and whole for each variable. I then tiled all the graphs together using grid/gridExtra
library(gridExtra)
```

    ## 
    ## Attaching package: 'gridExtra'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     combine

``` r
library(grid)

g_age_plot <- ggplot(summary_df_g, aes(x=dataset, y=mean_age)) + geom_point() + 
              geom_errorbar(aes(ymin = mean_age-std_age, ymax = mean_age+std_age), width = 0.3) + 
              theme_classic() + labs(title = "Age", y = NULL, x = NULL) + 
              theme(plot.title = element_text(hjust = 0.5, vjust = 2))
g_OP_plot <- ggplot(summary_df_g, aes(x=dataset, y=mean_OP)) + geom_point() + 
             geom_errorbar(aes(ymin = mean_OP-std_OP, ymax = mean_OP+std_OP), width = 0.3) + 
             theme_classic() + labs(title = "Ocular Pressure", y = NULL, x = NULL) + 
             theme(plot.title = element_text(hjust = 0.5, vjust = 2))
g_MD_plot <- ggplot(summary_df_g, aes(x=dataset, y=mean_MD)) + geom_point() + 
             geom_errorbar(aes(ymin = mean_MD-std_MD, ymax = mean_MD+std_MD), width = 0.3) + 
             theme_classic() + labs(title = "MD", y = NULL, x = NULL) + 
             theme(plot.title = element_text(hjust = 0.5, vjust = 2))
g_PSD_plot <- ggplot(summary_df_g, aes(x=dataset, y=mean_PSD)) + geom_point() + 
              geom_errorbar(aes(ymin = mean_PSD-std_PSD, ymax = mean_PSD+std_PSD), width = 0.3) + 
              theme_classic() + labs(title = "PSD", y = NULL, x = NULL) + 
              theme(plot.title = element_text(hjust = 0.5, vjust = 2))
g_GHT_plot <- ggplot(summary_df_g, aes(x=dataset, y=mean_GHT)) + geom_point() + 
              geom_errorbar(aes(ymin = mean_GHT-std_GHT, ymax = mean_GHT+std_GHT), width = 0.3) + 
              theme_classic() + labs(title = "GHT", y = NULL, x = NULL) + 
              theme(plot.title = element_text(hjust = 0.5, vjust = 2))
g_CT_plot <- ggplot(summary_df_g, aes(x=dataset, y=mean_CT)) + geom_point() + 
             geom_errorbar(aes(ymin = mean_CT-std_CT, ymax = mean_CT+std_CT), width = 0.3) + 
             theme_classic() + labs(title = "Cornea Thickness", y = NULL, x = NULL) + 
             theme(plot.title = element_text(hjust = 0.5, vjust = 2))
g_RNFL_plot <- ggplot(summary_df_g, aes(x=dataset, y=mean_RNFL)) + geom_point() + 
               geom_errorbar(aes(ymin = mean_RNFL-std_RNFL, ymax = mean_RNFL+std_RNFL), width = 0.3) + 
               theme_classic() + labs(title = "RNFL4.mean", y = NULL, x = NULL) + 
               theme(plot.title = element_text(hjust = 0.5, vjust = 2))

h_age_plot <- ggplot(summary_df_h, aes(x=dataset, y=mean_age)) + geom_point() + 
              geom_errorbar(aes(ymin = mean_age-std_age, ymax = mean_age+std_age), width = 0.3) + 
              theme_classic() + labs(title = "Age", y = NULL, x = NULL) + 
              theme(plot.title = element_text(hjust = 0.5, vjust = 2))
h_OP_plot <- ggplot(summary_df_h, aes(x=dataset, y=mean_OP)) + geom_point() + 
             geom_errorbar(aes(ymin = mean_OP-std_OP, ymax = mean_OP+std_OP), width = 0.3) + 
             theme_classic() + labs(title = "Ocular Pressure", y = NULL, x = NULL) + 
             theme(plot.title = element_text(hjust = 0.5, vjust = 2))
h_MD_plot <- ggplot(summary_df_h, aes(x=dataset, y=mean_MD)) + geom_point() + 
             geom_errorbar(aes(ymin = mean_MD-std_MD, ymax = mean_MD+std_MD), width = 0.3) + 
             theme_classic() + labs(title = "MD", y = NULL, x = NULL) + 
             theme(plot.title = element_text(hjust = 0.5, vjust = 2))
h_PSD_plot <- ggplot(summary_df_h, aes(x=dataset, y=mean_PSD)) + geom_point() + 
              geom_errorbar(aes(ymin = mean_PSD-std_PSD, ymax = mean_PSD+std_PSD), width = 0.3) + 
              theme_classic() + labs(title = "PSD", y = NULL, x = NULL) + 
              theme(plot.title = element_text(hjust = 0.5, vjust = 2))
h_GHT_plot <- ggplot(summary_df_h, aes(x=dataset, y=mean_GHT)) + geom_point() + 
              geom_errorbar(aes(ymin = mean_GHT-std_GHT, ymax = mean_GHT+std_GHT), width = 0.3) + 
              theme_classic() + labs(title = "GHT", y = NULL, x = NULL) + 
              theme(plot.title = element_text(hjust = 0.5, vjust = 2))
h_CT_plot <- ggplot(summary_df_h, aes(x=dataset, y=mean_CT)) + geom_point() + 
             geom_errorbar(aes(ymin = mean_CT-std_CT, ymax = mean_CT+std_CT), width = 0.3) + 
             theme_classic() + labs(title = "Cornea Thickness", y = NULL, x = NULL) + 
             theme(plot.title = element_text(hjust = 0.5, vjust = 2))
h_RNFL_plot <- ggplot(summary_df_h, aes(x=dataset, y=mean_RNFL)) + geom_point() + 
               geom_errorbar(aes(ymin = mean_RNFL-std_RNFL, ymax = mean_RNFL+std_RNFL), width = 0.3) + 
               theme_classic() + labs(title = "RNFL4.mean", y = NULL, x = NULL) + 
               theme(plot.title = element_text(hjust = 0.5, vjust = 2))

# Create the titles and the layout that will be added into grid.arrange along with the graphs
g_title = textGrob("Glaucoma-positive Patient Data",gp=gpar(fontsize=20,font=2),vjust=5)
h_title = textGrob("Healthy Patient Data",gp=gpar(fontsize=20,font=2),vjust=5)
layout <- rbind(c(1,1,1,1,1,1,1),
                c(2,3,4,5,6,7,8),
                c(9,9,9,9,9,9,9),
                c(10,11,12,13,14,15,16))

# Put it all together
grid.arrange(g_title, g_age_plot, g_OP_plot, g_MD_plot, g_PSD_plot, g_GHT_plot, g_CT_plot, g_RNFL_plot,
             h_title, h_age_plot, h_OP_plot, h_MD_plot, h_PSD_plot, h_GHT_plot, h_CT_plot, h_RNFL_plot,
             layout_matrix=layout)
```

![](report_r_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

As can be seen in these graphs, there are no significant differences
between the test, training, and whole datasets for any of the variables,
in either the glaucoma-positive or healthy patient data. However, there
are many slight (but not statistically significant) differences to be
seen. For nearly every variable, the mean of the training data is very
close to the mean of the whole dataset, however in many cases the mean
of the test data is noticeably different from the other two. It is
therefore possible that re-randomizing which data was chosen for the
training/test sets could result in the machine learning algorithm
scoring a higher accuracy (if the test data were more similar to the
training data, then the resulting algorithm would be more likely to
correct interpret the test data). However, that would not necessarily
mean that the resulting algorithm would be more accurate when given new
sets of patient data, so it is perhaps just as well that the data being
used to test the model is at least slightly different from the data used
to generate it.