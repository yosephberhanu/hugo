---
title: Song popularity prediction
description: Linear and non linear regression
date: 11-06-2023
categories:
  - Regression
draft: false
execute:
  warning: false
summary: In this blog post, we explore the use of linear and nonlinear regression algorithms to predict song popularity using a dataset of 30,000 Spotify songs. By examining attributes like artist details and musical features, we train various machine learning models, aiming to understand and forecast what makes a song popular. This endeavor blends data analysis with the complexities of musical trends, offering insights into the dynamics of song popularity.
---
# Introduction

## Dataset
In this blog post I'll try to explore linear and nonlinear regression ML algorithms to predict popularity of a song based on a data set of almost 30,000 songs from Spotify API collected using the [`spotifyr` package](https://www.rcharlie.com/spotifyr/). The package and original data set can be found [here](https://www.kaggle.com/datasets/joebeachcapital/30000-spotify-songs/). 

### Description of dataset
This dataset encompasses various attributes providing diverse information about songs, artists, popularity, musical features, and playlist characteristics, making it suitable for music-related analyses or applications in data analysis and machine learning.

* __Track_id__: Unique ID for a song
* __Track_name__: Name of the song
* __Track_artist__: Artist of the song
* __Track_popularity__: Song popularity rating (0-100)
* __Track_album_id__: Unique ID for the album
* __Track_album_name__: Name of the album
* __Track_album_release_date__: Release date of the album
* __Playlist_name__: Name of the playlist
* __Playlist_id__: ID of the playlist
* __Playlist_genre__: Genre of the playlist
* __Playlist_subgenre__: Subgenre of the playlist
* __Danceability__: Describes how suitable a track is for dancing (0.0 to 1.0)
* __Energy__: Represents intensity and activity of the track (0.0 to 1.0)
* __Key__: Estimated overall key of the track
* __Loudness__: Overall loudness of a track in decibels (dB)
* __Mode__: Indicates the modality (major or minor) of a track (0 for minor, 1 for major)
* __Speechiness__: Detects the presence of spoken words in a track (0.0 to 1.0)
* __Acousticness__: Confidence measure of whether the track is acoustic (0.0 to 1.0)
* __Instrumentalness__: Predicts whether a track contains no vocals (0.0 to 1.0)
* __Liveness__: Detects the presence of an audience in the recording
* __Valence__: Describes the musical positiveness conveyed by a track (0.0 to 1.0)
* __Tempo__: Estimated tempo of a track in beats per minute (BPM)
* __Duration_ms__: Duration of the song in milliseconds

## Task
The task for this blog post is to train a couple of regression models and predict how popular a song is given the other features about the song.  
# Preprocessing
## Imports


```
import numpy as np
import seaborn as sns
import pandas as pd
from sklearn import preprocessing
import matplotlib.pyplot as plt
from sklearn.model_selection import train_test_split
from sklearn.model_selection import RandomizedSearchCV
from sklearn.ensemble import (
    AdaBoostRegressor,
    RandomForestRegressor,
    GradientBoostingRegressor,
    BaggingRegressor,
    ExtraTreesRegressor,
)
from sklearn.neighbors import KNeighborsRegressor
from sklearn.neighbors import RadiusNeighborsRegressor
from sklearn import svm

from sklearn.metrics import mean_squared_error
```

## Load the data


```
songs = pd.read_csv("../data/spotify_songs.csv")
songs.head()
```

## Remove identifiers
Features like track_id, track_name, track_artist, track_album_id, track_album_name, playlist_id,playlist_name will have little contribution to our model as they are unique identifiers. Hence we'll remove them from the data.


```
songs = songs.drop(
    [
        "track_id",
        "track_name",
        "track_artist",
        "track_album_id",
        "track_album_name",
        "playlist_id",
        "playlist_name",
    ],
    axis="columns",
)
songs.head()
```

## Date Data
To capture seasonal popularity, I'll split the date value into three columns with year, month and day values. We first convert the string representation of track_album_release_date to date datatype and takeout the individual values separately.


```
songs["track_album_release_date"] = pd.to_datetime(
    songs["track_album_release_date"], format="%Y-%m-%d", errors="coerce"
)

songs["release_year"] = songs["track_album_release_date"].dt.year
songs["release_month"] = songs["track_album_release_date"].dt.month
songs["release_day"] = songs["track_album_release_date"].dt.day
songs = songs.drop(["track_album_release_date"], axis="columns")
```

## Missing values
Let's check if there are any missing values in our dataset


```
songs.isnull().values.any()
```

It seems we've some


```
songs.isnull().sum()
```

Given the missing date values are around 17% our data simply dropping the rows does not yield an ideal outcome. Hence, let's to fill the missing columns with the most common value in that feature


```
songs["release_year"].fillna(
    songs["release_year"].value_counts().index[0], inplace=True
)
songs["release_month"].fillna(
    songs["release_month"].value_counts().index[0], inplace=True
)
songs["release_day"].fillna(songs["release_day"].value_counts().index[0], inplace=True)
songs
```

If we check once more, we see there are no missing values anymore


```
songs.isnull().values.any()
```

## Visualization
Let's visualize the data and get a better understanding of the values 
### Genre


```
sns.countplot(songs, x="playlist_subgenre", hue="playlist_genre")
plt.xticks(rotation=90)
```

From this we can see playlist_subgenre is full contained in playlist_genre hence we can remote the playlist_genre as the playlist_subgenre has more fine-grained information 


```
songs = songs.drop("playlist_genre", axis="columns")
```

