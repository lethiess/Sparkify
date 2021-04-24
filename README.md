# **Sparkify**

## Table of Contents

1. [Project Overview](#overview)
2. [Data Exploration](#data-exploration)
3. [Methodology](#methodology)
4. [Results](#results)
5. [File Descriptions](#files)
6. [Installation](#installation)


# Project Overview<a name="overview"></a>

This is the capstone project of Udacity's Data Science Nanodegree programm. This repository contains the Jupyter-notebook of this project and below you can find an informative brief overview of the project.

## Problem

Sparkify is an imaginary music streaming platform with two types of membership:
1. premium: monthly paid rate
2. free: advert financed

As many other modern platforms Sparkify has to deal with **user churn**. This is a huge problem and can cost millions in revenue. In Sparkify's case there are two types of user churn:
1. Downgrade from paid premium membership to a free ad-financed membership 
2. Users who terminate their membership

Both scenarios needs to be prevented. This could be done with targeted discounts and incentives, but therefore it is necessary to know which users will churn next. This is a **classification problem** which can be solved with Machine Learning algorithms. 

Fortunately Sprakify has a logfile of all user events which can be analysed and used to predict the user churn in an early state. Thus the enourmous amount of data this project usses **Apache Spark** as engine for the large-scale data processing and applying Machine Learning algorithms.

## Event logfile

Sparkify's event log is very detailled and every entry of the log contains the following variables:

| variable	    | datatype	    | description
| :---          | :---          | :---
`userId	        `|  (string)	|   User identification number.
`page	        `|  (string)	|   Kind of page accessed e.g. downgrade/upgrade/next song/...
`registration   `|  (string)	|   Date of the registration.
`ts	            `|  (string)	|   Timestamp the the recored event.
`level	        `|  (string)	|   Account type - free or payed.
`status	        `|  (string)	|   HTTP status code of the page event.
`method	        `|  (string)	|   HTTP request type - GET or POST
`gender	        `|  (string)	|   Gender of the user.
`sessionId	    `|  (string)	|   Unique Id of each session.
`auth	        `|  (string)	|   Indicates if the user was logged in.
`firstName	    `|  (string)	|   First name of the user.
`lastName	    `|  (string)	|   Last name of the user.
`location	    `|  (string)	|   Geographical location of the user.
`userAgent 	    `|  (string)	|   Contains information about the used browser and operating system.
`itemInSession  `|  (string)	|   Number of items in current sesseion.
`song	        `|  (string)	|   Name of a song.
`artist	        `|  (string)	|   Name of the artist.
`length	        `|  (string)	|   Length (minutes) of the song.

**Note:** The data types of some variables need to be converted for further processing, e.g. ts and registration into a datetime object or length into a floating point number. 

### Variable: `page`

Beside the `userId` the variable `page` is the most important variable thus it determines exactly which event was triggered by the user.

# Data Exploration<a name="dataexploration"></a>

For insights in the data exploration please take a look at the notebook (file: `Sparkidy.ipynb`) and [this blospost](https://lethiess.medium.com/how-to-detect-user-churn-in-a-early-state-4a51e52c37e7).

# Methodology<a name="methodology"></a>

In this section the methodology of this project is explained.

## Data preprocessing

Once the dataset is loaded with Spark the data needs to be cleaned in an preprocessing step.

### Data cleaning

A single event in the event log is valid, if none of the following variables is null:
`userId`, `sessionId`, `method`, `page`, `ts`,`registration`, `level`, `userAgent`, `method`, `status`

All invalid datasets will be dropped.

## Feature engineering

The following section gives an brief overview about the features needed to train the Machine Learning models and predict the user churn in the event log. 

### Define churn feature

The most important feature is the feature churn which indicates the user churn.

| Churn value | User action
| :---        | :---
| 0           | User never accessed the page "Cancellation Confirmation"
| 1           | User at lear once accessed the page "Cancellation Confirmation"

**Note I:** This vaule should be predicted and is the target variable for all applyied Machine Learning algorithms.

**Note II:** The distribution of the churn values are imbalanced in the given dataset.

### Define categorical features

The following table contains all used categorical variables:

| Variable name	   |Number of categories |	Description
| :---             | :---          | :---
| Level	           | 2             | Membership type of the user: Paid premium subsrciption or free ad-financed account.
| Downgrade         | 2	|  Determines if a user at least once performed a downgrade from a paid premium subscription to a free account.
| Gender	|2|	 Describes the gender of the user. In this dataset only two gender ocurred, but it could be more.
| Operating System	|5 | This variable describes which operating system was used to access Sparkify.
| Browser |	4	| Which browser was used to access Sparkify is described by the user.

**Note:** A [String Indexer](https://spark.apache.org/docs/latest/ml-features#stringindexer) is used to convert the categorical variables into a numerical value which represents a category.

### Define numerical features

The following table contains all used numerical variables:

| Variable  name	| Description
| :---        | :---
| Number of friends | Describes how often a user has accessed the page "Add Friend".
| Numer of Thumbs up |	Describes how often a user has accessed the page "Thumbs up".
| Number of Thumbs down	| Describes how often a user has accessed the page "Thumbs down".
| Like-Dislike-Ratio |	This ration describes the ratio between the number of songs liked (Thumbs up) vs |disliked (Thumbds down).
| Playlist size	| This variable corresponds to the size of the playlist measures by the number how often a |user accessed the page "Add to Playlist".
| User days	| Time in days between the registration and the last time the user accessed Sparkify.
| Songs per session	| This describes to how many songs a user listened during a sesserion on average.
| Num of Downgrades	| The number how often a user performed a downgrade from paid to free account.
| Advert rolled	| Number of adverts where showed to user with a free account.

**Note:** A [Standard Scaler](https://spark.apache.org/docs/latest/ml-features#standardscaler) is used to normalize all numeric variables. 


## Creating the model dataset

The model dataset contrains the churn feature and the categorical and numerical variables. Whereas the churn features is used as target variable in the subsequent Machine Learning algorithms to classify if a user will churn or not. Each prediction is based on the categorical and numerical features. 

For an efficient processing the features will be processed with a [Spark ML Pipeline](https://spark.apache.org/docs/latest/ml-pipeline.html) with the following stages:
* [String Indexer](https://spark.apache.org/docs/latest/ml-features#stringindexer)
* [Standard Scaler](https://spark.apache.org/docs/latest/ml-features#standardscaler)
* [Vector Assembler](https://spark.apache.org/docs/latest/ml-features#vectorassembler)

The last step is very important and the result is a single vector including all features for each dataset entry.

## Building Model

For solving the classification problem the following four classifier are used:
* Logistic Regression
* Random-Forest-Classifier
* Gradient-Boosting-Trees (GBT) Classifier
* Support Vector Machine (LinearSVC)

All classifiers are build with [PySpark's Machine Learning model](https://spark.apache.org/docs/0.9.0/mllib-guide.html) including a [Cross-Validator](https://spark.apache.org/docs/latest/ml-tuning.html) for finding the best hyperparameters in the parameter grid.

## Results<a name="results"></a>

All created models are able to predict the user churn in Sparkify's event data. The results of the models are listed in the table below:


| Classifier                | F1-score  | Accuracy
| :---                      | :---      | :---
| LogisticRegression        | 0.77      | 0.80
| RandomForestClassifier    | 0.71      | 0.76
| GBTClassifier             | 0.77      | 0.80 
| LinearSVC                 | 0.77      | 0.80


After cross validation with the used parameter grid three classifier have performed equaly. This is a bit unexpected and could be a sign of overfitting or a dataset in need of improvement.

For furhter detailes please take a look at the notebook and the [this](https://lethiess.medium.com/how-to-detect-user-churn-in-a-early-state-4a51e52c37e7) blogpost.


# File Descriptions <a name="files"></a>

The following files are included in this repository:

| File                                      | Description   
| :---                                      |:--- 
| `Sparkify.ipynb`                          | Jupyter-Notebook of the project 
| `Sparkify.html`                           | Output of the Jupyter-Notebook as HTML file 
| `.gitignore`                              | Specifies which files are ignored by Git      
| `.gitattributes`                          | Specifies which files are stored in Git-LFS 
| `README.md`                               | This file
| `data/mini_sparkify_event_data.json`      | Small dataset (~120 MB)
| `data/medium_sparkifty_event_data.json`   | Medium sized dataset (~240 MB)


# Installation <a name="installation"></a>

This sections gives an overview about how to install Jupyter notebook and which libraries
are used to run the given notebook.

### Jupyter Notebook

To run this the Jupyter Notebook a installation of Anaconda is recommended.

[Install Anaconda](https://www.anaconda.com/products/individual#Downloads)

Alternatively you can also pip-install Jupyter-Notebook: 

[Install Jupyter without Anaconda](https://jupyter.org/install)

### Libraries

In both cases the following Python packages are required:
* **PySpark** (v2.4.3 or higher)
* Pandas
* Numpy
* Matplotlib
* re

### Run notebook on AWS EMR Cluster

If you want to run this notebook on a Aamazon Web Services (AWS) EMR cluster the following configuration is recommended:
* *release:* emr-5.20.0 to emr-5.29.x 
* *application:* spark  
* *instance-type:* m5.xlarge
* *number of instances:* 3

**Attention:** The version (*release*) of the EMR installation must be in the above mentioned version range. Newer versions doesn't support jupyter notebooks out of the box.









