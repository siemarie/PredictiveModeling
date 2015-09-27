---
title: "Modeling Weight Lifting Form with Accelerometer Readings"
author: "Jessie Lamb"
date: "Sept. 26, 2015"
output: html_document
---
# Modeling Weight Lifting Form with Accelerometer Readings

## Executive Summary
This analysis explores the Weight Lifting Exercises Dataset (<http://groupware.les.inf.puc-rio.br/har>) from the Human Activity Recognition study.

Specifically, this analysis is interested in using predictive modeling to predict weight lifting form (variable *classe*) based upon previously observed accelerometer readings.
 
### Methodology
 * Training data (<https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv>) was split into a training set and a hold out set for cross validation testing.
 * The training set was cleaned to remove variables missing more than 50% of readings, those not directly the result of the accelerometer readings and those with near zero variance.
 * A model was fit to the training set using Random Forests with 3 cross-validating folds.
 * The model was tested out of sample on the testing (hold out) set from the original training dataset.

## Loading the data
The caret library and training data are loaded. The na.strings() method is used to convert all missing data to NA records.

```{r}
require("caret");
library(caret);
dataSet<<-read.csv("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv",na.strings=c("NA","#DIV/0!",""));
```

## Spliting the training set
The training dataset is split into training and testing groups in order to have a data set to test the model against. This will help estimate the accuracy of the prediction model before it is applied to the test set to predict exercise form.

```{r}
set.seed(1234);
inTrain <- createDataPartition(y=dataSet$classe,p=0.6,list=FALSE)
training <<- dataSet[inTrain,]
testing <<- dataSet[-inTrain,]
```

## Cleaning the training data
Not all variables in the training set are useful for predicting weight lifting form. Some columns are missing too many values to help predict outcomes. Additionally, some data -- such as user name (variable *user_name*), record ID (variable *X*) and several columns recording timestamps -- are unlikely to provide meaningful insight and should be removed from the model. Finally, some columns contain limited variability, indicating that they also may not be useful in predicting outcomes.

```{r}
tr2<<-training[-grep("timestamp", names(training ))] ## Remove timestamp variables.
tr3<<-tr2[colSums(is.na(tr2))/nrow(tr2)<.5] ## Remove columns with more than 50% of values missing.
exclude<<-nearZeroVar(tr3) ## Remove variables with limited variability.
exclude[length(exclude)+1]<-1 ## Remove the record ID variable (X).
exclude[length(exclude)+1]<-2 ## Remove the participant name variable (user_name).
trainingClean<<-tr3[-exclude]
```

## Building the prediction model
The Random Forest training method was used with 3-fold cross validation controls to fit the model. Random Forests was used due to its accuracy and ease of use with factor variables such as the classe variable. The best model used 27 variables and fit the training set with 99.5% accuracy and a low estimated error rate of 0.34%.

```{r}
control <- trainControl(method="cv", number=3, verboseIter=F)
modelFit<<-train(classe~., method="rf", data=trainingClean,trControl=control)
modelFit
modelFit$finalModel
```

## Testing the model
In order to insure the model is not overfit to the training set, it was next tested against the hold out data, *testing*. The out-of-sample accuracy of the model actually improved to 99.7% with a strong confidence interval (99.55% - 99.80%) and a statistically significant p-value. The out-of-sample error rate was 0.31%.

```{r}
test<-predict(modelFit,newdata=testing)
confusionMatrix(testing$classe,test)
```

## Predicting with the model
The test set is then loaded and the prediction model used to predict *classe*, the quality of the participant's weight listing technique. The model predicts that 7 of the 20 participants performed the exercise correctly (class A) while the remaining 13 are predicted to have a variety of incorrect techniques (classes B-E).

```{r}
dataSet2<<-read.csv("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv");
predictClasse<-predict(modelFit,newdata=dataSet2)
dataSet2$classe<-predictClasse
dataSet2
```