## One hot encoding
Notice how we have a categorical feature (i.e.,  playlist_subgenre), we need to one-hot encode this column.


```
playlist_subgenre = pd.get_dummies(songs["playlist_subgenre"])
songs = pd.concat([songs, playlist_subgenre], axis="columns")
songs = songs.drop("playlist_subgenre", axis="columns")
songs
```

### Correlation


```
cor = songs.corr()
sns.heatmap(cor)
cor = cor.abs()
```

Let's see the top 5 correclations


```
cor_values = (
    cor.where(np.triu(np.ones(cor.shape), k=1).astype(bool))
    .stack()
    .sort_values(ascending=False)
)
top_n = 5
print("Top 5 correlation values are")
for i, v in cor_values[:top_n].items():
    print("{} - {} ".format(i, v))
```

As expected 'energy', 'loudness' are correlated to a degree but none of these correlation values are significantly high enough to warrant removal before the training so we keep all our features.

## Target and Data
Let's now split features into X and Y with the latter being our target for prediction


```
y = songs["track_popularity"].to_numpy()
x = songs.drop("track_popularity", axis="columns").to_numpy()
```

## Normalization
Normalize the features to bring them all to 0-1 range


```
x_normalized = preprocessing.normalize(x)
```

## Training and Test Split and Data
Let's split our data into training and test sets with 80 to 20 ratio.


```
x_train, x_test, y_train, y_test = train_test_split(x_normalized, y, test_size=0.20)
print("Training X Shape: {}".format(x_train.shape))
print("Training Y Shape: {}".format(y_train.shape))
print("Test X Shape: {}".format(x_test.shape))
print("Test y Shape: {}".format(y_test.shape))
```

# Training and Optimization
## Regression models
We will try to train on multiple algorithms and perform hyperparameter tuning on each to find the best performing model


```
models = {
    "ExtraTreesRegressor": {
        "model": ExtraTreesRegressor(),
        "params": {
            "n_estimators": [25, 50, 75, 100],
            "max_depth": [5, 10, 20],
            "min_samples_split": [3, 5, 7],
            "min_samples_leaf": [3, 4, 5],
        },
    },
    "adaboost": {
        "model": AdaBoostRegressor(),
        "params": {
            "n_estimators": [25, 50, 75, 100],
            "loss": ["linear", "square", "exponential"],
        },
    },
    "svm": {
        "model": svm.SVR(),
        "params": {
            "degree": [2, 4, 6],
            "C": [0.5, 1.0, 1.5],
        },
    },
    "knr": {
        "model": KNeighborsRegressor(),
        "params": {
            "n_neighbors": [3, 5, 8],
            "weights": ["uniform", "distance"],
            "leaf_size": [20, 40, 60],
        },
    },
    "rnr": {
        "model": RadiusNeighborsRegressor(),
        "params": {
            "radius": [0.5, 1.0, 1.5],
            "weights": ["uniform", "distance"],
            "leaf_size": [20, 40, 60],
        },
    },
    "RandomForestRegressor": {
        "model": RandomForestRegressor(),
        "params": {
            "n_estimators": [25, 50, 75, 100],
            "max_depth": [5, 10, 20],
            "min_samples_split": [3, 5, 7],
            "min_samples_leaf": [3, 4, 5],
        },
    },
    "boosting": {
        "model": GradientBoostingRegressor(),
        "params": {
            "n_estimators": [25, 50, 75, 100],
            "loss": ["linear", "square", "exponential"],
            "min_samples_split": [3, 5, 7],
        },
    },
}
```

Now let's perform the training and hyperparameter tuning.
Start by preparing a variable to store the best model and it's param values in 


```
best_model, best_params, best_performance = None, None, float("inf")
```

Loop through each model and perform randomized search.
__Note__: Running the following code takes a lot of time so what I did was run it on google colab and bring the results here. 
```{.python}
for name, info in models.items():
    random_search = RandomizedSearchCV(
        info["model"],
        info["params"],
        scoring="neg_mean_squared_error",
        cv=5,
        n_jobs=-1,
    )

    random_search.fit(x_train, y_train)

    # Get the best model and its parameters
    winning_model = random_search.best_estimator_
    winning_model_params = random_search.best_params_
    # Check the performance of the best model using the test data.
    # We need to round the predicted values because the target feature is integer
    y_pred = np.round(winning_model.predict(x_test))
    error = mean_squared_error(y_test, y_pred)
    # Update the best_model_info if the current model has lower MSE
    if error < best_performance:
        best_performance = error
        best_model = winning_model
        best_params = winning_model_params

# Print the best model information
print(f"Best Model: {best_model}")
print(f"Best Parameters: {best_params}")
```

# Conclusion

I found the best performance in terms of mean absolute error comes from AdaBoostRegressor with the following details

### Best Model
Best Model: AdaBoostRegressor(n_estimators=75)

Best Parameters: {'n_estimators': 75, 'loss': 'linear'}

Lowest Mean Squared Error: 591.7667123496269

## Other sample runs are as follows
### KNeighborsRegressor
* Model: KNeighborsRegressor(leaf_size=20, n_neighbors=8)
* Parameters: {'n_neighbors': 8, 'leaf_size': 20}
* Mean Squared Error: 619.6920968478757

### RadiusNeighborsRegressor
* Model: RadiusNeighborsRegressor(leaf_size=40, radius=0.5, weights='distance')
* Parameters: {'weights': 'distance', 'radius': 0.5, 'leaf_size': 40}
* Mean Squared Error: 615.0418760469012

