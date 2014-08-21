---
title: "Prediction Assignment - Practical Machine Learning"
output: html_document
---

###Background:
Using devices such as *Jawbone Up*, *Nike FuelBand*, and *Fitbit* it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify *how well they do it*. This analysis uses data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. The goal of this project is to predict the manner in which they did the exercise.

###Environment initialization:
Load libraries used and set the seed for the pseudorandom number generator (for reproducibility).

```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
library(gbm)
```

```
## Loading required package: survival
## Loading required package: splines
## 
## Attaching package: 'survival'
## 
## The following object is masked from 'package:caret':
## 
##     cluster
## 
## Loading required package: parallel
## Loaded gbm 2.1
```

```r
set.seed(12345)
```

###Data Load and Processing:
Load the data from files, exclude the first six columns (not beneficial to this analysis), remove any "NA" columns, and finally, remove any "near zero" columns. Use the final "not near zero" column list from the training data to prepare the test data as well.

```r
trainData <- read.csv('pml-training.csv')
trainData <- trainData[,7:length(colnames(trainData))]
trainData <- trainData[,!apply(trainData, 2, function(x) any(is.na(x)))]
nsv <- nearZeroVar(trainData, saveMetrics=TRUE)
nsv <- nsv[!nsv$nzv,]
trainData <- trainData[,(names(trainData) %in% row.names(nsv))]

testData <- read.csv('pml-testing.csv')
testData <- testData[,(names(testData) %in% row.names(nsv))]
```

Partition the Data:
-----------
Partition the training data into three groups: 60% for training, 20% for testing, and 20% for validation.

```r
inTrain <- createDataPartition(y=trainData$classe, p=0.6, list=FALSE)
trainTrain <- trainData[inTrain,]
trainTV <- trainData[-inTrain,]
inTrain <- createDataPartition(y=trainTV$classe, p=0.5, list=FALSE)
trainTest <- trainTV[inTrain,]
trainValidate <- trainTV[-inTrain,]
```


Selecting a model:
==================

Recursive Partitioning:
-----------
Initially, I selected a Recursive Partitioning and Regression Trees model ("rpart"). I will not replicate the results here, but accuracy hovered around 50%. Pre-processing and cross validation did not help the fit meaningfully. Based on in-class lectures, I've decided to focus on Boosting and Random Forest models.

Boosting:
-----------
My second choice was "gbm" (Gradient Boosting Machine/Generalized Boosting Model). I tried gbm alone, with pre-processing (centered and scaled), with k-fold cross validation (k=5), and with both. Here is a summary of my results:

First Header  | Second Header
------------- | -------------
Content Cell  | Content Cell
Content Cell  | Content Cell


Generalized Boosting Model Accuracy:  
Options | Testing | Validation 
--- | --- | --- 
gbm alone | 98.98% | 98.78% 
preprocessing | 98.93% | 98.78% 
cross validation | 98.98% | 98.6% 
preproc and crossval | 99.08% | 98.93% 

I also wanted to try a few different values for k-fold cross validation in order to find the optimal balance of bias and variance. Here is a summary of my results:

Cross Validation Accuracy (gbm):  
"k"-fold | Testing | Validation  
--- | --- | ---  
3 | 98.98% | 98.78%  
5 | 98.93% | 98.78%  
7 | 98.98% | 98.6%  

Here are the final testing and validation results for my optimized gbm. The results for both were very close to 99%, with an **average (expected) out-of-sample error rate of 0.995%**.

```r
#gbmFit <- train(classe ~ ., method="gbm", data=trainTrain, verbose=FALSE, 
#                preProcess=c("center","scale"),
#                trControl=trainControl(method="cv", number=5))
```
Testing (gbm):

```r
#predTest <- predict(gbmFit, newdata=trainTest)
#print(confusionMatrix(predTest, trainTest$classe)) #99.08%
```
Validation (gbm):

```r
#predVal <- predict(gbmFit, newdata=trainValidate)
#print(confusionMatrix(predVal, trainValidate$classe)) #98.93%
```

Random Forest:
-----------
My final choice for a model was "random forest." Based on the marginal gain from pre-processing and cross validation results in the gbm model, I incuded both in the rf fit.


```r
#rfmFit <- train(classe ~ ., method="rf", data=trainTrain, verbose=FALSE, 
#                preProcess=c("center","scale"),
#                trControl=trainControl(method="cv", number=5))
```
Testing (rf):

```r
#predTest <- predict(rfmFit, newdata=trainTest)
#print(confusionMatrix(predTest, trainTest$classe)) #99.85%
```
Validation (rf):

```r
#predVal <- predict(rfmFit, newdata=trainValidate)
#print(confusionMatrix(predVal, trainValidate$classe)) #99.80%
```

Breiman and Cutler's random forests for classification and regression proved to be the best fit based on the training data. The accuracy on my test data was 99.85%, and 99.80% on my validation data. This averages to an **expected out-of-sample error rate of just 0.175%**. 

I applied the Random Forest model to the official testing data, submitted the 20 results, and achieved 100% accuracy for the predictions.
