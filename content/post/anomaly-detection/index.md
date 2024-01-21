---
title: Anomaly Detection
description: Using DBSCAN
date: '2023-11-04'
categories:
  - Anomaly Detection
  - DBSCAN
draft: false
execute:
  warning: false
summary: This blog post explores Anomaly Detection, a key application of unsupervised machine learning, also known as Outlier Analysis. It's a process of identifying unusual patterns or data points in a dataset that deviate significantly from the norm. Anomaly detection is crucial in various fields like finance for fraud detection, network security, health monitoring, and industrial systems. The post discusses different machine learning algorithms used for anomaly detection, including clustering methods (DBSCAN, K-means), density-based approaches (Gaussian Mixture Models), one-class classification algorithms (Isolation Forest, One-Class SVM), and deep learning techniques.
---
Banner Image Credit [Evidently AI](https://www.evidentlyai.com)

# Intro
In this blog post I'll demonstrate a category of application for unsupervised machine learning - Anomaly Detection. Also known as Outlier analysis, it refers to the process of identifying patterns or instances within a dataset that deviate significantly from the norm or expected behavior. These anomalies can represent events, observations, or data points that differ from the majority of the dataset due to various reasons like errors, unexpected occurrences, or genuine novel behavior.

The primary goal of anomaly detection is to distinguish these irregular patterns or outliers from the regular or "normal" data points. This process is crucial across multiple domains and applications, including fraud detection in finance, network security, health monitoring, industrial systems, and more.

Supervised, unsupervised, and semi-supervised machine learning algorithms, including clustering (like DBSCAN, K-means), density-based methods (like Gaussian Mixture Models), one-class classification algorithms (like Isolation Forest, One-Class SVM), and deep learning approaches, are utilized to detect anomalies based on patterns, deviations, or differences in the data.

Anomaly detection is a critical task as it helps in identifying potential problems, threats, or opportunities that may require immediate attention or further investigation. However, it can also be challenging due to the dynamic nature of anomalies and the need for adaptable algorithms capable of handling diverse types of anomalies within different datasets and contexts.

# DBSCAN

In this blog post we'll be using DBSCAN as our outlier analysis technique. DBSCAN is an unsupervised machine learning technique rooted in density-based principles. This algorithm effectively detects clusters within extensive spatial datasets by assessing the local density of data points. It typically performs well in anomaly detection scenarios. Data points that do not fit within any cluster are assigned their distinct class, labeled as -1, simplifying their identification. Notably, DBSCAN exhibits robustness in handling outliers, especially when dealing with data represented by non-discrete points.
DBSCAN focuses on the density of data points rather than assuming a specific number of clusters beforehand. It identifies clusters based on the density of points within a neighborhood. The algorithm defines clusters as regions where there is a high density of data points. It considers two essential parameters:

* Epsilon (ε): This represents the radius within which neighboring points are considered part of the same cluster.
* Minimum Points (MinPts): It sets the minimum number of points required within the epsilon radius to form a cluster.

DBSCAN starts by selecting a random unvisited point and examines its neighborhood. If the point is a core point, it forms a cluster and expands it by adding neighboring points. If the point is a border point, it's added to an existing cluster. If neither, it's labeled as noise.

DBSCAN can identify clusters of varying shapes and handle varying densities within clusters effectively. It is robust in detecting clusters of arbitrary shapes, can handle noise/outliers, and doesn’t require specifying the number of clusters beforehand. However, setting suitable values for ε and MinPts can impact the performance of DBSCAN. Choosing appropriate parameters often requires domain knowledge or experimentation. 

# Data
For the purpose of demonstration, we will be using a dataset that comes bundled with the seaborn package in python. Among the handful of options, I've chosen to use the titanic dataset. Let's get started

## Imports


```
import numpy as np
import seaborn as sns
from sklearn.cluster import DBSCAN
```

Let's see what comes with seaborn


```
print(sns.get_dataset_names())
```

## Load the data
Let us load the car car titanic dataset


```
titanic = sns.load_dataset("titanic")
titanic.head()
```

We'll use the age and fair columns in this case to make our demonstrations either. Let's plot the data and get a sense of how it looks


```
sns.scatterplot(data=titanic, x="fare", y="age")
```

## Missing values
Let's check if there are any missing values in our dataset


```
titanic_age_fare = titanic[["fare", "age"]]
titanic_age_fare.isnull().values.any()
```

Looks like we've missing values, let's see how many in each feature


```
print("Missing fare values", titanic_age_fare["fare"].isnull().sum())
print("Missing age values", titanic_age_fare["age"].isnull().sum())
```

Given that there is a high number of missing age values we can't simpply drop these rows, so let's fill them with the avarage age 


```
titanic_age_fare["age"].fillna((titanic_age_fare["age"].mean()), inplace=True)
titanic_age_fare.isnull().values.any()
```

# Model fitting
Let's try to fit a DBSCAN model with the two features and see  


```
dbscan = DBSCAN()
dbscan.fit(titanic_age_fare)
```

Now let's visualize and see if there are outliers based on DBSCAN's classification


```
sns.scatterplot(titanic_age_fare, x="fare", y="age", c=dbscan.labels_)
```


```
print(titanic_age_fare[dbscan.labels_ == -1])
```

Based on this visualization and the labels created we can see DBSCAN was not able to detect any outliers with the given features. This holds even after experimenting with multiple values of epsilon and min_samples for DBSCAN.

