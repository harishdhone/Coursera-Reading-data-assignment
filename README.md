# This  document describes step by step, the analysis done in "run_analysis.R" to transform the input data to create tidy data. Please refer to codebook in this repository for details on the input data and the processed tidy data


## 1. Loading required packages
library(dplyr)
library(tidyr)

## 2. Downloading the zip file

if(!file.exists("data")){dir.create("data")}
download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip",
              "./data/data.zip",method = "curl")

## 3. Extracting data from zip file
unzip("./data/data.zip",exdir = "./data")

## 4. Loading test, train and common datasets into R
x_test <- read.table("./data/UCI HAR Dataset/test/X_test.txt")
y_test <- read.table("./data/UCI HAR Dataset/test/Y_test.txt")
subject_test <- read.table("./data/UCI HAR Dataset/test/subject_test.txt")

x_train <- read.table("./data/UCI HAR Dataset/train/X_train.txt")
y_train <- read.table("./data/UCI HAR Dataset/train/Y_train.txt")
subject_train <- read.table("./data/UCI HAR Dataset/train/subject_train.txt")

features <- read.table("./data/UCI HAR Dataset/features.txt")
activity_labels <- read.table("./data/UCI HAR Dataset/activity_labels.txt")

## 5. Creating variable names
### 5a. Assigning feature names to variables in x train and x test 
colnames(x_test) <- t(features[,2])
colnames(x_train) <- t(features[,2])

### 5b. Assigning variable names to the rest
colnames(y_test) <- c("activity_id")
colnames(y_train) <- c("activity_id")

colnames(subject_test) <- c("subject")
colnames(subject_train) <- c("subject")

colnames(activity_labels) <- c("activity_id","activity")

## 6. Binding x-test, y-test and subject ids to create test dataset
test_data <- bind_cols(subject_test,y_test,x_test)

## 7. Binding x-train, y-train and subject ids to create training dataset
train_data <- bind_cols(subject_train,y_train,x_train)

## 8. Binding train and test datasets
test_data <- mutate(test_data,subject_type="test")
train_data <- mutate(train_data,subject_type = "train")
full_activity_data <- bind_rows(test_data,train_data)

## 9. Creating descriptive acitivity labels
full_activity_data <- merge(full_activity_data,activity_labels,
                         by.x = "activity_id",
                         by.y = "activity_id")
## 9. Converting to data frame tbl for better viewability
full_activity_data <- tbl_df(full_activity_data) 

## 10. Selecting only the mean/std variables
full_activity_data_1 <- select(full_activity_data,subject,activity, subject_type,
                               contains("mean()"),contains("std()"))

## 11. Grouping and summarizing by activity and subject
full_activity_data_2 <- group_by(full_activity_data_1,activity,subject,subject_type)

full_activity_data_3 <- summarise_all(full_activity_data_2,funs(mean))

## 12. Exporting the tidy data
write.table(full_activity_data_3,
            file = "./data/UCI HAR Dataset/avg_features_data.txt",row.names = FALSE,
            sep = "\t")
