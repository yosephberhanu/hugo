---
title: Principal Component Analysis
description: 'What''s PCA, and how to calculate it'
author:
  - name: Yoseph Berhanu
date: now
categories:
  - Probability theory
  - PCA
draft: false
summary: This blog post delves into Principal Component Analysis (PCA), a statistical method for reducing the dimensions of large datasets while preserving most information. Originating in the early 1900s, PCA is crucial in Machine Learning for tasks like data preprocessing and feature extraction. The process involves standardizing data, computing covariance, and identifying principal components through eigenvectors and eigenvalues. Using the seaborn package's car_crashes dataset, the post demonstrates PCA's ability to simplify complex datasets. It also discusses scikit-learn's PCA implementation, highlighting its efficiency in balancing dimensionality reduction with information retention.
---
Banner Image Credit: [Jernej Furman](https://www.flickr.com/photos/91261194@N06/)

## Introduction
Principal component analysis (PCA) is a widely used statistical method for examining extensive datasets characterized by numerous dimensions or features per observation. PCA is a relatively well-established theory with its roots dating back to the beginning of the 1900s. Its primary aim is to enhance data interpretability while conserving the maximum information possible, thereby enabling the visualization of multidimensional data. Essentially, PCA serves as a statistical approach to diminish dataset dimensionality. This is achieved by transforming the data through linear methods into a fresh coordinate system where a reduced number of dimensions can effectively encapsulate (most of) the data variation compared to the original dataset. 

In various studies, researchers often employ the initial two principal components to plot data in a two-dimensional space, facilitating the identification of clusters among closely associated data points. The applications of principal component analysis span across diverse fields including population genetics, microbiome studies, and atmospheric science. As Machine Learning(ML) is a data-driven approach to building algorithms, PCA plays an important role in ML for Dimensionality Reduction, Data Preprocessing, Visualization, Decorrelation and Feature Extraction, Speeding Up Learning Algorithms, and Noise Filtering.

The idea behind PCA is, that given a dataset with some statistical distribution (i.e., not deterministic), we would like to find features that can best describe as much of the data as possible. By doing so we will be able to explain the data with fewer sets of features than the actual data has. In data involving numerous variables and dimensions, not all variables hold equal importance. Some variables are key while others are less critical. The Principal Component Analysis (PCA) method offers a systematic way to identify and eliminate less significant variables. By transforming the original variables into uncorrelated principal components, PCA retains essential information while discarding less important variables. This process simplifies complex datasets by focusing on the principal components that capture the majority of the dataset's variance. Consequently, PCA enhances data transparency by emphasizing the critical factors while reducing unnecessary complexity.

In this blog, we will go through the steps for calculating PCA with an example data set demonstrating PCA in action to perform dimensionality reduction.

### What Exactly is Principal Component
The principal component represents a novel feature or feature formed by combining the original features linearly. By crafting one or more of these new features, the objective is to ensure that these combinations result in uncorrelated variables, known as principal components. This process involves condensing or compressing most of the information from the initial variables into the first component. In essence, when dealing with n-dimensional data, PCA generates n principal components. However, the primary goal of PCA is to maximize information encapsulation in the initial component, followed by retaining the maximum remaining information in subsequent components. This systematic approach prioritizes the encapsulation of significant data details within these newly constructed components in a step-by-step manner.

Arranging information within principal components enables effective dimensionality reduction without significant loss of information. This process involves discarding components that hold minimal information while regarding the remaining components as the new variables. By prioritizing the retention of informative components and disregarding those with lower significance, one can streamline the dataset and create a new set of variables that effectively captures the essential information from the original dataset. This approach facilitates a more concise representation of the data while preserving the most critical information contained within the retained components.
While representing data with less number of features reduces the complexity of the data aiding in different aspects of machine learning it complicates understanding of the data as the newly formed features (i.e., principal components) do not have a one-to-one mapping with the original feature set. 
## Data
The data we will use in this post comes with [seaborn](https://seaborn.pydata.org/) plotting package. Specifically, we'll use the `car_crashes` dataset from seaborn package. If you're new to `seaborn`, I've [another](/posts/seaborn_demo.ipynb) post that demonstrates different plot types available in seaborn.

The `car_crashes` dataset is data about car accidents, their cause, and cost to insurance companies in the states of the USA. It consists of the following features.

* __total__: Number of drivers involved in fatal collisions per billion miles (5.900–23.900)
* __speeding__: Percentage Of Drivers Involved In Fatal Collisions Who Were Speeding (1.792–9.450)
* __alcohol__: Percentage Of Drivers Involved In Fatal Collisions Who Were Alcohol-Impaired (1.593–10.038)
* __not_distracted__: Percentage Of Drivers Involved In Fatal Collisions Who Were Not Distracted (1.760–23.661)
* __no_previous__: Percentage Of Drivers Involved In Fatal Collisions Who Had Not Been Involved In Any Previous Accidents (5.900–21.280)
* __ins_premium__: Car Insurance Premiums (641.960–1301.520)
* __ins_losses__: Losses incurred by insurance companies for collisions per insured driver (82.75–194.780)
* __abbrev__: A two-letter abbreviation of US state name the data stands for

#### Import the required packages
Let's start by importing the necessary packages


```
import numpy as np
import seaborn as sns
import pandas as pd
```

### Load the data
We can now load the data and try to understand it


```
car_crashes = sns.load_dataset("car_crashes")
car_crashes.head()
```

We can see all features except the abbreviation (which represents the US state the sample stands for) are continuous-valued. 


```
car_crashes.info()
```

We'll remove the apprev column as it is not relevant to our demonstration in this post


```
car_crashes = car_crashes.drop('abbrev', axis=1)
```

### Maximum, Minimum, Standard deviation and Average Values
The maximum value in each of the columns


```
car_crashes.max()
```

The minimum value in each of the columns


```
car_crashes.min()
```

The range in each column is the difference between the max and minimum values 


```
car_crashes_range = car_crashes.max() - car_crashes.min()
car_crashes_range
```

We can visualize the difference in the range 


```
p1 = car_crashes_range.plot(
    legend=False,
    kind="bar",
    rot=45,
    color="blue",
    fontsize=16,
)
p1.set_title("Range of values in our car crash data", fontsize=16)
p1.set_xlabel("Feature", fontsize=14)
p1.set_ylabel("Range", fontsize=14)
```

We can get the mean value and standard deviation of each column as well 


```
car_crashes.mean(axis=0)
```


```
car_crashes.std(axis=0)
```

## Computing PCA

Calculating PCA involves five steps. These are

1. Standardization
2. Covariance Matrix Computation
3. Eigenvectors and Eigenvalues
4. Feature Vector
5. Recast the Data

Now that we've explained the data we'll use for demonstrating PCA and the steps involved, let's go through each step with the aforementioned data and see PCA in action.

## PCA in Action
### Step 1. Standardization
PCA is sensitive to variance, meaning if features different range in values the PCA calculation will be dominated by features with a larger range. In our dataset, we have features that represent a percentage value (hence the potential values range from 0 to 100, but the actual maximum value in the dataset is way below 100). We also have other features that have a more inconsistent range. For example, the `ins_premium` has a range of `659.560` while `speading` has a range of `7.658`. As a result, the PCA analysis will be greatly (and incorrectly) influenced by a change in `ins_premium` value. 

To address this issue, we perform standardization before PCA, that is we bring values of all features to a standard range. The one common way to standardize any data is the following formula.

$$
z = \frac{v - \mu}{\sigma}
$$
Where $z$ is the standardized value, $v$ is the original value,  $\mu$ stands for mean and $\sigma$ is the standard deviation

We can perform this calculation in python as follows


```
car_crashes_std = (car_crashes - car_crashes.mean(axis=0)) / car_crashes.std(axis=0)
car_crashes_std.head()
```

Let's perform a similar visualization on the range as we did earlier


```
p2 = (car_crashes_std.max() - car_crashes_std.min()).plot(
    legend=False, kind="bar", rot=45, color="blue", fontsize=16
)
p2.set_title("Range of values in our car crash data after standardization", fontsize=16)
p2.set_xlabel("Features", fontsize=14)
p2.set_ylabel("Range", fontsize=14)
```

We can see that now the range is more consistent across the features

### Step 2. Covariance Matrix
Covariance Matrix
To get a sense of how the values of the input dataset vary from the mean to each other we compute the covariance matrix. The covariance matrix is a $m × m$ symmetric matrix (where m is the number of features, we've in our input dataset). Features might be highly correlated and through computing the covariance matrix we can identify this relationship. 

For example, a covariance matrix of an input dataset with 4 features $(f_1,f_2,f_3,f_4)$ will be a 4x4 matrix of the following form

$$ 
Covariance Matrix = \begin{bmatrix}
      cov(f_1,f_1) & cov(f_1,f_2) & cov(f_1,f_3) & cov(f_1,f_4) \\
      cov(f_2,f_1) & cov(f_2,f_2) & cov(f_2,f_3) & cov(f_2,f_4) \\
      cov(f_3,f_1) & cov(f_3,f_2) & cov(f_3,f_3) & cov(f_3,f_4) \\
      cov(f_4,f_1) & cov(f_4,f_2) & cov(f_3,f_3) & cov(f_4,f_4)
\end{bmatrix} 
$$

We compute the covariance between to features $f_x$ and $f_y$ as follows
$$
cov(f_x,f_y) = \dfrac{\sum(x_i-\mu_x) (y_i-\mu_y)}{n}
$$
Where $x_i$ and $y_i$ are the $i^{th}$ values for feature $x$ and $y$ and $n$ is the total number of data points.

With this equation notice how 

* covariance of a feature f with itself (i.e., $Cov(f_x,f_x)$) the variance of the feature (i.e., $Var(f_x)$)
* covariance is symmetric, i.e., $Cov(fx,fy) = Cov(f_y,f_x)$

Now that we know that the covariance matrix is not more than a table that summarizes the correlations between all the possible pairs of variables, let’s move to the next step. While we can use these equations to calculate the covariance matrix pandas provide a built-in method for calculating it in one go.


```
cov_mat = car_crashes_std.cov()
```

We can visualize this matrix using `seaborn`'s heatmap 


```
sns.heatmap(cov_mat, annot=True)
```

### Step 3. Eigenvectors and Eigenvalues

Eigenvectors and eigenvalues, fundamental concepts from linear algebra, play a crucial role in computing the principal components of a dataset from its covariance matrix. In an n-dimensional dataset comprising n features, there exist n eigenvectors accompanied by n corresponding eigenvalues.

Eigenvectors derived from the Covariance matrix represent the axes' directions where the highest variance or most information is concentrated. These eigenvectors are termed Principal Components. Eigenvalues, on the other hand, serve as coefficients associated with eigenvectors, signifying the quantity of variance encapsulated within each Principal Component. Essentially, eigenvalues provide a measure of the amount of variance carried by the respective eigenvectors, highlighting their significance in determining the variance distribution across the dataset's principal axes.

Arranging your eigenvectors based on their corresponding eigenvalues, from highest to lowest, provides the principal components in descending order of significance. This ranking scheme ensures that the principal components are prioritized based on the amount of variance or information they encapsulate. The eigenvector associated with the highest eigenvalue represents the most significant principal component, followed by subsequent components in decreasing order of importance, allowing for a systematic determination of the most influential axes within the dataset.

We can use `numpy`'s `linalg.eigh( )` method to calcuate eigenvectors and  eigenvalues a matrix.


```
eigen_values, eigen_vectors = np.linalg.eigh(cov_mat)
```


```
eigen_values
```


```
eigen_vectors
```

We should now sort the Eigenvalues in the descending order along with their corresponding Eigenvector.


```
# sort the eigenvalues in descending order
sorted_index = np.argsort(eigen_values)[::-1]

sorted_eigenvalue = eigen_values[sorted_index]

# Similarly sort the eigenvectors
sorted_eigenvectors = eigen_vectors[:, sorted_index]
```

### Step 4. Feature Vector

Once sorted, we can select the subset of the Eigenvalue as per our requirement. In this case, since we are interested in the two principal components, we take the first two values (i.e. n_components = 2).
This forms our feature vector which is a matrix that has as columns the eigenvectors of the components that we decide to keep. By only keeping a subset of the Eigenvectors we are reducing the number of features hence the notion of dimensionality reduction.


```
n_components = 2 
feature_vector = sorted_eigenvectors[:,0:n_components]

feature_vector
```

### Step 5. Recast the Data


```
car_crash_reduced = -np.dot(
    feature_vector.transpose(), car_crashes_std.transpose()
).transpose()
cars_pca = pd.DataFrame(data=car_crash_reduced, columns=["PC 1", "PC 2"])
cars_pca.head()
```

### Using `scikit-learn` package 

The `scikit-learn` package comes with an API to calculate PCA without having to follow the above steps


```
from sklearn.decomposition import PCA

pca = PCA(n_components=2)

pcs = pca.fit_transform(car_crashes_std)

principal_cars = pd.DataFrame(data=pcs, columns=["PC 1", "PC 2"])

principal_cars.head()
```

One advantage of the `scikit-learn` implementation of PCA is that it implements _Probabilistic Principal Component Analysis_ ^[“Mixtures of Probabilistic
Principal Component Analysers" - Michael E. Tipping and Christopher M. Bishop http://www.miketipping.com/papers/met-mppca.pdf]. This enables us obtain the log maximum-likelihood of each sample. Maximum likelihood refers to the degree (in mean or standard deviation) the probablity distribution we've is going to represent most of the data we are trying to model. The PPCA implementation in `scikit-learn` gives as access to the log likelihood score which we can use to compare different distributions (in this case different number of principal components ) 



```
pca.score_samples(car_crashes_std)
```

These values represent how much of the data (likelihood) our PCA components are covering. To get an average value we can use a different method from PCA as follows


```
pca.score(car_crashes_std)
```

 The log-likelihood value for a given model can range from negative infinity to positive infinity. The actual log-likelihood value for a given model is mostly meaningless, but it's useful for comparing two or more models. We can compare our previous PCA model with one that has 3, 4, 5 components 


```
pca3 = PCA(n_components=3)

pcs3 = pca3.fit_transform(car_crashes_std)

pca3.score(car_crashes_std)
```


```
pca4 = PCA(n_components=4)

pcs4 = pca4.fit_transform(car_crashes_std)

pca4.score(car_crashes_std)
```


```
pca5 = PCA(n_components=5)

pcs5 = pca5.fit_transform(car_crashes_std)

pca5.score(car_crashes_std)
```

While there is an improvement in the actual score we are increasing the number of components. Since our objective is to reduce the number of dimensions (features) and adding more components works against that aim, the improvements gained in log-likelihood score are not worth the additional dimensions we are introducing.

