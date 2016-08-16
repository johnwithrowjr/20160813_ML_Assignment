---
title: "Predicting Manner of Exercise"
author: "John Withrow, PhD"
date: "August 13, 2016"
output: html_document
---



## Executive Summary

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. 

In this project, we use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website  http://groupware.les.inf.puc-rio.br/har. Grateful acknowledgement is extended to the administrators of this website for their generosity in allowing us to use these data.

## Analytical Context
### Session Information

```r
sessionInfo()
```

```
## R version 3.2.5 (2016-04-14)
## Platform: x86_64-w64-mingw32/x64 (64-bit)
## Running under: Windows >= 8 x64 (build 9200)
## 
## locale:
## [1] LC_COLLATE=English_United States.1252 
## [2] LC_CTYPE=English_United States.1252   
## [3] LC_MONETARY=English_United States.1252
## [4] LC_NUMERIC=C                          
## [5] LC_TIME=English_United States.1252    
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] randomForest_4.6-12 caret_6.0-70        ggplot2_2.1.0      
## [4] lattice_0.20-33    
## 
## loaded via a namespace (and not attached):
##  [1] Rcpp_0.12.4        formatR_1.4        compiler_3.2.5    
##  [4] nloptr_1.0.4       plyr_1.8.3         class_7.3-14      
##  [7] bitops_1.0-6       iterators_1.0.8    tools_3.2.5       
## [10] testthat_1.0.0     lme4_1.1-12        digest_0.6.9      
## [13] evaluate_0.9       memoise_1.0.0      nlme_3.1-125      
## [16] gtable_0.2.0       mgcv_1.8-12        Matrix_1.2-4      
## [19] foreach_1.4.3      swirl_2.4.1        yaml_2.1.13       
## [22] parallel_3.2.5     SparseM_1.7        e1071_1.6-7       
## [25] stringr_1.0.0      httr_1.1.0         knitr_1.13        
## [28] MatrixModels_0.4-1 stats4_3.2.5       grid_3.2.5        
## [31] nnet_7.3-12        R6_2.1.2           rmarkdown_0.9.6   
## [34] minqa_1.2.4        reshape2_1.4.1     car_2.1-2         
## [37] magrittr_1.5       htmltools_0.3.5    scales_0.4.0      
## [40] codetools_0.2-14   MASS_7.3-45        splines_3.2.5     
## [43] pbkrtest_0.4-6     colorspace_1.2-6   quantreg_5.26     
## [46] stringi_1.0-1      RCurl_1.95-4.8     munsell_0.4.3     
## [49] markdown_0.7.7     crayon_1.3.1
```
### Setting of Random Number Seed

```r
set.seed(19690828)
```
### Set Working Directory

```r
setwd("C:\\Users\\John\\Desktop\\Projects\\20160813_ML_Assignment")
```

***

## Analytical Process
### Data Acquisition
Training and testing data are already divided into two files, downloadable from two different sources:

```r
download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv","c:\\Users\\John\\Desktop\\Projects\\20160813_ML_Assignment\\pml-training.csv")
download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv","c:\\Users\\John\\Desktop\\Projects\\20160813_ML_Assignment\\pml-testing.csv")
```

### Manual Pre-Processing
In both downloaded files, the title of the first column is missing.  It was added as "num" for both files, as it appears in both to be a simple enumeration column.

### Reading and Pre-processing of Data
The following pre-processing of data took place:
1. All columns after the seventh and before the last were set to type numeric
2. It was found that any column that had missing data had at most a few hundred records without missing data (out of over 13,000 total records).  These fields were removed from the study, reducing the total number of columns from 154 to 60.

```r
training <- read.csv("c:\\Users\\John\\Desktop\\Projects\\20160813_ML_Assignment\\pml-training.csv",header=TRUE)
testing <- read.csv("C:\\Users\\John\\Desktop\\Projects\\20160813_ML_Assignment\\pml-testing.csv",header=TRUE)
for (i in seq(7,159))
{
  if (class(training[,i])=="factor")
  {
    training[,i] <- as.numeric(levels(training[,i]))[training[,i]]
  }
}
```

```
## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion

## Warning: NAs introduced by coercion
```

```r
training <- training[,-c(14,17,89,92,127,130)]
training <- training[,-c(seq(12,34),seq(48,57),seq(67,81),seq(85,97),seq(99,108),seq(121,133),seq(135,144))]
testing <- testing[,-c(14,17,89,92,127,130)]
testing <- testing[,-c(seq(12,34),seq(48,57),seq(67,81),seq(85,97),seq(99,108),seq(121,133),seq(135,144))]
str(training)
```

