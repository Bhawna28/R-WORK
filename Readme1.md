---
title: "Getting and Cleaning data Project"
author: "Bhawna"
date: "8/26/2020"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

#Human Activity Recognition Using Smartphones Dataset
##Version 1.0

The experiments have been carried out with a group of 30 volunteers within an age bracket of 19-48 years. Each person performed six activities (WALKING, WALKING_UPSTAIRS, WALKING_DOWNSTAIRS, SITTING, STANDING, LAYING) wearing a smartphone (Samsung Galaxy S II) on the waist. Using its embedded accelerometer and gyroscope, we captured 3-axial linear acceleration and 3-axial angular velocity at a constant rate of 50Hz. The experiments have been video-recorded to label the data manually. The obtained dataset has been randomly partitioned into two sets, where 70% of the volunteers was selected for generating the training data and 30% the test data. 

The sensor signals (accelerometer and gyroscope) were pre-processed by applying noise filters and then sampled in fixed-width sliding windows of 2.56 sec and 50% overlap (128 readings/window). The sensor acceleration signal, which has gravitational and body motion components, was separated using a Butterworth low-pass filter into body acceleration and gravity. The gravitational force is assumed to have only low frequency components, therefore a filter with 0.3 Hz cutoff frequency was used. From each window, a vector of features was obtained by calculating variables from the time and frequency domain.

```{r}
library(R.utils)
library(data.table)
library(tidyr)
library(dplyr)
```

#Reading in features vector
```{r}
features<-fread("C:/Users/KALYANI/Desktop/R Projects/R-WORK/UCI HAR Dataset/features.txt")
dim(features)
```

#Reading in x test and x-train data and merging
```{r}
xtest<-read.table("C:/Users/KALYANI/Desktop/R Projects/R-WORK/x/test/X_test.txt")
xtrain<-read.table("C:/Users/KALYANI/Desktop/R Projects/R-WORK/x/train/X_train.txt")
xmerged<-rbind(xtest,xtrain)
dim(xmerged)
```

#Renaming columns of xmerged data frame
```{r}
for(i in 1:561){
  colnames(xmerged)[i]<-features$V2[i] 
}

colnames(xmerged)[1]
```

#Reading in activities data 
```{r}
ytest<-read.table("C:/Users/KALYANI/Desktop/R Projects/R-WORK/x/test/y_test.txt")
ytrain<-read.table("C:/Users/KALYANI/Desktop/R Projects/R-WORK/x/train/y_train.txt")
ymerged<-rbind(ytest,ytrain)
dim(ymerged)
```

#Reading in subject data
```{r}
subjectstest<-read.table("C:/Users/KALYANI/Desktop/R Projects/R-WORK/X/subject_test.txt")
subjectrain<-read.table("C:/Users/KALYANI/Desktop/R Projects/R-WORK/UCI HAR Dataset/train/subject_train.txt")
subjectmerged<-rbind(subjectstest,subjectrain)
dim(subjectmerged)
```

#Dataset combining subject,activities and observations
```{r}
actdata<-cbind(subjectmerged,ymerged,xmerged)
colnames(actdata)[1]<-"subject"
colnames(actdata)[2]<-"activities"
```

#Re-labelling acticities levels
```{r}
actdata$activities<-factor(actdata$activities,levels=c("1","2","3","4","5","6" ))
levels(actdata$activities)<-c("WALKING","WALKING_UPSTAIRS","WALKING_DOWNSTAIRS","SITTING","STANDING","LAYING")
unique(actdata$activities)
```

#Extracting oservation on mean and standard deviation
```{r}
obsmean<-actdata[,grep("mean",colnames(actdata))]
obsstd<-actdata[,grep("std",colnames(actdata))]
obs<-cbind(obsmean,obsstd)
colnames(obs)
```

#Renaming variable names
```{r}
colnames(obs)<-gsub("t^","time",colnames(obs))
colnames(obs)<-gsub("f","frequency",colnames(obs))
obs<-cbind(actdata[,1:2],obs)
```



#Averaging out observations in each column per activity per subject
```{r}
tidydata<-obs%>%group_by(activities,subject)%>%summarise_all(mean)
dim(tidydata)
str(tidydata)
```

#Creating tidy data with write.table() function
```{r}
write.table(obs,file ="C:/Users/KALYANI/Desktop/R Projects/R-WORK/UCI HAR Dataset/finaltable.csv",row.names = FALSE)
finaltable<-read.table("C:/Users/KALYANI/Desktop/R Projects/R-WORK/UCI HAR Dataset/finaltable.csv",header=TRUE,sep=" ")
```
