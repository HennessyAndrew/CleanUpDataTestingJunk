# CleanUpDataTestingJunk
CourseraWeek4
Not sure what we input here as it has not been part of the last four classes on coursera.
#I used some examples here https://rpubs.com/ninjazzle/DS-JHU-3-4-Final
#I needed to load library(dplyr) &  library(tidyverse)
#Set a working directory and look for zip and existing and create as needed 
setwd("C:/Users/Charlotte/Census/Coursera/NewNumber4")
filename <- "Coursera_DS3_Final.zip"
# Checking if already exists.
if (!file.exists(filename)){
  fileURL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
  download.file(fileURL, filename)
}  
# Checking if folder exists
if (!file.exists("UCI HAR Dataset")) { 
  unzip(filename) 
}

Unpack ZIP files
##Load the unzipped files
# load features.txt. It is a list of all features 
# col.names names columns in the table = "n" and "functons"
features <- read.table("UCI HAR Dataset/features.txt", col.names = c("n","functions"))

#load activity_labels. These are the activities performed
# col.names names columns = "code" & "activity
activities <- read.table("UCI HAR Dataset/activity_labels.txt", col.names = c("code", "activity"))

#Work with test files TEST###############################TEST############################################
#load subject_test.txt and name the column = "subject"
subject_test <- read.table("UCI HAR Dataset/test/subject_test.txt", col.names = "subject")

#loads data set X_test.txt
#Adds column names via the contents of features using the functions column features$functions
x_test <- read.table("UCI HAR Dataset/test/X_test.txt", col.names = features$functions)
# Import y_test.txt
# set col.name = "Code"
y_test <- read.table("UCI HAR Dataset/test/y_test.txt", col.names = "code")


#Work with TRAINfiles ###############################TRAIN############################################
#load subject_test.txt and name the column = "subject"
subject_train <- read.table("UCI HAR Dataset/train/subject_train.txt", col.names = "subject")

#loads data set X_train.txt
#Adds column names via the contents of features using the functions column features$functions
x_train <- read.table("UCI HAR Dataset/train/X_train.txt", col.names = features$functions)
# Import y_train.txt
# set col.name = "Code"
y_train <- read.table("UCI HAR Dataset/train/y_train.txt", col.names = "code")

#Merge x_train & x_test data sets using the columns from features$functions
X <- rbind(x_train, x_test)

#Merge y_train & y_test data sets using the column = Code
Y <- rbind(y_train, y_test)

#Merge subject_train & subject_test data sets using. They both use the column = Subject
Subject <- rbind(subject_train, subject_test)
#Merge Subject, X, & Y datasets
Merged_Data <- cbind (Subject, X, Y)

# USe select on Merged_Data We want to get measurements on the mean and standard deviation from Merged_Data.
#Get some junk here sine mean and std are all over the data set
TidyData <- Merged_Data %>% select (subject, code, contains("mean"), contains("std"))

#use activities file to update the "Code" column numeric values with strings from activites
TidyData$code <- activities[TidyData$code, 2]
names(TidyData)[2] = "activity"
# use gsub() to update names/labels in the vector
# change ACC to Accelerometer, "Gyro" to "Gyroscope", etc
names(TidyData)<-gsub("Acc", "Accelerometer", names(TidyData))
names(TidyData)<-gsub("Gyro", "Gyroscope", names(TidyData))
names(TidyData)<-gsub("BodyBody", "Body", names(TidyData))
names(TidyData)<-gsub("Mag", "Magnitude", names(TidyData))
names(TidyData)<-gsub("^t", "Time", names(TidyData))
names(TidyData)<-gsub("^f", "Frequency", names(TidyData))
names(TidyData)<-gsub("tBody", "TimeBody", names(TidyData))
names(TidyData)<-gsub("-mean()", "Mean", names(TidyData), ignore.case = TRUE)
names(TidyData)<-gsub("-std()", "STD", names(TidyData), ignore.case = TRUE)
names(TidyData)<-gsub("-freq()", "Frequency", names(TidyData), ignore.case = TRUE)
names(TidyData)<-gsub("angle", "Angle", names(TidyData))
names(TidyData)<-gsub("gravity", "Gravity", names(TidyData))

#create new data set from TidyData
#That creates means subject to subject and activity values
#writes out data to locally
FinalData <- TidyData %>%
  group_by(subject, activity) %>%
  summarise_all(funs(mean))
write.table(FinalData, "FinalData.txt", row.name=FALSE)

