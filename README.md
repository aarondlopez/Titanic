# Titanic

Kaggle Competition

Load Data and Libraries

```{r, message=F}
# Load data and libraries
library(dplyr)
library(tidyr)
library(ggplot2)
library(ggthemes)
library(randomForest)
library(rpart)
library(rpart.plot)
library(tibble)
train <- read.csv("train.csv", stringsAsFactors = F)
test <- read.csv("test.csv", stringsAsFactors = F)
full <- bind_rows(train, test)
```

Data Clean Up

```{r, message=F, results="hide"}
# Data Wrangling
str(full)
summary(full)
# Age has alot of missing values, so I'll look at replacing those first
summary(full$Age)
# We'll replace the NA's with the mean age of 29.88
full$Age[is.na(full$Age)] <- 29.88
# Remove two missing embarked data points 62 and 830
full <- full[-c(62, 830), ]
# Replace missing fare values
summary(full$Fare)
full$Fare[is.na(full$Fare)] <- 33.22
```

Explore the Data

```{r, message=F}
# Exploratory Analysis
# There is a wide range of ages and since the data was missing many ages the median age is amplified.
ggplot(full, aes(Age, fill=Age)) + geom_histogram(binwidth = 5)
# The higher the class the higher the averaage fare paid, but some passengers got better deals than others.
ggplot(full, aes(Pclass, Fare, group=Pclass, fill=Pclass)) + geom_boxplot() + theme_gdocs()
# How does where a passenger embarked from effect the fare they paid? 
ggplot(full, aes(Embarked, Fare, fill=Embarked)) + geom_boxplot() + theme_gdocs()
# Does where passangeres embarked have an impact on surviving?
table(full$Survived, full$Embarked)
# Most people embarked from S and they also had the most casualties
# Females survived at a m uch higher rate than males
table(full$Survived, full$Sex)
# What about class? 3rd class passangers were least likely to survive
table(full$Survived, full$Pclass)
```

Build Model(s)

```{r, message=F}
# Build our model
# Split the clean data back into our train and test data sets
full$Embarked <- as.factor(full$Embarked)
full$Sex <- as.factor(full$Sex)
train <- full[1:889,]
test <- full[890:1307,]
# Create random forest model
set.seed(123)
rf_model <- randomForest(factor(Survived) ~ Pclass + Sex + Age + SibSp + Fare + Embarked, data=train)
# Model error
plot(rf_model, ylim=c(0,0.40))
# Create regression tree model
rt_model <- rpart(factor(Survived) ~ Pclass + Sex + Age + SibSp + Fare + Embarked, data=train)
prp(rt_model)
# Test and compare models
# Random Forest
rf_prediction <- predict(rf_model, test)
plot(rf_prediction)
rf_solution <- data.frame(PassengerID = test$PassengerId, Survived = rf_prediction, row.names = T)
rf_solution <- rownames_to_column(rf_solution, var="PassengerID")
summary(rf_solution)
# Regression Tree
rt_prediction <- predict(rt_model, test, type = "class")
plot(rt_prediction)
rt_solution <- data.frame(PassengerID = test$PassengerId, Survived = rt_prediction, row.names = T)
rt_solution <- rownames_to_column(rt_solution, var="PassengerID")
summary(rf_solution)
# Write solutions to files
write.csv(rf_solution, file = 'rf_solution.csv', row.names=F, quote = F)
write.csv(rt_solution, file = 'rt_solution.csv', row.names=F, quote = F)
```
