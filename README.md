---
title: "AutoBarriersCode"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
Here we are loading and cleaning the data.  I placed the hand coded responses along with the RBBCSC and MCCSC files in this folder.  The extra code was just to establish the interrater reliablity from another data set.
```{r}
setwd("~/Desktop/QualPreAuto")
# matt = data.frame(read.csv("HandCodesMattMolly.csv", header = TRUE))
# head(matt)
# matt = matt[c(3:4)]
# head(matt)
# library(psych)
# cohen.kappa(matt)

```
Agreement above chance.  Do not use weighted, but this assumes there is some order to the nomial values.  So we are penalized more when I answer 1 and she answers 6 even though 

Now we are running the actual program.
```{r}
library(devtools)
#install_github("iqss-research/VA-package")
#install_github("iqss-research/ReadMeV1")
library(ReadMe)

```
Here we are cleaning the data.  Just grabbing the data and barriers question to SEL for both school districts and getting rid of the NA's.
```{r}
# rbbcsc = read.csv("RBBCSCStaffSurvey.csv", header = TRUE)
# rbbcsc = as.data.frame(rbbcsc)
# mccsc = read.csv("MCCSCStaffSurvey.csv", header = TRUE)

mccsc2 = mccsc[c("Q3")]
mccsc2 = mccsc2[-c(1:2), ]
mccsc2 = as.matrix(mccsc2)


rbbcsc1 = rbbcsc[c("Q3")]
rbbcsc2 = rbbcsc1[-c(1:2), ]
rbbcsc2 = as.matrix(rbbcsc2)


both = rbind(mccsc2, rbbcsc2)
dim(both)

both = rbind(mccsc2, rbbcsc2)
write.csv(both, "both.csv")
both = read.csv("both.csv", header= TRUE, na.strings = c("", "NA"))
both = na.omit(both)
id = 1:nrow(both)
both = cbind(id, both)
both = both[c(1,3)]
write.csv(both, "both.csv", row.names = FALSE)
head(both)
dim(both)
```
Now we need to place each of the responses into their own txt file.  Need to change the working directory to QualAuto to place the new documents into the new folder then you need to also include the both csv file so it can grab each text value and place it into its own txt file.   
```{r}
library(devtools)
source_url("https://gist.github.com/benmarwick/9266072/raw/csv2txts.R")
csv2txt("~/Desktop/QualAuto", labels = 1)
```
Now we are creating the control.txt file, because you already created the txt file with the responses.  Just creating the filename here.  Just creating a vector with 1:401.
```{r}
set.seed(1)
both1 <- replicate(401, rnorm(1, 0, 1))  
both1 = as.data.frame(t(both1))
dim(both1)


names(both1) <- paste0( "/Users/matthewhanauer/Desktop/QualAuto/",1:ncol(both1), ".txt")
library(reshape2)
both1 = melt(both1)
both1 = both1$variable
both1 = as.data.frame(both1)
colnames(both1) = c("filename")
filename = both1$filename
filename = as.data.frame(filename)
dim(filename)
```
Now we need to create the training and truth data sets.  We are grabbing matt's codes from the matt data set and that will be the "truth" data set.  Then we will stack on the "" for the rest of the values

Grabbing my codes for truth, which are the 1 through 7 values.  Then lining those up with the same text values as in the other data.  So my id 1 for my code needs to match the id 1 for the both data set.  Then creating 1 for stating that this in the truth data set and a 0 if it is not and we need a code.
```{r}
# setwd("~/Desktop/QualPreAuto")
#matt = read.csv("HandCodesMattMolly.csv", header = TRUE)
# setwd("~/Desktop/QualAuto")
truth = matt[c(3)]
names(truth) = c("truth")
truth = as.data.frame(truth)
dim(truth)
truth1 = data.frame(a = rep("", 401-100))
names(truth1) = c("truth")
truth1 = as.data.frame(truth1)
truth = rbind(truth, truth1)
dim(truth)
# Now we are creating the codes to indicate the training set
trainingset1 = data.frame(a = rep(1,100))
trainingset1 = as.data.frame(trainingset1)
trainingset2 = data.frame(a = rep(0, 401-100))
trainingset2 = as.data.frame(trainingset2)
trainingset = rbind(trainingset1, trainingset2)
names(trainingset) = c("trainingset")
dim(trainingset)
dim(filename)
dim(truth)
control = cbind(filename, truth, trainingset)
write.table(control, "control.txt", row.names = FALSE, sep = ",")
```
Now we run the program.  Before you run the program, you need to get rid of the "" everywhere.
```{r}
setwd("~/Desktop/QualAuto")
undergrad.results = undergrad(sep = ',')
undergrad.preprocess <- preprocess(undergrad.results)
readme.results <- readme(undergrad.preprocess, boot.se = TRUE)
readme.results$CSMF.se
readme.results$est.CSMF
testReadme = readme.results$subsets.est
write.csv(testReadme, "testReadme.csv", row.names = FALSE)
```