```
## 'data.frame':	19622 obs. of  60 variables:
##  $ X                   : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ user_name           : Factor w/ 6 levels "adelmo","carlitos",..: 2 2 2 2 2 2 2 2 2 2 ...
##  $ raw_timestamp_part_1: int  1323084231 1323084231 1323084231 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 ...
##  $ raw_timestamp_part_2: int  788290 808298 820366 120339 196328 304277 368296 440390 484323 484434 ...
##  $ cvtd_timestamp      : Factor w/ 20 levels "02/12/2011 13:32",..: 9 9 9 9 9 9 9 9 9 9 ...
##  $ new_window          : Factor w/ 2 levels "no","yes": 1 1 1 1 1 1 1 1 1 1 ...
##  $ num_window          : int  11 11 11 12 12 12 12 12 12 12 ...
##  $ roll_belt           : num  1.41 1.41 1.42 1.48 1.48 1.45 1.42 1.42 1.43 1.45 ...
##  $ pitch_belt          : num  8.07 8.07 8.07 8.05 8.07 8.06 8.09 8.13 8.16 8.17 ...
##  $ yaw_belt            : num  -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 ...
##  $ total_accel_belt    : int  3 3 3 3 3 3 3 3 3 3 ...
##  $ gyros_belt_x        : num  0 0.02 0 0.02 0.02 0.02 0.02 0.02 0.02 0.03 ...
##  $ gyros_belt_y        : num  0 0 0 0 0.02 0 0 0 0 0 ...
##  $ gyros_belt_z        : num  -0.02 -0.02 -0.02 -0.03 -0.02 -0.02 -0.02 -0.02 -0.02 0 ...
##  $ accel_belt_x        : int  -21 -22 -20 -22 -21 -21 -22 -22 -20 -21 ...
##  $ accel_belt_y        : int  4 4 5 3 2 4 3 4 2 4 ...
##  $ accel_belt_z        : int  22 22 23 21 24 21 21 21 24 22 ...
##  $ magnet_belt_x       : int  -3 -7 -2 -6 -6 0 -4 -2 1 -3 ...
##  $ magnet_belt_y       : int  599 608 600 604 600 603 599 603 602 609 ...
##  $ magnet_belt_z       : int  -313 -311 -305 -310 -302 -312 -311 -313 -312 -308 ...
##  $ roll_arm            : num  -128 -128 -128 -128 -128 -128 -128 -128 -128 -128 ...
##  $ pitch_arm           : num  22.5 22.5 22.5 22.1 22.1 22 21.9 21.8 21.7 21.6 ...
##  $ yaw_arm             : num  -161 -161 -161 -161 -161 -161 -161 -161 -161 -161 ...
##  $ total_accel_arm     : int  34 34 34 34 34 34 34 34 34 34 ...
##  $ gyros_arm_x         : num  0 0.02 0.02 0.02 0 0.02 0 0.02 0.02 0.02 ...
##  $ gyros_arm_y         : num  0 -0.02 -0.02 -0.03 -0.03 -0.03 -0.03 -0.02 -0.03 -0.03 ...
##  $ gyros_arm_z         : num  -0.02 -0.02 -0.02 0.02 0 0 0 0 -0.02 -0.02 ...
##  $ accel_arm_x         : int  -288 -290 -289 -289 -289 -289 -289 -289 -288 -288 ...
##  $ accel_arm_y         : int  109 110 110 111 111 111 111 111 109 110 ...
##  $ accel_arm_z         : int  -123 -125 -126 -123 -123 -122 -125 -124 -122 -124 ...
##  $ magnet_arm_x        : int  -368 -369 -368 -372 -374 -369 -373 -372 -369 -376 ...
##  $ magnet_arm_y        : int  337 337 344 344 337 342 336 338 341 334 ...
##  $ magnet_arm_z        : int  516 513 513 512 506 513 509 510 518 516 ...
##  $ roll_dumbbell       : num  13.1 13.1 12.9 13.4 13.4 ...
##  $ pitch_dumbbell      : num  -70.5 -70.6 -70.3 -70.4 -70.4 ...
##  $ yaw_dumbbell        : num  -84.9 -84.7 -85.1 -84.9 -84.9 ...
##  $ total_accel_dumbbell: int  37 37 37 37 37 37 37 37 37 37 ...
##  $ gyros_dumbbell_x    : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ gyros_dumbbell_y    : num  -0.02 -0.02 -0.02 -0.02 -0.02 -0.02 -0.02 -0.02 -0.02 -0.02 ...
##  $ gyros_dumbbell_z    : num  0 0 0 -0.02 0 0 0 0 0 0 ...
##  $ accel_dumbbell_x    : int  -234 -233 -232 -232 -233 -234 -232 -234 -232 -235 ...
##  $ accel_dumbbell_y    : int  47 47 46 48 48 48 47 46 47 48 ...
##  $ accel_dumbbell_z    : int  -271 -269 -270 -269 -270 -269 -270 -272 -269 -270 ...
##  $ magnet_dumbbell_x   : int  -559 -555 -561 -552 -554 -558 -551 -555 -549 -558 ...
##  $ magnet_dumbbell_y   : int  293 296 298 303 292 294 295 300 292 291 ...
##  $ magnet_dumbbell_z   : num  -65 -64 -63 -60 -68 -66 -70 -74 -65 -69 ...
##  $ roll_forearm        : num  28.4 28.3 28.3 28.1 28 27.9 27.9 27.8 27.7 27.7 ...
##  $ pitch_forearm       : num  -63.9 -63.9 -63.9 -63.9 -63.9 -63.9 -63.9 -63.8 -63.8 -63.8 ...
##  $ yaw_forearm         : num  -153 -153 -152 -152 -152 -152 -152 -152 -152 -152 ...
##  $ total_accel_forearm : int  36 36 36 36 36 36 36 36 36 36 ...
##  $ gyros_forearm_x     : num  0.03 0.02 0.03 0.02 0.02 0.02 0.02 0.02 0.03 0.02 ...
##  $ gyros_forearm_y     : num  0 0 -0.02 -0.02 0 -0.02 0 -0.02 0 0 ...
##  $ gyros_forearm_z     : num  -0.02 -0.02 0 0 -0.02 -0.03 -0.02 0 -0.02 -0.02 ...
##  $ accel_forearm_x     : int  192 192 196 189 189 193 195 193 193 190 ...
##  $ accel_forearm_y     : int  203 203 204 206 206 203 205 205 204 205 ...
##  $ accel_forearm_z     : int  -215 -216 -213 -214 -214 -215 -215 -213 -214 -215 ...
##  $ magnet_forearm_x    : int  -17 -18 -18 -16 -17 -9 -18 -9 -16 -22 ...
##  $ magnet_forearm_y    : num  654 661 658 658 655 660 659 660 653 656 ...
##  $ magnet_forearm_z    : num  476 473 469 469 473 478 470 474 476 473 ...
##  $ classe              : Factor w/ 5 levels "A","B","C","D",..: 1 1 1 1 1 1 1 1 1 1 ...
```

