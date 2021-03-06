---
title: "MachineLearning_Assignment"
author: "Mohsin"
date: "October 03, 2018"
output: html_document
---


## Introduction
### This report is the requirement for the course Practical Machine Learning as final assignment.  Broadly, the goal of the assignment is to explore personal activity data generated by mobile devices using machine learning. The packages used for analysis are mainly caret, rpart, and rattle. The report was published using knitr which is a method for elegantly publishing code, results, and charts in a single document of various formats, but here it is in HTML.

## Background Information
### Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset). (Source: Practical Machine Learning Course- Assignment Instructions)
.




## ---***Data Loading***---
### We are loading training and testing datasets from the working directory.

```r
training_data<-read.csv("pml-training.csv")
testing_data<-read.csv("pml-testing.csv")
```

## ---***Data Purging***---
### Here we are going to delete all columns that contain NA values and do away with the attributes not present in the testing data 
### The following attributes contain NA values: variance, mean and standard deviation. We are going to remove these variables.
### Furthermore, we are also going to do away with the first 7 columns that are having non-numeric nature. 

## ---***splitting the data***---
### We are going to split the dataset with 60% in the training set and the remaining 40% in the testing set. 

### In the end, we will filter out the characteristics that are in the testing.



```r
library(caret)
library(rpart)
library(rattle)
library(rpart.plot)
library(e1071)
library(ggplot2)
library(lattice)


attributes <- names(testing_data[,colSums(is.na(testing_data)) == 0])
attributes<-attributes[8:59]

training_data <- training_data[,c(attributes,"classe")]
testing_data <- testing_data[,c(attributes,"problem_id")]

dim(training_data) 
```

```
  [1] 19622    53
```

```r
dim(testing_data)
```

```
  [1] 20 53
```

```r
set.seed(12345)

inTrain <- createDataPartition(training_data$classe, p=0.6, list=FALSE)
train <- training_data[inTrain,]
test <- training_data[-inTrain,]

dim(train)
```

```
  [1] 11776    53
```

```r
dim(test)
```

```
  [1] 7846   53
```

## ---***Build  Decision Tree and Prediction Models***---
### In this part we shouldn't  be thinking of accuracy to be near 100% rather if it's near 80% it is worth accepting.
### Drecision Tree Model Prediction 
### Random Forest Model:An error  estimate of less than 3% should be expected.
### Random Forest Model Predictions
### Confusion Matrix 
### Predicting Through Decision Tree
### Predicting Through Random Forest-the optimal one


```r
library(randomForest)
library(RColorBrewer)


model_DTree <- rpart(classe ~ ., data = train, method="class")
fancyRpartPlot(model_DTree)
```

```
  Warning: labs do not fit even at cex 0.15, there may be some overplotting
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png)

```r
set.seed(12345)

predicting <- predict(model_DTree, test, type = "class")
confusionMatrix(predicting, test$classe)
```

```
  Confusion Matrix and Statistics
  
            Reference
  Prediction    A    B    C    D    E
           A 1879  260   30   69   66
           B   56  759   88   34   54
           C  105  340 1226  354  234
           D  155  132   23  807   57
           E   37   27    1   22 1031
  
  Overall Statistics
                                            
                 Accuracy : 0.7267          
                   95% CI : (0.7167, 0.7366)
      No Information Rate : 0.2845          
      P-Value [Acc > NIR] : < 2.2e-16       
                                            
                    Kappa : 0.6546          
   Mcnemar's Test P-Value : < 2.2e-16       
  
  Statistics by Class:
  
                       Class: A Class: B Class: C Class: D Class: E
  Sensitivity            0.8418  0.50000   0.8962   0.6275   0.7150
  Specificity            0.9243  0.96334   0.8405   0.9441   0.9864
  Pos Pred Value         0.8155  0.76589   0.5427   0.6874   0.9222
  Neg Pred Value         0.9363  0.88928   0.9746   0.9282   0.9389
  Prevalence             0.2845  0.19347   0.1744   0.1639   0.1838
  Detection Rate         0.2395  0.09674   0.1563   0.1029   0.1314
  Detection Prevalence   0.2937  0.12631   0.2879   0.1496   0.1425
  Balanced Accuracy      0.8831  0.73167   0.8684   0.7858   0.8507
```

```r
set.seed(12345)
model_RF <- randomForest(classe ~ ., data = train, ntree = 1000)

predicting <- predict(model_RF, test, type = "class")

confusionMatrix(predicting, test$classe)
```

```
  Confusion Matrix and Statistics
  
            Reference
  Prediction    A    B    C    D    E
           A 2230    9    0    0    0
           B    2 1505    7    0    0
           C    0    4 1361   16    2
           D    0    0    0 1268    4
           E    0    0    0    2 1436
  
  Overall Statistics
                                            
                 Accuracy : 0.9941          
                   95% CI : (0.9922, 0.9957)
      No Information Rate : 0.2845          
      P-Value [Acc > NIR] : < 2.2e-16       
                                            
                    Kappa : 0.9926          
   Mcnemar's Test P-Value : NA              
  
  Statistics by Class:
  
                       Class: A Class: B Class: C Class: D Class: E
  Sensitivity            0.9991   0.9914   0.9949   0.9860   0.9958
  Specificity            0.9984   0.9986   0.9966   0.9994   0.9997
  Pos Pred Value         0.9960   0.9941   0.9841   0.9969   0.9986
  Neg Pred Value         0.9996   0.9979   0.9989   0.9973   0.9991
  Prevalence             0.2845   0.1935   0.1744   0.1639   0.1838
  Detection Rate         0.2842   0.1918   0.1735   0.1616   0.1830
  Detection Prevalence   0.2854   0.1930   0.1763   0.1621   0.1833
  Balanced Accuracy      0.9988   0.9950   0.9957   0.9927   0.9978
```

```r
predicting_DTree <- predict(model_DTree, testing_data, type = "class")
predicting_DTree
```

```
   1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
   C  A  C  A  A  E  D  D  A  A  A  C  A  A  C  E  A  D  C  B 
  Levels: A B C D E
```

```r
predicting_RF <- predict(model_RF, testing_data, type = "class")
predicting_RF
```

```
   1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
   B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
  Levels: A B C D E
```




