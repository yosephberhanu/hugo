---
title: "Music Data Clustering"
description: "K-Means Clustering"
# author:
#   - name: Yoseph Berhanu
#     url: https://yosephberhanu.github.io/
#     orcid: 0000-0002-2922-4337
#     affiliation: PhD student at Virginia Tech
#     affiliation-url: https://code-world-no-blanket.github.io/
date: '2023-11-04'
categories: [KMeans,Clustering] # self-defined categories
# citation: 
#   url: https://yosephberhanu.github.io/ml-blog/2023-11-24-quarto-blogs/ 
draft: false # setting this to `true` will prevent your post from appearing on your listing page until you're ready!
execute: 
  warning: false
summary: In this blog post, we apply the K Means clustering algorithm to a Spotify API dataset, comprising song attributes and musical features. After preprocessing, including normalization and removal of unique identifiers, the optimal number of clusters is determined using the Elbow method. The analysis, visualized through clustering, highlights how various attributes, especially song duration, influence song categorization, offering insights into the factors that drive song popularity.
---
Banner Image Credit: [University of Rochester](https://www.rochester.edu/newscenter/global-grouping-theory-math-strategies-students-529492/)

In this post I'll explore the K Means algorithm to cluster [data](../data/spotify_songs.csv) fetched from Spotify API. This data contains the following attributes

* __Track_id__: Unique identifier for a song
* __Track_name__: Name of the song
* __Track_artist__: Artist of the song
* __Track_popularity__: Song popularity rating (0-100)
* __Track_album_id__: Unique identifier for the album
* __Track_album_name__: Name of the album
* __Track_album_release_date__: Release date of the album
* __Playlist_name__: Name of the playlist
* __Playlist_id__: Identifier of the playlist
* __Playlist_genre__: Genre of the playlist
* __Playlist_subgenre__: Subgenre of the playlist
* __Danceability__: Indicates how suitable a track is for dancing (ranging from 0.0 to 1.0)
* __Energy__: Represents the intensity and activity level of the track (ranging from 0.0 to 1.0)
* __Key__: Estimated overall key of the track
* __Loudness__: Overall loudness of a track in decibels (dB)
* __Mode__: Indicates the modality of a track (0 for minor, 1 for major)
* __Speechiness__: Detects the presence of spoken words in a track (ranging from 0.0 to 1.0)
* __Acousticness__: Indicates the likelihood of a track being acoustic (ranging from 0.0 to 1.0)
* __Instrumentalness__: Predicts whether a track contains no vocals (ranging from 0.0 to 1.0)
* __Liveness__: Detects the presence of a live audience in the recording
* __Valence__: Describes the musical positiveness conveyed by a track (ranging from 0.0 to 1.0)
* __Tempo__: Estimated tempo of a track in beats per minute (BPM)
* __Duration_ms__: Duration of the song in milliseconds

## Task
The task for this blog post is to train a couple of regression models and predict how popular a song is given the other features about the song.  
# Preprocessing
## Imports


```
import numpy as np
import pandas as pd
from sklearn import preprocessing
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.cluster import KMeans
```

## Load the data


```
songs = pd.read_csv("../data/spotify_songs.csv")
songs.head()
```

## Remove Features
### identifiers
Features like track_id, track_name, track_artist, track_album_id, track_album_name, playlist_id,playlist_name will have little contribution to our model as they are unique identifiers. Hence, we'll remove them from the data.


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

### Date Data
In this post we'll also drop date data we'd like to focus on music features itself


```
songs = songs.drop(["track_album_release_date"], axis="columns")
```

### Genre
We'll also drop the genre info to ensure that we have a clustering based off undeclared labels to make it truly unsupervised learning 


```
genres = songs[["playlist_genre", "playlist_subgenre"]]
songs = songs.drop(["playlist_genre", "playlist_subgenre"], axis="columns")
```

## Missing values
Let's check if there are any missing values in our dataset


```
songs.isnull().values.any()
```

We don't have any missing values, which is good. Let's revisit what's remaining


```
songs.head()
```

## Visualization
Let's visualize the data and get a better understanding of the values 


```
sns.pairplot(songs)
```

This plots can say a lot about the data, e.g., songs in the lower end of danceability spectrum are less likely to be popular. Aside from that, most of the features seem to have a fairly normal distribution on the data.
## Normalization
Let's scales data to a similar range to ensure consistency in impact of the features.


```
songs_normalized = preprocessing.normalize(songs)
```

# Clustering 
Let us now start clustering the data using KMeans. 
K-means clustering is an unsupervised machine learning algorithm used for partitioning a dataset into K distinct, non-overlapping clusters. The primary objective of K-means is to group data points based on similarities in their features, where each cluster is represented by its centroid, a point at the center of the cluster.

Here's how the K-means algorithm typically works:

1. __Initialization__: Start by randomly selecting K data points from the dataset as initial cluster centroids.
2. __Assignment__: Assign each data point to the nearest centroid, forming K clusters based on proximity or similarity to centroids. This is often done using a distance metric, such as Euclidean distance.
3. __Update Centroids__: Recalculate the centroids of the K clusters by taking the mean of all data points assigned to each centroid.
4. __Reassignment__: Reassign data points to the nearest centroid based on the updated centroids.
5. __Repeat__: Iteratively update centroids and reassign data points until convergence criteria are met. Convergence occurs when centroids no longer change significantly or after a specified number of iterations.

The algorithm aims to minimize the within-cluster sum of squares (WCSS) or inertia, which represents the total squared distance of each point within its respective cluster to its centroid. K-means aims to optimize this objective function to create compact, well-separated clusters.

Key points about K-means clustering:

* K-means is sensitive to the initial placement of centroids, which can lead to different outcomes.
* It works well when clusters are well-separated and have a roughly spherical shape.
* The algorithm may struggle with non-linear clusters, varying cluster sizes, or clusters with irregular shapes.
* Determining the optimal number of clusters (K) is crucial but not always straightforward. Techniques like the elbow method or silhouette score can help in selecting an appropriate K value.
K-means clustering finds applications in various fields, including customer segmentation, image segmentation, anomaly detection, and recommendation systems, among others, where identifying natural groupings or patterns within data is essential.

We can use the Elbow method to have an indication of clusters for our data. It consists in the interpretation of a line plot with an elbow shape. The number of clusters is where the elbow bends. The x axis of the plot is the number of clusters and the y axis is the Within Clusters Sum of Squares (WCSS) for each number of clusters:


```
from sklearn.cluster import KMeans

ks = list(range(1, 14))
wcss = []
for i in ks:
    kmeans = KMeans(n_clusters=i, init="k-means++", random_state=42)
    kmeans.fit(songs_normalized)
    wcss.append(kmeans.inertia_)

lp = sns.lineplot(x=ks, y=wcss)
lp.set_xticks(range(len(wcss)), labels=ks)
```

It seems we've three clustors


```
sns.set(rc={"figure.figsize": (11.7, 8.27)})
kmeans = KMeans(n_clusters=3, init="k-means++", random_state=42)
kmeans.fit(songs_normalized)

fig = plt.figure()
fig.subplots_adjust(hspace=0.4, wspace=0.4)

cols = len(songs.columns)
fig_count = 1
for j in range(cols - 1):
    # if i == j:
    #     break
    ax = fig.add_subplot(4, 4, fig_count)
    fig_count = fig_count + 1
    sns.scatterplot(
        data=songs,
        x=songs.columns[j + 1],
        y="track_popularity",
        hue=kmeans.labels_,
        ax=ax,
    )

plt.show()
```

From this figure we see song duration of the song in milliseconds has a clear impact on the clustering categories.