```r
str(testing)
```

```
## 'data.frame':	20 obs. of  60 variables:
##  $ X                   : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ user_name           : Factor w/ 6 levels "adelmo","carlitos",..: 6 5 5 1 4 5 5 5 2 3 ...
##  $ raw_timestamp_part_1: int  1323095002 1322673067 1322673075 1322832789 1322489635 1322673149 1322673128 1322673076 1323084240 1322837822 ...
##  $ raw_timestamp_part_2: int  868349 778725 342967 560311 814776 510661 766645 54671 916313 384285 ...
##  $ cvtd_timestamp      : Factor w/ 11 levels "02/12/2011 13:33",..: 5 10 10 1 6 11 11 10 3 2 ...
##  $ new_window          : Factor w/ 1 level "no": 1 1 1 1 1 1 1 1 1 1 ...
##  $ num_window          : int  74 431 439 194 235 504 485 440 323 664 ...
##  $ roll_belt           : num  123 1.02 0.87 125 1.35 -5.92 1.2 0.43 0.93 114 ...
##  $ pitch_belt          : num  27 4.87 1.82 -41.6 3.33 1.59 4.44 4.15 6.72 22.4 ...
##  $ yaw_belt            : num  -4.75 -88.9 -88.5 162 -88.6 -87.7 -87.3 -88.5 -93.7 -13.1 ...
##  $ total_accel_belt    : int  20 4 5 17 3 4 4 4 4 18 ...
##  $ gyros_belt_x        : num  -0.5 -0.06 0.05 0.11 0.03 0.1 -0.06 -0.18 0.1 0.14 ...
##  $ gyros_belt_y        : num  -0.02 -0.02 0.02 0.11 0.02 0.05 0 -0.02 0 0.11 ...
##  $ gyros_belt_z        : num  -0.46 -0.07 0.03 -0.16 0 -0.13 0 -0.03 -0.02 -0.16 ...
##  $ accel_belt_x        : int  -38 -13 1 46 -8 -11 -14 -10 -15 -25 ...
##  $ accel_belt_y        : int  69 11 -1 45 4 -16 2 -2 1 63 ...
##  $ accel_belt_z        : int  -179 39 49 -156 27 38 35 42 32 -158 ...
##  $ magnet_belt_x       : int  -13 43 29 169 33 31 50 39 -6 10 ...
##  $ magnet_belt_y       : int  581 636 631 608 566 638 622 635 600 601 ...
##  $ magnet_belt_z       : int  -382 -309 -312 -304 -418 -291 -315 -305 -302 -330 ...
##  $ roll_arm            : num  40.7 0 0 -109 76.1 0 0 0 -137 -82.4 ...
##  $ pitch_arm           : num  -27.8 0 0 55 2.76 0 0 0 11.2 -63.8 ...
##  $ yaw_arm             : num  178 0 0 -142 102 0 0 0 -167 -75.3 ...
##  $ total_accel_arm     : int  10 38 44 25 29 14 15 22 34 32 ...
##  $ gyros_arm_x         : num  -1.65 -1.17 2.1 0.22 -1.96 0.02 2.36 -3.71 0.03 0.26 ...
##  $ gyros_arm_y         : num  0.48 0.85 -1.36 -0.51 0.79 0.05 -1.01 1.85 -0.02 -0.5 ...
##  $ gyros_arm_z         : num  -0.18 -0.43 1.13 0.92 -0.54 -0.07 0.89 -0.69 -0.02 0.79 ...
##  $ accel_arm_x         : int  16 -290 -341 -238 -197 -26 99 -98 -287 -301 ...
##  $ accel_arm_y         : int  38 215 245 -57 200 130 79 175 111 -42 ...
##  $ accel_arm_z         : int  93 -90 -87 6 -30 -19 -67 -78 -122 -80 ...
##  $ magnet_arm_x        : int  -326 -325 -264 -173 -170 396 702 535 -367 -420 ...
##  $ magnet_arm_y        : int  385 447 474 257 275 176 15 215 335 294 ...
##  $ magnet_arm_z        : int  481 434 413 633 617 516 217 385 520 493 ...
##  $ roll_dumbbell       : num  -17.7 54.5 57.1 43.1 -101.4 ...
##  $ pitch_dumbbell      : num  25 -53.7 -51.4 -30 -53.4 ...
##  $ yaw_dumbbell        : num  126.2 -75.5 -75.2 -103.3 -14.2 ...
##  $ total_accel_dumbbell: int  9 31 29 18 4 29 29 29 3 2 ...
##  $ gyros_dumbbell_x    : num  0.64 0.34 0.39 0.1 0.29 -0.59 0.34 0.37 0.03 0.42 ...
##  $ gyros_dumbbell_y    : num  0.06 0.05 0.14 -0.02 -0.47 0.8 0.16 0.14 -0.21 0.51 ...
##  $ gyros_dumbbell_z    : num  -0.61 -0.71 -0.34 0.05 -0.46 1.1 -0.23 -0.39 -0.21 -0.03 ...
##  $ accel_dumbbell_x    : int  21 -153 -141 -51 -18 -138 -145 -140 0 -7 ...
##  $ accel_dumbbell_y    : int  -15 155 155 72 -30 166 150 159 25 -20 ...
##  $ accel_dumbbell_z    : int  81 -205 -196 -148 -5 -186 -190 -191 9 7 ...
##  $ magnet_dumbbell_x   : int  523 -502 -506 -576 -424 -543 -484 -515 -519 -531 ...
##  $ magnet_dumbbell_y   : int  -528 388 349 238 252 262 354 350 348 321 ...
##  $ magnet_dumbbell_z   : int  -56 -36 41 53 312 96 97 53 -32 -164 ...
##  $ roll_forearm        : num  141 109 131 0 -176 150 155 -161 15.5 13.2 ...
##  $ pitch_forearm       : num  49.3 -17.6 -32.6 0 -2.16 1.46 34.5 43.6 -63.5 19.4 ...
##  $ yaw_forearm         : num  156 106 93 0 -47.9 89.7 152 -89.5 -139 -105 ...
##  $ total_accel_forearm : int  33 39 34 43 24 43 32 47 36 24 ...
##  $ gyros_forearm_x     : num  0.74 1.12 0.18 1.38 -0.75 -0.88 -0.53 0.63 0.03 0.02 ...
##  $ gyros_forearm_y     : num  -3.34 -2.78 -0.79 0.69 3.1 4.26 1.8 -0.74 0.02 0.13 ...
##  $ gyros_forearm_z     : num  -0.59 -0.18 0.28 1.8 0.8 1.35 0.75 0.49 -0.02 -0.07 ...
##  $ accel_forearm_x     : int  -110 212 154 -92 131 230 -192 -151 195 -212 ...
##  $ accel_forearm_y     : int  267 297 271 406 -93 322 170 -331 204 98 ...
##  $ accel_forearm_z     : int  -149 -118 -129 -39 172 -144 -175 -282 -217 -7 ...
##  $ magnet_forearm_x    : int  -714 -237 -51 -233 375 -300 -678 -109 0 -403 ...
##  $ magnet_forearm_y    : int  419 791 698 783 -787 800 284 -619 652 723 ...
##  $ magnet_forearm_z    : int  617 873 783 521 91 884 585 -32 469 512 ...
##  $ problem_id          : int  1 2 3 4 5 6 7 8 9 10 ...
```

