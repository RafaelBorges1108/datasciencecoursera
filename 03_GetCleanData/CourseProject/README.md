## Getting and Cleaning Data
### Course Project

#### The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis.

#### The data was obtained from UCI Machine Leanirng Repository. A full description is avaliable at the site (http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones)

The script was divided in 5 steps, as follows:
1. Merges the training and the test sets to create one data set.
2. Extracts only the measurements on the mean and standard deviation for each measurement.
3. Uses descriptive activity names to name the activities in the data set
4. Appropriately labels the data set with descriptive variable names.
5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

##### Please, see CodeBook.md to obtain more details of the variables meaning

Loading packages
```
library(dplyr)
library(utils)
```
Downloading the data for the project and creating the directory
```
fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileUrl, "./Dataset.zip", method = "curl")
unzip("./Dataset.zip")
```
#### 1. Merges the training and the test sets to create one data set

Reading training dataset files
```
xtrain <- read.table("./UCI HAR Dataset/train/X_train.txt")
ytrain <- read.table("./UCI HAR Dataset/train/y_train.txt")
subjectTrain <- read.table("./UCI HAR Dataset/train/subject_train.txt")
```

Reading test dataset files
```
xtest <- read.table("./UCI HAR Dataset/test/X_test.txt")
ytest <- read.table("./UCI HAR Dataset/test/y_test.txt")
subjectTest <- read.table("./UCI HAR Dataset/test/subject_test.txt")
```

Checking the structure of each dataset
```
str(xtrain)
str(ytrain)
str(subjectTrain)
str(xtest)
str(ytest)
str(subjectTest)
```

Merging the training sets
```
dataTrain <- cbind(subjectTrain, ytrain, xtrain)
```

Merging the test sets
```
dataTest <- cbind(subjectTest, ytest, xtest)
```

Merging the training and the test datasets
```
MergedDataset <- rbind(dataTrain, dataTest)
```

#### 2. Extracts only the measurements on the mean and standard deviation for each measurement

Reading features file
```
features <- read.table("./UCI HAR Dataset/features.txt")
```

Combining the features names with subject id and activities in a character vector
```
featuresNames <- c("subject", "activity", as.character(features$V2))
```

Applying names to the variables of the merged dataset
```
colnames(MergedDataset) <- featuresNames
```

Filtering only measurements on the mean and standard deviation for each measurement and the two first columns with subject id and activities
```
ColNumbers <- grep("[Mm]ean\\(\\)|[Ss]td\\(\\)", names(MergedDataset))
ColNumbers <- c(1, 2, ColNumbers)
MeanDataset <- MergedDataset[, ColNumbers]
```

#### 3. Uses descriptive activity names to name the activities in the data set

Reading activity labels
```
activity <- read.table("./UCI HAR Dataset/activity_labels.txt")
```

Applying descriptive activities names to names the activities in the data set
```
dataset <- merge(MeanDataset, activity, by.x = "activity", by.y = "V1", all = TRUE, sort = FALSE)
```

Reordering the data set and cutting duplicated columns
```
dataset <- dataset[,c(2,69,3:68)]
```

#### 4. Appropriately labels the data set with descriptive variable names

Renaming columns
```
featuresNames <- names(dataset)
featuresNames[2] <- "activity"
featuresNames <- gsub("^t", "time", featuresNames)
featuresNames <- gsub("^f", "frequency", featuresNames)
featuresNames <- gsub("BodyBody", "Body", featuresNames)
colnames(dataset) <- featuresNames
```

#### 5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject

Reordering data set rows
```
dataset <- arrange(dataset, subject, activity)
dataset <- group_by(dataset, subject, activity)
```

Summarising the data set
```
tidydata <- summarise_all(dataset, mean)
```

Writing data set into txt file
```
write.table(tidydata, "./tidydata.txt", row.names = FALSE)
```
To read and view the tidy data set into R, please use the code
```
data <- read.table(file_path, header = TRUE)
View(data)
