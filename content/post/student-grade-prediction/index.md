---
title: Student Grade Prediction
description: Using three ML classifier to predict student grade
author:
  - name: Yoseph Berhanu
date: now
categories:
  - Confusion Matrix
  - Classification
execute:
  warning: false
summary: This blog post details the use of skit-learn's classification algorithms to predict student grades using Moodle LMS data, including quiz and assignment performance. The process involves installing essential packages, data loading and preprocessing, and normalization. Various classifiers like Logistic Regression, Decision Tree, and KNeighborsClassifier are trained and evaluated. The best-performing models are further optimized using GridSearchCV, with LinearSVC achieving the highest accuracy of 82%. The approach demonstrates the effective application of machine learning in educational data analysis.
---
Banner Image Credit: [Jernej Furman](https://www.flickr.com/photos/91261194@N06/)

In this blog I will use three different machine learning classification algorithms provided in skit-learn to predict student grade using data about students' performance in quizzes, assignments and other features collected in Moodle LMS

## 1. Preparation
### 1.1 Install required packages

```python
%pip install numpy sklearn pandas matplotlib seaborn
```

### 1.2 Perform the necessary imports


```
import numpy as np
import pandas as pd
from matplotlib import pyplot as plt
import seaborn as sns
import warnings
from sklearn import preprocessing

from sklearn.linear_model import (
    LogisticRegression,
    SGDClassifier,
)
from sklearn.tree import DecisionTreeClassifier
from sklearn.neighbors import KNeighborsClassifier
from sklearn.naive_bayes import MultinomialNB
from sklearn.ensemble import (
    BaggingClassifier,
    GradientBoostingClassifier,
    RandomForestClassifier,
)
from sklearn.naive_bayes import GaussianNB
from sklearn.neighbors import KNeighborsClassifier
from sklearn.neural_network import MLPClassifier
from sklearn.svm import SVC
from sklearn.tree import DecisionTreeClassifier
from sklearn.svm import LinearSVC
from sklearn.discriminant_analysis import QuadraticDiscriminantAnalysis
from sklearn.ensemble import AdaBoostClassifier, RandomForestClassifier
from sklearn.gaussian_process import GaussianProcessClassifier
from sklearn.gaussian_process.kernels import RBF

from sklearn.model_selection import train_test_split
from sklearn.metrics import (
    accuracy_score,
    # classification_report,
    # mean_squared_error,
    # r2_score,
    # mean_absolute_error,
    # confusion_matrix
)
```

Suppress Warnings


```
warnings.simplefilter(action='ignore', category=FutureWarning)
```

## 2. Data Understanding and Preprocessing

### 2.1 Data description
The data was collected from a fully online nine-week-long course on, hosted on the online learning management system Moodle. The dataset containes anonymized information relating to 107 enrolled students. The data included students’ grades (from 3 mini projects, 3 quizzes and 3 peer reviews and the final overall grade) as well as the course logs. The deadline for the three mini projects fell within weeks 3, 5 and 8 of the course, whereas the deadline for the quizzes fell within weeks 2, 4 and 8.

* Status0: course / lectures / content related (Course module viewed, Course viewed, Course ac2vity comple2on updated, Course module instance list viewed, Content page viewed, Lesson started, Lesson resumed, Lesson restarted, Lesson ended)
* Status1: assignment related (Quiz aPempt reviewed, Quiz aPempt submiPed, Quiz aPempt summary viewed, Quiz aPempt viewed, Quiz aPempt started, Ques2on answered, Ques2on viewed, Submission re-assessed, Submission assessed, Submission updated, Submission created, Submission viewed)
* Status2: grade related (Grade user report viewed, Grade overview report viewed, User graded, Grade deleted, User profile viewed, Recent ac2vity viewed, User report
viewed, Course user report viewed, Outline report viewed)
* Status3: forum related (Post updated, Post created, Discussion created, Some content has been posted, Discussion viewed)
* 9 grades (Week2_Quiz1, Week3_MP1, ... Week7_MP3)
* 36 logs (Week1_Stat0, Week1_Stat1, Week1_Stat2, Week1_Stat3, ... Week9_Stat0, Week9_Stat1, Week9_Stat2, Week9_Stat3)

### 2.2 Load the data


```
df = pd.read_csv("../data/MP2_Data.csv")
```

## 2.3 Try to learn some information about the data
### Display the first and last 5 rows


```
df.head()
```


```
df.tail()
```

### Display details about each attribute


```
df.info()
```

### Set 'ID' as the index column


```
df.set_index('ID', inplace=True)
df.head()
```

Get additional information about each column


```
df.describe().T
```

From the description (in the pdf file) Week8_total seems to be a sum of the 9 grades, to check, I create a temporary column called sum and compare


```
grade_cols = ['Week2_Quiz1','Week4_Quiz2','Week6_Quiz3','Week3_MP1','Week5_MP2','Week7_MP3','Week3_PR1','Week5_PR2','Week7_PR3']
df['sum'] = df[grade_cols].sum(1)
df[['sum', 'Week8_Total']]
```

These columns are the same representing the same data, hence, I can drop the individual values and keep to total. Also, since I do not plan to perform any time serious analysis (i.e., I'm not interested in the changes over weeks on the grades) I can keep the Week8_Total and drop the individual grade to reduce the features I need to consider.
P.S: I also drop the temporary 'sum' colum


```
grade_cols.append('sum')
print(grade_cols)
df=df.drop(columns=grade_cols)
df.describe().T
```

### Plot the number of students who scored each grade


```
sns.countplot(df, x="Grade")
```

### See if there are missng values 


```
sns.heatmap(df.isnull())
```

There are no missing values for any of the students, perhaps missing a quiz or an assignment has been equated to a grade of zero


```
cor = df.corr()
sns.heatmap(cor)

cor_target = abs(cor['Grade'])
# Selecting highly correlated features
relevant_features = cor_target[cor_target>0.7]
relevant_features
```

We see, as expected, __Week8_total__ is a highly correlated to final __Grade__. Interestingly __Week6_Stat1__ seems to have a high correlation to __Grade__. This means, students who had high activity during the sixth week of the course had scored better on the final grade (I say they scored better because we can see below the correlation is positive )
## 2.4 Preprocessing


```
cor["Week6_Stat1"]["Grade"]
```

I can remove the Week8_Total grade from the features list if I can be sure that the final grade is full based on the this total grade or I can keep it in case other factors were considered in the final grade calculation. In this case the machine learning model I build will be highly dependent on this variable to the point the other features will became irrelevant given the high correlation value observed between this feature and the target (i.e., Grade)


```
#df = df.drop(columns=['Week8_Total'], axis=1)
```


```
#irrelevant_features = cor_target[(cor_target>=0.4) & (cor_target<=0.5)]
irrelevant_features = cor_target[(cor_target>=0.45) & (cor_target<=0.55)]
irrelevant_features
```


```
#df = df.drop(columns=list(irrelevant_features.index), axis=1)
```

## Split Features and target column


```
x = df.drop(columns=['Grade'], axis=1).to_numpy()
y = df['Grade'].to_numpy()
```

### Normalize the features to bring them all to 0-1 range


```
x_normalized = preprocessing.normalize(x)
x_normalized
```

### Split training and Test data


```
x_train, x_test, y_train, y_test = train_test_split(x_normalized,y, test_size=.20)
```


```
print("Training X Shape: {}".format(x_train.shape))
print("Training Y Shape: {}".format(y_train.shape))
print("Test X Shape: {}".format(x_test.shape))
print("Test y Shape: {}".format(y_test.shape))
```

#### Summary of Preprocessing
##### Feature Selection
Given 107 records with 47 attributes I have removed 11 of the attributes during the pre-processing for the following reasons

1. ID: THis is a unique identifier of each row of the data that dose not contain relevant information for our prediction task.
2. grade_columns: These are columns that add up to give the Week8_Total column, since there is a high correlation between these and the total column, I removed them in favor of Week8_Total which provides the same information in a single feature.
3. Week8_Total: I finally remove Week8_Total because it has a high correlation with the target feature which will make our prediction very skewed towards this feature. The is a design decision to avoid a deterministic prediction.

##### Normalization
Then I normalized the data to bring values for each feature between [0-1]. 

##### Training/Test Split

I split the data into training and testing set of, first, 90% to 10% ratio. Even though it is recommended to use a 20% testing split, given our small data set I started with 10% and I came back to update this ratio based on my finding later which showed over-fitting. 

## 3. Training

The problem of predicting one of the six grades given a set of features is a classification problem. I'll first try to train and test multiple classifiers with their default hyperparameter values.


```
models = {
    'LogisticRegression':LogisticRegression(),
    'MultinomialNB': MultinomialNB(),
    'BaggingClassifier': BaggingClassifier(), 
    'DecisionTreeClassifier': DecisionTreeClassifier(),
    'LinearSVC':LinearSVC(),
    'SGDClassifier':SGDClassifier(),
    'KNeighborsClassifier':KNeighborsClassifier(), 
    'RandomForestClassifier':RandomForestClassifier(), 
    'GradientBoostingClassifier':GradientBoostingClassifier(),
    "SVC":SVC(),
    "GaussianProcessClassifier": GaussianProcessClassifier(),
    "MLPClassifier": MLPClassifier(),
    "AdaBoostClassifier": AdaBoostClassifier(),
    "GaussianNB":GaussianNB(),
    "QuadraticDiscriminantAnalysis":QuadraticDiscriminantAnalysis(),
}
```


```
performance = pd.DataFrame(columns=["Train Accuracy","Test Accuracy"], index=list(models.keys()))
```


```
for name, model in models.items():
    # Train the model
    model.fit(x_train, y_train)

    # Make predictions on the test set
    y_train_pred = model.predict(x_train)
    y_test_pred = model.predict(x_test)

    # Calculate evaluation metrics
    test_accuracy = accuracy_score(y_test, y_test_pred)
    train_accuracy = accuracy_score(y_train, y_train_pred)

    #Save Test and Training Accuracies
    performance.loc[name, :] = [train_accuracy,test_accuracy]
```

## 4. Evaluation


```
performance
```

## 5. Optimization 

From our trial run above SGDClassifier,LinearSVC and MLPClassifier seem to have the best accuracy score. Here I'll try to optimize these algorithms with different hyperparameter values to improve accuracy. We do this with the GirdSearch algorithm for hyperparameter tuning.


```
from sklearn.model_selection import GridSearchCV
```

### SGDClassifier


```
model =  SGDClassifier()

grid = {
        'alpha' :[0.0001,0.001, 0.01, 0.1, 1, 10]
        }
gridSearch = GridSearchCV(estimator=model, param_grid=grid, n_jobs=-1, scoring="accuracy")
searchResults = gridSearch.fit(x_train, y_train)

score = accuracy_score(searchResults.best_estimator_.predict(x_test),y_test)
params = searchResults.best_estimator_.get_params()
print("SGDClassifier  best estimator \n alpha = {} \n accuracy_score = {}".format(params["alpha"],score))
```

### LinearSVC


```
model = LinearSVC ()
grid = {'C':[0.0001,0.001, 0.01, 0.1, 1, 10]}
gridSearch = GridSearchCV(estimator=model, param_grid=grid, n_jobs=-1, scoring="accuracy")
searchResults = gridSearch.fit(x_train, y_train)

score = accuracy_score(searchResults.best_estimator_.predict(x_test),y_test)
params = searchResults.best_estimator_.get_params()
print("LinearSVC best estimator \n C = {} \n accuracy_score = {}".format(params["C"],score))
```

### MLPClassifier


```
model = MLPClassifier()

grid={
'learning_rate': ["constant", "invscaling", "adaptive"],
'alpha': [0.0001,0.001, 0.01, 0.1, 1, 10],
}

gridSearch = GridSearchCV(estimator=model, param_grid=grid, n_jobs=-1, scoring="accuracy")
searchResults = gridSearch.fit(x_train, y_train)

score = accuracy_score(searchResults.best_estimator_.predict(x_test),y_test)
params = searchResults.best_estimator_.get_params()
learning_rate = params["learning_rate"]
print("MLPClassifier best estimator")
print(" alpha = {} \n learning_rate = {}".format(params['alpha'],params['learning_rate']))
print(" accuracy_score = {}".format(score))
```

The Best performance is obtained by LinearSVC classifier which is 82%