```r
n <- 2
```

### Dimensionality Reduction
It has been noticed that the set of correlates can be divided into five sets:
1. Variables describing date, time, and wearer.  These are all variables that would not provide useful predictive assistance in any model.  What if the wearers in the testing set are different than those in the training set?  There would be no way for the model to provide predictions if this variable were to be included.  Similar arguments can be made with date and time.  They are all excluded from model formulation.
2. Measurements taken in the belt region.  These variables are useful, but many and correlated.  A principal component analysis will be performed, and only the first five components retained.
3. Measurements taken on the arm.  These will be handled in a fashion similar to variables in set 2.
4. Measurements taken with dumbbells.  These will be handled the same as in parts 2 and 3.
5. Measurements taken on the forearm.  Again, these will be handled the same as in parts 2, 3, and 4.

#### Principal Component Analysis of the Belt Variables

```r
library(caret)
head(training[,8:20])
```

```
##   roll_belt pitch_belt yaw_belt total_accel_belt gyros_belt_x gyros_belt_y
## 1      1.41       8.07    -94.4                3         0.00         0.00
## 2      1.41       8.07    -94.4                3         0.02         0.00
## 3      1.42       8.07    -94.4                3         0.00         0.00
## 4      1.48       8.05    -94.4                3         0.02         0.00
## 5      1.48       8.07    -94.4                3         0.02         0.02
## 6      1.45       8.06    -94.4                3         0.02         0.00
##   gyros_belt_z accel_belt_x accel_belt_y accel_belt_z magnet_belt_x
## 1        -0.02          -21            4           22            -3
## 2        -0.02          -22            4           22            -7
## 3        -0.02          -20            5           23            -2
## 4        -0.03          -22            3           21            -6
## 5        -0.02          -21            2           24            -6
## 6        -0.02          -21            4           21             0
##   magnet_belt_y magnet_belt_z
## 1           599          -313
## 2           608          -311
## 3           600          -305
## 4           604          -310
## 5           600          -302
## 6           603          -312
```

