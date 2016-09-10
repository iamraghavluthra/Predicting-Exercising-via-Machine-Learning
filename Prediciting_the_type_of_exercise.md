Synopsis
--------

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now
possible to collect a large amount of data about personal activity
relatively inexpensively. In this report, I have used data from
accelerometers on the belt, forearm, arm, and dumbell of 6 participants.
They were asked to perform barbell lifts correctly and incorrectly in 5
different ways. I would like to thank Groupware @ LEs for letting me use
their dataset. The experimentation echnique and the dataset, can be
found on their [website](http://groupware.les.inf.puc-rio.br/har).

Also, this would have not been possible with our mentor, Mr. Greski,
whose articles helped us perform Parallel Processing in R.

Pre processing and Loading data
-------------------------------

    library(caret)

    ## Warning: package 'caret' was built under R version 3.3.1

    ## Loading required package: lattice

    ## Loading required package: ggplot2

    library(parallel)
    library(doParallel)

    ## Warning: package 'doParallel' was built under R version 3.3.1

    ## Loading required package: foreach

    ## Warning: package 'foreach' was built under R version 3.3.1

    ## Loading required package: iterators

    training <- read.csv("pml-training.csv")

    testing <- read.csv("pml-testing.csv")

Removing unnecessary variables
------------------------------

After having a quick look into the testing and training data, I observed
that many of the variables were NA throughout the dataset. So, I removed
them all with columns containing values of **kurtosis, skewness, avg,
max, min, var, stddev and amplitude**

    b <- names(testing)

    cols <- grep("kurtosis|skewness|avg|max|min|var|stddev|amplitude",b)

    train_ok <- training[,-cols]

Next, it was obvious that X, timestamp, number of window or new window
will be of any help in predicting if the user did the exercise correctly
ot not. So, I removed them also.

    train1_new <-train_ok[,c(2,8:60)]

Partioning the data
-------------------

I have reserved 30% of the training data to be reserved for cross
validation for myself and building the model on other 70%.

    inTrain <- createDataPartition(train1_new$classe,p=0.7,list=F)

    train1 <- train1_new[inTrain,]

    crossValid <- train1_new[-inTrain,]

Creating the algorithm
----------------------

Firstly, I have devoted 3 cores out of 4 to R for faster computations.

    cluster <- makeCluster(detectCores() - 1)

    registerDoParallel(cluster)

Creating the model
------------------

Hiding the algorithm due to Coursera guidelines.

However, the confusion matrix is as follows :

    confusionMatrix(pr,crossValid$classe)

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction    A    B    C    D    E
    ##          A 1670    6    0    0    0
    ##          B    3 1131    5    0    1
    ##          C    0    2 1019   23    1
    ##          D    0    0    2  940    7
    ##          E    1    0    0    1 1073
    ## 
    ## Overall Statistics
    ##                                           
    ##                Accuracy : 0.9912          
    ##                  95% CI : (0.9884, 0.9934)
    ##     No Information Rate : 0.2845          
    ##     P-Value [Acc > NIR] : < 2.2e-16       
    ##                                           
    ##                   Kappa : 0.9888          
    ##  Mcnemar's Test P-Value : NA              
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: A Class: B Class: C Class: D Class: E
    ## Sensitivity            0.9976   0.9930   0.9932   0.9751   0.9917
    ## Specificity            0.9986   0.9981   0.9946   0.9982   0.9996
    ## Pos Pred Value         0.9964   0.9921   0.9751   0.9905   0.9981
    ## Neg Pred Value         0.9990   0.9983   0.9986   0.9951   0.9981
    ## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
    ## Detection Rate         0.2838   0.1922   0.1732   0.1597   0.1823
    ## Detection Prevalence   0.2848   0.1937   0.1776   0.1613   0.1827
    ## Balanced Accuracy      0.9981   0.9955   0.9939   0.9866   0.9956

    stopCluster(cluster)
