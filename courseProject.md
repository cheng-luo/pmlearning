Predicting the way you exercised using activity data
========================================================

## Synopsis
With the advent of wearable devices, it becomes cheaper and easier to collet large amount of data about personal activities. These data can help researchers model and quantify activitiy patterns. In this project, we use the Weight Lifting Exercise Dataset from http://groupware.les.inf.puc-rio.br/har to model the exercise classes based on other features.

## Data Processing
### Loading the Raw Data

```r
testData = read.csv("pml-testing.csv", stringsAsFactors = T, na.strings = c("", 
    NA))

trainData = read.csv("pml-training.csv", stringsAsFactors = T, na.strings = c("", 
    NA))
```


### remove features that have more than 1/10 of missing values in the training data

```r
rowNum = dim(trainData)[1]
colNum = dim(trainData)[2]
indexes = integer()

for (index in 1:colNum) {
    naNum = sum(is.na(trainData[, index]))
    if (naNum >= rowNum * 0.1) {
        indexes = append(indexes, index)
    }
}

trainData = trainData[, -indexes]
```


### remove the row number,name, and time stamps

```r
trainData = trainData[, -(1:5)]
```


### set up cross-validation method for random forest to control training

```r
library(caret)
```

```
## Warning: package 'caret' was built under R version 3.0.3
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
fitControl = trainControl(method = "oob")
```


### Use the traning data and random forest method to train the model

```r
set.seed(13825)
modfit = train(classe ~ ., data = trainData, method = "rf", trControl = fitControl, 
    verbose = F)
```


## Results and discussions
### Random forest is used in this project because of its high accuracy.

### Cross validation
As discussed above, the parameter we used to find the optimal random forest model is OOB, out-of-bag errors. The final model has the smallest out-of-bag error.

### Out of bag error 

```r
modfit$results
```

```
##   Accuracy  Kappa mtry
## 1   0.9969 0.9961    2
## 2   0.9988 0.9985   28
## 3   0.9963 0.9953   54
```


The final value used for the model was mtry=41. Therefore the out of bag error is roughly 1-0.9988=0.0012=0.12%

### Predicting the test set

```r
pred = predict(modfit, testData)
```

```
## Loading required package: randomForest
## randomForest 4.6-7
## Type rfNews() to see new features/changes/bug fixes.
```

```r
pred
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```