```r
beltPCA <- prcomp(na.omit(training[,8:20]))
beltPCA$sdev/sum(beltPCA$sdev)
```

```
##  [1] 0.4127198699 0.2238854590 0.1773136515 0.0661137809 0.0475660251
##  [6] 0.0285696755 0.0176203210 0.0112312822 0.0102890640 0.0035708699
## [11] 0.0005776826 0.0004240594 0.0001182592
```

```r
sum(beltPCA$sdev[1:n])/sum(beltPCA$sdev)
```

```
## [1] 0.6366053
```

```r
trainingPCAbelt <- as.matrix(training[,8:20]) %*% beltPCA$rotation[,1:n]
testingPCAbelt <- as.matrix(testing[,8:20]) %*% beltPCA$rotation[,1:n]
colnames(trainingPCAbelt) <- paste("belt",seq(1,n),sep="")
colnames(testingPCAbelt) <- paste("belt",seq(1,n),sep="")
head(trainingPCAbelt)
```

```
##          belt1    belt2
## [1,] -96.12325 47.72601
## [2,] -97.69540 46.51049
## [3,] -96.40425 42.13355
## [4,] -96.76751 46.74666
## [5,] -98.60676 41.87369
## [6,] -94.93995 45.06446
```
The above shows a preview of the original dataset columns followed by the proportion of variance in each component.  We take only the first two components and show a preview of the resulting data columns.

#### Principal Component Analysis of the Arm Variables

```r
head(training[,21:33])
```

```
##   roll_arm pitch_arm yaw_arm total_accel_arm gyros_arm_x gyros_arm_y
## 1     -128      22.5    -161              34        0.00        0.00
## 2     -128      22.5    -161              34        0.02       -0.02
## 3     -128      22.5    -161              34        0.02       -0.02
## 4     -128      22.1    -161              34        0.02       -0.03
## 5     -128      22.1    -161              34        0.00       -0.03
## 6     -128      22.0    -161              34        0.02       -0.03
##   gyros_arm_z accel_arm_x accel_arm_y accel_arm_z magnet_arm_x
## 1       -0.02        -288         109        -123         -368
## 2       -0.02        -290         110        -125         -369
## 3       -0.02        -289         110        -126         -368
## 4        0.02        -289         111        -123         -372
## 5        0.00        -289         111        -123         -374
## 6        0.00        -289         111        -122         -369
##   magnet_arm_y magnet_arm_z
## 1          337          516
## 2          337          513
## 3          344          513
## 4          344          512
## 5          337          506
## 6          342          513
```

```r
armPCA <- prcomp(na.omit(training[,21:33]))
armPCA$sdev/sum(armPCA$sdev)
```

```
##  [1] 0.4270205093 0.2024744668 0.0881417540 0.0844043549 0.0623707773
##  [6] 0.0456874358 0.0362013996 0.0279719705 0.0183606063 0.0053229711
## [11] 0.0015672286 0.0002846484 0.0001918774
```

```r
sum(armPCA$sdev[1:n])/sum(armPCA$sdev)
```

```
## [1] 0.629495
```

```r
trainingPCAarm <- as.matrix(training[,21:33]) %*% armPCA$rotation[,1:n]
testingPCAarm <- as.matrix(testing[,21:33]) %*% armPCA$rotation[,1:n]
colnames(trainingPCAarm) <- paste("arm",seq(1,n),sep="")
colnames(testingPCAarm) <- paste("arm",seq(1,n),sep="")
head(trainingPCAarm)
```

```
##           arm1      arm2
## [1,] -714.0576 -145.1857
## [2,] -713.8246 -141.6011
## [3,] -715.0025 -142.8081
## [4,] -717.9053 -141.2072
## [5,] -714.2291 -134.8741
## [6,] -715.5662 -143.6828
```
The above shows a preview of the original dataset columns followed by the proportion of variance in each component.  We take only the first two components and show a preview of the resulting data columns.

#### Principal Component Analysis of the Dumbbell Variables

```r
head(training[,34:46])
```

