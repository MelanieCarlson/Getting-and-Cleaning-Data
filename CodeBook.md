---
title: "run_analysis.R"
author: "Melanie Carlson"
date: "Sunday, May 24, 2015"
output: html_document
---

This script will perform the following steps on the UCI HAR Dataset downloaded from 
# https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip 
# 1. Merge the training and the test sets to create one data set.
# 2. Extract only the measurements on the mean and standard deviation for each measurement. 
# 3. Use descriptive activity names to name the activities in the data set
# 4. Appropriately label the data set with descriptive activity names. 
# 5. Creates a second, independent tidy data set with the average of each variable for each activity and each subject. 
#######################################################################

Set Working Directory:

```{r}
setwd("C:/Users/Melanie/Desktop/R Code Class/Getting and Cleaning Data")
```

Read Training Data from Files
```{r}
> features     = read.table('./features.txt',header=FALSE); #imports features.txt
> activityType = read.table('./activity_labels.txt',header=FALSE); #imports activity_labels.txt
> subjectTrain = read.table('./train/subject_train.txt',header=FALSE); #imports subject_train.txt
> xTrain       = read.table('./train/x_train.txt',header=FALSE); #imports x_train.txt
> yTrain       = read.table('./train/y_train.txt',header=FALSE); #imports y_train.txt
```

Set Training COlumn Names
```{r}
> colnames(activityType)  = c('activityId','activityType');
> colnames(subjectTrain)  = "subjectId";
> colnames(xTrain)        = features[,2]; 
> colnames(yTrain)        = "activityId";
```

Merge Column in Training Data Set
```{r}
> trainingData = cbind(yTrain,subjectTrain,xTrain);
```

Read Testing Data from Files
```{r}
> subjectTest = read.table('./test/subject_test.txt',header=FALSE); #imports subject_test.txt
> xTest       = read.table('./test/x_test.txt',header=FALSE); #imports x_test.txt
> yTest       = read.table('./test/y_test.txt',header=FALSE); #imports y_test.txt
```

Set Test Data COlumn Names
```{r}
> colnames(subjectTest) = "subjectId";
> colnames(xTest)       = features[,2];
> colnames(yTest)       = "activityId";
```

Merge Column in Testing Data Set
```{r}
> testData = cbind(yTest,subjectTest,xTest);
```

# 1. Merge the training and the test sets to create one data set.
```{r}
> finalData = rbind(trainingData,testData);
```

# 2. Extract only the measurements on the mean and standard deviation for each measurement.
```{r}
> colNames  = colnames(finalData);
> logicalVector = (grepl("activity..",colNames) | grepl("subject..",colNames) | grepl("-mean..",colNames) & !grepl("-meanFreq..",colNames) & !grepl("mean..-",colNames) | grepl("-std..",colNames) & !grepl("-std()..-",colNames));
> finalData = finalData[logicalVector==TRUE];
```

# 3. Use descriptive activity names to name the activities in the data set
```{r}
> finalData = merge(finalData,activityType,by='activityId',all.x=TRUE);
> colNames  = colnames(finalData); 
```

# 4. Appropriately label the data set with descriptive activity names. 
```{r}
> for (i in 1:length(colNames)) {
+ colNames[i] = gsub("\\()","",colNames[i])
+   colNames[i] = gsub("-std$","StdDev",colNames[i])
+   colNames[i] = gsub("-mean","Mean",colNames[i])
+   colNames[i] = gsub("^(t)","time",colNames[i])
+   colNames[i] = gsub("^(f)","freq",colNames[i])
+   colNames[i] = gsub("([Gg]ravity)","Gravity",colNames[i])
+   colNames[i] = gsub("([Bb]ody[Bb]ody|[Bb]ody)","Body",colNames[i])
+   colNames[i] = gsub("[Gg]yro","Gyro",colNames[i])
+   colNames[i] = gsub("AccMag","AccMagnitude",colNames[i])
+   colNames[i] = gsub("([Bb]odyaccjerkmag)","BodyAccJerkMagnitude",colNames[i])
+   colNames[i] = gsub("JerkMag","JerkMagnitude",colNames[i])
+   colNames[i] = gsub("GyroMag","GyroMagnitude",colNames[i]) }
> colnames(finalData) = colNames; 
```
# 5. Creates a second, independent tidy data set with the average of each variable for each activity and each subject.
```{r}
> finalDataNoActivityType  = finalData[,names(finalData) != 'activityType'];
> tidyData    = aggregate(finalDataNoActivityType[,names(finalDataNoActivityType) != c('activityId','subjectId')],by=list(activityId=finalDataNoActivityType$activityId,subjectId = finalDataNoActivityType$subjectId),mean);
> tidyData    = merge(tidyData,activityType,by='activityId',all.x=TRUE);
> write.table(tidyData, './tidyData.txt',row.names=TRUE,sep='\t');
```
