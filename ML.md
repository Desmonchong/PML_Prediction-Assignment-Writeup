# Peer-graded Assignment: Prediction Assignment Writeup

# Synopsis

##Instructions

One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants.


##What you should submit

The goal of your project is to predict the manner in which they did the exercise. This is the "classe" variable in the training set. You may use any of the other variables to predict with. You should create a report describing how you built your model, how you used cross validation, what you think the expected out of sample error is, and why you made the choices you did. You will also use your prediction model to predict 20 different test cases.

##Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the [website](http://groupware.les.inf.puc-rio.br/har): see the section on the Weight Lifting Exercise Dataset.

##Source of data

The training data for this project are available [here](https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv):

The test data are available [here](https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv):

The data for this project come from this [source](http://groupware.les.inf.puc-rio.br/har). If you use the document you create for this class for any purpose please cite them as they have been very generous in allowing their data to be used for this kind of assignment.


## Loading and preprocessing the data

### 1. Load required library

```
## Loading required package: caret
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```
## Loading required package: corrplot
```

```
## Loading required package: randomForest
```

```
## randomForest 4.6-12
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```
## 
## Attaching package: 'randomForest'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     margin
```

```
## Loading required package: rpart
```

```
## Loading required package: gbm
```

```
## Loading required package: survival
```

```
## 
## Attaching package: 'survival'
```

```
## The following object is masked from 'package:caret':
## 
##     cluster
```

```
## Loading required package: splines
```

```
## Loading required package: parallel
```

```
## Loaded gbm 2.1.1
```

```
## Loading required package: plyr
```
### 2. Load the data


```r
trainfile <- "pml-training.csv"
testfile <- "pml-testing.csv"

if (!file.exists(trainfile)){
        fileURL <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
        download.file(fileURL, trainfile, method="libcurl")
}  

if(trainfile %in% dir()){ 
        training <- read.csv(trainfile,header = TRUE) 
} 

if (!file.exists(testfile)){
        fileURL <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
        download.file(fileURL, testfile, method="libcurl")
}  

if(testfile %in% dir()){ 
        testing <- read.csv(testfile,header = TRUE) 
} 
```

### 3. Process/transform the data into a format suitable for analysis
+ #### 3.1. Cleaning Data


```r
# remove identification only variables (columns 1 to 5)
training <- training[, -(1:5)]
testing  <- testing[, -(1:5)]

c(length(training),length(testing))
```

```
## [1] 155 155
```

It is observed that both files have 155 variables, it will be very slow to predict will all the variables. To reduce the number of variables, one way is to remove column that is irrelevant to prediction like missing information (ie. NA).


```r
# determine the column with more than 95% NA
Low_NA_Column <- sapply(training, function(x) mean(is.na(x))) > 0.95
# remove those columns from both data
training <- training[, Low_NA_Column==FALSE]
testing  <- testing[, Low_NA_Column==FALSE]

c(length(training),length(testing))
```

```
## [1] 88 88
```

Now the number of variable is reduced to 88. The other way to reduce the variables further is to determine the variables that is near zero variance using nearZeroVar function and remove them.


```r
# Get the columns (variables) that is near zero variance
NZV <- nearZeroVar(training)
# Check the number of near zero variance
length(NZV)
```

```
## [1] 34
```

```r
# remove column with near zero variance
training <- training[, -NZV]
testing  <- testing[, -NZV]

c(length(training),length(testing))
```

```
## [1] 54 54
```
Now the number of variable is reduced to 54 only this is much more manageble and the data is said to be satisfy to be model.

+ #### 3.2. Data Partitioning

Partitioning training data set into two more data sets,70% for Model_training data, 30% for Model_test data as this will be used for cross validation purpose:


```r
# Randomly separate the data into respective data set
intraining <- createDataPartition(training$classe, p=0.70, list=F)

Model_training <- training[intraining, ]
Model_test <- training[-intraining, ]
```
### 4.Analysis Results

+ #### 4.1. Coorection Analysis

```r
corrplot(cor(Model_training[, -54]), order = "FPC", method = "color", type = "full", tl.cex = 0.6, tl.col = rgb(0, 0, 0))
```

![](ML_files/figure-html/unnamed-chunk-7-1.png)<!-- -->
The highly correlated variables are shown in dark colors in the graph above.

### 5.Prediction Model Building

There are 3 popular prediction model that can be used to predict the data, Random Forests, Decision Tree and Generalized Boosted Model. The best model based on accuracy will be selected for the quiz predictions.

+ #### 5.1. Random Forests

```r
contRF <- trainControl(method="cv", number=5, verboseIter=FALSE)
TrainRF <- train(classe ~ ., data=Model_training, method="rf",trControl=contRF)

predictRF <- predict(TrainRF, newdata=Model_test)
confMatRF <- confusionMatrix(predictRF, Model_test$classe)

round(confMatRF$overall['Accuracy'], 4)
```

```
## Accuracy 
##    0.998
```
+ #### 5.2. Decision Tree

```r
TrainDecTree <- rpart(classe ~ ., data=Model_training, method="class")

predictDecTree <- predict(TrainDecTree, newdata=Model_test, type="class")
confMatDecTree <- confusionMatrix(predictDecTree, Model_test$classe)
round(confMatDecTree$overall['Accuracy'], 4)
```

```
## Accuracy 
##   0.8221
```
+ #### 5.3. Generalized Boosted Model (GBM)

```r
contGBM <- trainControl(method = "repeatedcv", number = 5, repeats = 1)
TrainGBM  <- train(classe ~ ., data=Model_training, method = "gbm",
                    trControl = contGBM, verbose = FALSE)

predictGBM <- predict(TrainGBM, newdata=Model_test)
confMatGBM <- confusionMatrix(predictGBM, Model_test$classe)
round(confMatGBM$overall['Accuracy'], 4)
```

```
## Accuracy 
##   0.9856
```

### 6.Conclusions

+ #### 6.1. Prediction model selection

Since Random Forests score the highest accuracy, it will be used to will be applied to predict the 20 quiz results (testing dataset).


```r
predictTEST <- predict(TrainRF, newdata=testing)
predictTEST
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```