```
##   roll_dumbbell pitch_dumbbell yaw_dumbbell total_accel_dumbbell
## 1      13.05217      -70.49400    -84.87394                   37
## 2      13.13074      -70.63751    -84.71065                   37
## 3      12.85075      -70.27812    -85.14078                   37
## 4      13.43120      -70.39379    -84.87363                   37
## 5      13.37872      -70.42856    -84.85306                   37
## 6      13.38246      -70.81759    -84.46500                   37
##   gyros_dumbbell_x gyros_dumbbell_y gyros_dumbbell_z accel_dumbbell_x
## 1                0            -0.02             0.00             -234
## 2                0            -0.02             0.00             -233
## 3                0            -0.02             0.00             -232
## 4                0            -0.02            -0.02             -232
## 5                0            -0.02             0.00             -233
## 6                0            -0.02             0.00             -234
##   accel_dumbbell_y accel_dumbbell_z magnet_dumbbell_x magnet_dumbbell_y
## 1               47             -271              -559               293
## 2               47             -269              -555               296
## 3               46             -270              -561               298
## 4               48             -269              -552               303
## 5               48             -270              -554               292
## 6               48             -269              -558               294
##   magnet_dumbbell_z
## 1               -65
## 2               -64
## 3               -63
## 4               -60
## 5               -68
## 6               -66
```

```r
dumbbellPCA <- prcomp(na.omit(training[,34:46]))
dumbbellPCA$sdev/sum(dumbbellPCA$sdev)
```

```
##  [1] 0.4199770044 0.1634770047 0.1232674611 0.1177567030 0.0550047599
##  [6] 0.0457948295 0.0302347344 0.0230373762 0.0148754743 0.0034172543
## [11] 0.0025330157 0.0004288225 0.0001955602
```

```r
sum(dumbbellPCA$sdev[1:n])/sum(dumbbellPCA$sdev)
```

```
## [1] 0.583454
```

```r
trainingPCAdumbbell <- as.matrix(training[,34:46]) %*% dumbbellPCA$rotation[,1:n]
testingPCAdumbbell <- as.matrix(testing[,34:46]) %*% dumbbellPCA$rotation[,1:n]
colnames(trainingPCAdumbbell) <- paste("dumbbell",seq(1,n),sep="")
colnames(testingPCAdumbbell) <- paste("dumbbell",seq(1,n),sep="")
head(trainingPCAdumbbell)
```

```
##      dumbbell1 dumbbell2
## [1,]  650.6766  278.0744
## [2,]  649.6142  272.8792
## [3,]  655.3094  274.9748
## [4,]  652.5456  265.0670
## [5,]  646.1456  276.1185
## [6,]  650.3671  276.5384
```
The above shows a preview of the original dataset columns followed by the proportion of variance in each component.  We take only the first two components and show a preview of the resulting data columns.

#### Principal Component Analysis of the Forearm Variables

```r
head(training[,47:59])
```

```
##   roll_forearm pitch_forearm yaw_forearm total_accel_forearm
## 1         28.4         -63.9        -153                  36
## 2         28.3         -63.9        -153                  36
## 3         28.3         -63.9        -152                  36
## 4         28.1         -63.9        -152                  36
## 5         28.0         -63.9        -152                  36
## 6         27.9         -63.9        -152                  36
##   gyros_forearm_x gyros_forearm_y gyros_forearm_z accel_forearm_x
## 1            0.03            0.00           -0.02             192
## 2            0.02            0.00           -0.02             192
## 3            0.03           -0.02            0.00             196
## 4            0.02           -0.02            0.00             189
## 5            0.02            0.00           -0.02             189
## 6            0.02           -0.02           -0.03             193
##   accel_forearm_y accel_forearm_z magnet_forearm_x magnet_forearm_y
## 1             203            -215              -17              654
## 2             203            -216              -18              661
## 3             204            -213              -18              658
## 4             206            -214              -16              658
## 5             206            -214              -17              655
## 6             203            -215               -9              660
##   magnet_forearm_z
## 1              476
## 2              473
## 3              469
## 4              469
## 5              473
## 6              478
```

```r
forearmPCA <- prcomp(na.omit(training[,47:59]))
forearmPCA$sdev/sum(forearmPCA$sdev)
```

```
##  [1] 0.3104384071 0.1986666221 0.1980176957 0.0753406940 0.0677066969
##  [6] 0.0524206362 0.0425997539 0.0356768125 0.0122481511 0.0042945740
## [11] 0.0018976303 0.0004622940 0.0002300321
```

```r
sum(forearmPCA$sdev[1:n])/sum(forearmPCA$sdev)
```

```
## [1] 0.509105
```

```r
trainingPCAforearm <- as.matrix(training[,47:59]) %*% forearmPCA$rotation[,1:n]
testingPCAforearm <- as.matrix(testing[,47:59]) %*% forearmPCA$rotation[,1:n]
colnames(trainingPCAforearm) <- paste("forearm",seq(1,n),sep="")
colnames(testingPCAforearm) <- paste("forearm",seq(1,n),sep="")
head(trainingPCAforearm)
```

```
##      forearm1  forearm2
## [1,] 770.9755 -305.7626
## [2,] 776.5266 -301.2062
## [3,] 773.2122 -297.6661
## [4,] 773.0005 -297.7772
## [5,] 771.7628 -302.2524
## [6,] 774.7822 -306.1716
```

