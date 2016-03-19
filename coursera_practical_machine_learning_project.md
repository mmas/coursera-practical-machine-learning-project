
# Practical machine learning - John Hopkins University | Coursera
## Project

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website [here](http://groupware.les.inf.puc-rio.br/har) (see the section on the Weight Lifting Exercise Dataset).


The training data for this project are available [here](https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv). The test data are available [here](https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv). The data for this project come from [this source](http://groupware.les.inf.puc-rio.br/har). If you use the document you create for this class for any purpose please cite them as they have been very generous in allowing their data to be used for this kind of assignment.

The goal of your project is to predict the manner in which they did the exercise. This is the "classe" variable in the training set. You may use any of the other variables to predict with. You should create a report describing how you built your model, how you used cross validation, what you think the expected out of sample error is, and why you made the choices you did. You will also use your prediction model to predict 20 different test cases. 


```R
library(caret)
if (!require('doParallel')) {
    install.packages('doParallel', repos='http://cran.us.r-project.org')
    library(doParallel)
}
set.seed(123)
```


```R
data <- read.csv('pml-training.csv', na.strings=c('NA', '', '#DIV/0!')) 
summary(data)
sapply(data, function(row) sum(is.na(row))/length(row))
```

As we can see, there are a lot of columns with about 98% missing data. We filter out these features among the first seven ones, which are not relevant to build a model.


```R
data <- data[1:ncol(data)>7 & sapply(data, function(row) sum(is.na(row)/length(row)) < .9)]
summary(data)
```

Split the dataset into training and testing data.


```R
partition <- createDataPartition(y=data$classe, p=.7, list=FALSE)
training <- data[partition,]
testing <- data[-partition,]
```

Set up the parallel environment for training the model.


```R
control <- trainControl(method='repeatedcv', number=10, repeats=5, selectionFunction='oneSE')
cluster <- makeCluster(detectCores())
registerDoParallel(cluster)
```

Train a random forest classification model.


```R
model <- train(classe ~ ., data=training, method='rf', trControl=control)
stopCluster(cluster)
print(model$finalModel)
```

Predict new values and diagnostic.


```R
pred <- predict(model, testing)
table(pred, testing$classe)
```

Predict test cases.


```R
cases <- read.csv('pml-testing.csv')
predict(model, cases)
```
