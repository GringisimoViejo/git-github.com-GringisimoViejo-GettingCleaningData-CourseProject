# SOURCE DATA:

This script cleans the data set of Samsung Galaxy II sensor readings that is found at: 
https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip 

The data set is made up of several text files that must be fit together from the origanal
zip file. To create the data set, these must be fit together. The training and test data 
were randomly split 70% - 30% respectively by participant in the original file.

- X_train.txt - The training data for the machine learning, with 561 columns
- X_test.txt - The test data for the machine learning, with 561 columns
- y_train.txt - Six activity codes matched by row to the training data
- y_test.txt - Six activity codes matched by row to the test data
- subject_train.txt - A selection from thirty participant codes matched by row to the training data
- subject_test.txt - A selection from thirty participant codes matched by row to the test data
- features.txt - A file of 561 column headers that match the columns in the training and test data
- activity_labels.txt - A table of values 1 through 6 to assign descriptive values to activity codes

==============================

# PROCESS:

Because the script relies on the Dplyr R package to properly clean the data, this package
must installed. The script automatically loads it, however, prior to running the other
steps.

After loading Dplyr, the run_analytics script pulls in the features.txt file that will make up the column headers 
and names it features. Only one column of features is needed so the other columns is stripped
out. 

Next, most longer terms used in the relevant column headers are shorted in order to
preserve as much descriptive text as possible. The first letter of each is capitalized
for ease of reading. This is done early in the process because once these terms 
become column headers, there will be an additional loss of information from name 
truncation that cannot be avoided at that stage. 

- features <-read.table("C:/Users/Robert/Desktop/Data Science - Johns Hopkins/SamsungAssignment/features.txt", quote="\"", comment.char="")
- features <- features[, 2]
- features <- gsub("Body","Bdy", features)
- features <- gsub("Jerk","Jrk", features)
- features <- gsub("Gyro","Gyr", features)
- features <- gsub("entropy","Ent", features)
- features <- gsub("Freq","Frq", features)
- features <- gsub("Gravity","Grav", features)

The other columns of the data set are then pulled in and given simplified names. The
files that became xtest and xtrain (X_test.txt and X_train.txt, respectively) came from
the same trials originally but were split apart. For our purposes, we are rejoining them.

As xtest and xtrain are loaded, the column names found in features replace the original,
non-descriptive column names. 

The y_test.txt and y_train.txt files are loaded and renamed as ytest and ytrain. These 
supply the six activity codes describing which activies were being measured.

The subject_test.txt and subject_train.txt files are loaded and renamed as subtest and 
subtrain.

Finally, activity_labels.txt is loaded and renamed as actlabels.

- xtest <- read.table("C:/Users/Robert/Desktop/Data Science - Johns Hopkins/SamsungAssignment/X_test.txt", quote="\"", comment.char="", col.names = features)
- ytest <- read.table("C:/Users/Robert/Desktop/Data Science - Johns Hopkins/SamsungAssignment/y_test.txt", quote="\"", comment.char="")
- xtrain <- read.table("C:/Users/Robert/Desktop/Data Science - Johns Hopkins/SamsungAssignment/x_train.txt", quote="\"", comment.char="", col.names = features)
- ytrain <- read.table("C:/Users/Robert/Desktop/Data Science - Johns Hopkins/SamsungAssignment/y_train.txt", quote="\"", comment.char="")
- subtest <- read.table("C:/Users/Robert/Desktop/Data Science - Johns Hopkins/SamsungAssignment/subject_test.txt", quote="\"", comment.char="")
- subtrain <- read.table("C:/Users/Robert/Desktop/Data Science - Johns Hopkins/SamsungAssignment/subject_train.txt", quote="\"", comment.char="")
- actlabels <- read.table("C:/Users/Robert/Desktop/Data Science - Johns Hopkins/SamsungAssignment/activity_labels.txt", quote="\"", comment.char="")

Next, rbind is used to create xfile out of xtest and xtrain as well as yfile out of 
ytest and ytrain and then subfile out of subtest and subtrain.

- xfile <- rbind(xtest, xtrain)
- yfile <- rbind(ytest, ytrain)
- subfile <- rbind(subtest, subtrain)

Then, actlabels is used as a look-up table to convert the numeric activity labels to
descriptive activity labels.

- yfile$V1 <- as.factor(actlabels$V2[yfile$V1])

A data frame is created with the numbers representing the subjects in the left-most
column, the descriptive activity names in the second column from the left, and the
measurements in the rest of the columns. This data frame is named rawset.

- rawset <- data.frame(subfile, yfile, xfile)

The two columns added to the left side of the data frame to create rawset are then 
renamed as Participant and Activity.

- rawset <- rename(rawset, Participant = V1, Activity = V1.1)

There were originally 561 columns of measurements of which only those with means or 
standard deviations, 66 total measurement columns, are needed. The other columns are
removed.

- rawset <- rawset[,(grepl("Participant|Activity|\\bmean\\b|\\bstd\\b", colnames(rawset)))]

Previously, longer terms in the features object had been shortened. Mean and STD
were not shortened or adjusted previously because the long form was preferred for the 
filtering in the last mitigate the possibility of incorrect filtering. Extraneous
periods are also removed at this stage.

- colnames(rawset) <- gsub("mean","Mn", colnames(rawset))
- colnames(rawset) <- gsub("std","STD", colnames(rawset))
- colnames(rawset) <- gsub("\\b...\\b","", colnames(rawset))

A refined set, named finalset, is grouped by Participant then further grouped by
Activity. For each column, the mean of all values for that group is taken as the only
value to be kept. The results are then exported as a file named cleantable.txt.

- finalset <- group_by(rawset, Participant, Activity)
- secondset <- (finalset %>% summarize_each(funs(mean)))
- write.table(secondset, file = "cleantable.txt", row.name = FALSE)

The cleantable.txt file has 180 rows, not including headers, and 68 columns of which 66 are
the means of values summarized for by participant and activity.