It is observed that in the four principal component analyses, the restriction to the top two principal components retains the vast majority of the variability in the group. More specifically for each group:
1. Belt: 63.7%
2. Arm: 62.9%
3. Dumbbell: 58.3%
4. Forearm: 50.9%

These variable groups are now reassembled along with the "classe" variable to predict, for the new matrix "training2", with which all analyses will be made.

```r
training2 <- as.data.frame(cbind(trainingPCAbelt,trainingPCAarm,trainingPCAdumbbell,trainingPCAforearm))
training2$classe <- training$classe
head(training2)
```

```
##       belt1    belt2      arm1      arm2 dumbbell1 dumbbell2 forearm1
## 1 -96.12325 47.72601 -714.0576 -145.1857  650.6766  278.0744 770.9755
## 2 -97.69540 46.51049 -713.8246 -141.6011  649.6142  272.8792 776.5266
## 3 -96.40425 42.13355 -715.0025 -142.8081  655.3094  274.9748 773.2122
## 4 -96.76751 46.74666 -717.9053 -141.2072  652.5456  265.0670 773.0005
## 5 -98.60676 41.87369 -714.2291 -134.8741  646.1456  276.1185 771.7628
## 6 -94.93995 45.06446 -715.5662 -143.6828  650.3671  276.5384 774.7822
##    forearm2 classe
## 1 -305.7626      A
## 2 -301.2062      A
## 3 -297.6661      A
## 4 -297.7772      A
## 5 -302.2524      A
## 6 -306.1716      A
```

```r
testing2 <- as.data.frame(cbind(testingPCAbelt,testingPCAarm,testingPCAdumbbell,testingPCAforearm))
head(testing2)
```

```
##        belt1      belt2       arm1      arm2 dumbbell1 dumbbell2  forearm1
## 1  136.77339 182.239780 -609.60141 -258.3134 -757.5611  12.02561  818.2145
## 2  -91.68464   3.669100 -690.61597 -145.0375  672.7235 121.04011 1106.7248
## 3 -101.07796   3.914502 -660.94694 -166.4886  653.3838 119.28929  942.5005
## 4  275.33215 -54.886409 -582.31985 -371.3951  613.7177 211.41477 1003.9039
## 5  -81.91938  88.296835 -583.09700 -365.2781  490.7029  22.67611 -796.6776
## 6 -100.12207  -7.089404  -24.04326 -603.4123  624.4923 173.52615 1143.0695
##    forearm2
## 1 -454.8086
## 2 -598.9654
## 3 -555.7036
## 4 -240.3284
## 5 -268.3964
## 6 -614.6515
```

Since the random forest technique has had a solid reputation with providing the best results for prediction, I am running this four times, each time training on 75% of the "training" data, and testing on the remaining 25% of the "training" data.  This should give a good estimate of the out-of-sample as well as the in-sample error.

#### Random Forests 1

```r
inTrain <- createDataPartition(training2$classe,p=3/4)[[1]]
training2train <- training2[inTrain,]
training2test <- training2[-inTrain,]
model.rf1 <- train(classe~.,method="rf",data=training2train)
model.rf1.predtrain <- predict(model.rf1,training2train)
model.rf1.predtest <- predict(model.rf1,training2test)
confusionMatrix(training2train$classe,model.rf1.predtrain)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 4185    0    0    0    0
##          B    0 2848    0    0    0
##          C    0    0 2567    0    0
##          D    0    0    0 2412    0
##          E    0    0    0    0 2706
## 
## Overall Statistics
##                                      
##                Accuracy : 1          
##                  95% CI : (0.9997, 1)
##     No Information Rate : 0.2843     
##     P-Value [Acc > NIR] : < 2.2e-16  
##                                      
##                   Kappa : 1          
##  Mcnemar's Test P-Value : NA         
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            1.0000   1.0000   1.0000   1.0000   1.0000
## Specificity            1.0000   1.0000   1.0000   1.0000   1.0000
## Pos Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
## Neg Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
## Prevalence             0.2843   0.1935   0.1744   0.1639   0.1839
## Detection Rate         0.2843   0.1935   0.1744   0.1639   0.1839
## Detection Prevalence   0.2843   0.1935   0.1744   0.1639   0.1839
## Balanced Accuracy      1.0000   1.0000   1.0000   1.0000   1.0000
```
#### Random Forests 2

```r
inTrain <- createDataPartition(training2$classe,p=3/4)[[1]]
training2train <- training2[inTrain,]
training2test <- training2[-inTrain,]
model.rf2 <- train(classe~.,method="rf",data=training2train)
model.rf2.predtrain <- predict(model.rf2,training2train)
model.rf2.predtest <- predict(model.rf2,training2test)
confusionMatrix(training2train$classe,model.rf2.predtrain)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 4185    0    0    0    0
##          B    0 2848    0    0    0
##          C    0    0 2567    0    0
##          D    0    0    0 2412    0
##          E    0    0    0    0 2706
## 
## Overall Statistics
##                                      
##                Accuracy : 1          
##                  95% CI : (0.9997, 1)
##     No Information Rate : 0.2843     
##     P-Value [Acc > NIR] : < 2.2e-16  
##                                      
##                   Kappa : 1          
##  Mcnemar's Test P-Value : NA         
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            1.0000   1.0000   1.0000   1.0000   1.0000
## Specificity            1.0000   1.0000   1.0000   1.0000   1.0000
## Pos Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
## Neg Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
## Prevalence             0.2843   0.1935   0.1744   0.1639   0.1839
## Detection Rate         0.2843   0.1935   0.1744   0.1639   0.1839
## Detection Prevalence   0.2843   0.1935   0.1744   0.1639   0.1839
## Balanced Accuracy      1.0000   1.0000   1.0000   1.0000   1.0000
```
#### Random Forests 3

```r
inTrain <- createDataPartition(training2$classe,p=3/4)[[1]]
training2train <- training2[inTrain,]
training2test <- training2[-inTrain,]
model.rf3 <- train(classe~.,method="rf",data=training2train)
model.rf3.predtrain <- predict(model.rf3,training2train)
model.rf3.predtest <- predict(model.rf3,training2test)
confusionMatrix(training2train$classe,model.rf3.predtrain)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 4185    0    0    0    0
##          B    0 2848    0    0    0
##          C    0    0 2567    0    0
##          D    0    0    0 2412    0
##          E    0    0    0    0 2706
## 
## Overall Statistics
##                                      
##                Accuracy : 1          
##                  95% CI : (0.9997, 1)
##     No Information Rate : 0.2843     
##     P-Value [Acc > NIR] : < 2.2e-16  
##                                      
##                   Kappa : 1          
##  Mcnemar's Test P-Value : NA         
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            1.0000   1.0000   1.0000   1.0000   1.0000
## Specificity            1.0000   1.0000   1.0000   1.0000   1.0000
## Pos Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
## Neg Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
## Prevalence             0.2843   0.1935   0.1744   0.1639   0.1839
## Detection Rate         0.2843   0.1935   0.1744   0.1639   0.1839
## Detection Prevalence   0.2843   0.1935   0.1744   0.1639   0.1839
## Balanced Accuracy      1.0000   1.0000   1.0000   1.0000   1.0000
```
#### Random Forests 4

```r
inTrain <- createDataPartition(training2$classe,p=3/4)[[1]]
training2train <- training2[inTrain,]
training2test <- training2[-inTrain,]
model.rf4 <- train(classe~.,method="rf",data=training2train)
model.rf4.predtrain <- predict(model.rf4,training2train)
model.rf4.predtest <- predict(model.rf4,training2test)
confusionMatrix(training2train$classe,model.rf4.predtrain)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 4185    0    0    0    0
##          B    0 2848    0    0    0
##          C    0    0 2567    0    0
##          D    0    0    0 2412    0
##          E    0    0    0    0 2706
## 
## Overall Statistics
##                                      
##                Accuracy : 1          
##                  95% CI : (0.9997, 1)
##     No Information Rate : 0.2843     
##     P-Value [Acc > NIR] : < 2.2e-16  
##                                      
##                   Kappa : 1          
##  Mcnemar's Test P-Value : NA         
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            1.0000   1.0000   1.0000   1.0000   1.0000
## Specificity            1.0000   1.0000   1.0000   1.0000   1.0000
## Pos Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
## Neg Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
## Prevalence             0.2843   0.1935   0.1744   0.1639   0.1839
## Detection Rate         0.2843   0.1935   0.1744   0.1639   0.1839
## Detection Prevalence   0.2843   0.1935   0.1744   0.1639   0.1839
## Balanced Accuracy      1.0000   1.0000   1.0000   1.0000   1.0000
```
#### Final Testing
In this last part we apply the four models to the final testing dataset.  These predictions are compiled into a data frame and their results compared.  We output the proportion of agreement.

```r
model.rf1.predtesttest <- predict(model.rf1,testing2)
model.rf2.predtesttest <- predict(model.rf2,testing2)
model.rf3.predtesttest <- predict(model.rf3,testing2)
model.rf4.predtesttest <- predict(model.rf4,testing2)
comp12 <- model.rf1.predtesttest == model.rf2.predtesttest
comp13 <- model.rf1.predtesttest == model.rf3.predtesttest
comp14 <- model.rf1.predtesttest == model.rf4.predtesttest
comp23 <- model.rf2.predtesttest == model.rf3.predtesttest
comp24 <- model.rf2.predtesttest == model.rf4.predtesttest
comp34 <- model.rf3.predtesttest == model.rf4.predtesttest
compall <- comp12 & comp23 & comp34
model.predtesttest <- data.frame(model.rf1.predtesttest,model.rf2.predtesttest,model.rf3.predtesttest,model.rf4.predtesttest,comp12,comp13,comp14,comp23,comp24,comp34,compall)
totalagreement <- dim(model.predtesttest[model.predtesttest$compall,])[1]/dim(model.predtesttest)[1]
totalagreement
```

```
## [1] 1
```

The above indicates complete agreement for all four models.

From these findings, we conclude the in-sample error to be, incredibly, zero. If out-of-sample error is not zero, then the peculiarities of the testing dataset will be only understandable via an extension of the model building process to include more principal components than just the first two. 

***

#### End of Analysis
